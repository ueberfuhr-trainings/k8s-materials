---
layout: default
title: PostgreSQL auf Kubernetes – Best Practices
---

# PostgreSQL auf Kubernetes / OpenShift – Best Practices

## Warum ein StatefulSet für Datenbanken?

Datenbanken wie PostgreSQL sind **stateful** und haben andere Anforderungen als typische stateless Anwendungen:

### 1. Stabile Identität

Pods im StatefulSet haben feste Namen:

```
postgres-0, postgres-1, ...
```

Diese bleiben auch nach Neustarts erhalten. Das ist wichtig für Replikation, Leader Election und Cluster-Kommunikation.

### 2. Feste Volume-Zuordnung

Jeder Pod bekommt sein eigenes Persistent Volume:

```
postgres-0 → PVC data-postgres-0
postgres-1 → PVC data-postgres-1
```

Das garantiert: keine Datenvermischung und Persistenz über Pod-Restarts hinweg.

### 3. Geordnetes Starten / Stoppen

StatefulSets starten Pods sequenziell:

```
postgres-0 → postgres-1 → postgres-2
```

Das ist wichtig für Replikationsketten und einen sauberen Clusteraufbau.

### 4. Skalierung mit Bedeutung

Jeder Pod ist eine eigenständige DB-Instanz. Es gibt kein zufälliges Load Balancing wie bei Deployments.

## Warum mehrere Pods auf einem PV nicht funktionieren

PostgreSQL ist **kein Shared-Disk-System**. Wenn mehrere Pods dasselbe Volume nutzen, drohen:

- Datenkorruption
- WAL-Konflikte (Write-Ahead Log)
- Fehlende Prozess-Synchronisation
- Scheiternde Crash Recovery

> **Regel:** Ein PostgreSQL-Prozess = ein Data Directory = ein Volume

## Rolle von Kubernetes Services

Ein Service bietet eine virtuelle IP und verteilt Traffic auf Pods. Aber er bietet keine stabile Identität pro Pod und keine Unterscheidung von Rollen (Primary/Replica). Ein Service ersetzt **keine** StatefulSet-Identität.

Für StatefulSets verwendet man einen **Headless Service** (`clusterIP: None`), der statt einer gemeinsamen IP direkte DNS-Einträge pro Pod erzeugt (z.B. `postgres-0.postgres`).

## Wann reicht ein einfaches Deployment + PVC?

Ein einfaches Deployment mit einem PVC ist ausreichend, wenn:

- nur **eine einzige DB-Instanz** benötigt wird
- keine Replikation oder Clustering nötig ist
- es sich um eine **Dev/Test-Umgebung** handelt

## Beispiel: PostgreSQL mit StatefulSet

### 1. Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
type: Opaque
stringData:
  POSTGRES_DB: appdb
  POSTGRES_USER: appuser
  POSTGRES_PASSWORD: secretpassword
```

### 2. Headless Service (für DNS)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
    - port: 5432
      name: postgres
```

Erzeugt DNS-Einträge wie `postgres-0.postgres`.

### 3. Client Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-client
spec:
  selector:
    app: postgres
  ports:
    - port: 5432
      targetPort: 5432
```

### 4. StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: postgres
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: quay.io/sclorg/postgresql-16-c9s
          ports:
            - containerPort: 5432
          envFrom:
            - secretRef:
                name: postgres-secret
          volumeMounts:
            - name: data
              mountPath: /var/lib/pgsql/data
          readinessProbe:
            exec:
              command:
                - pg_isready
                - -U
                - appuser
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            exec:
              command:
                - pg_isready
                - -U
                - appuser
            initialDelaySeconds: 30
            periodSeconds: 10
  volumeClaimTemplates:
    - metadata:
        name: data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 5Gi
```

### Deployment

```bash
oc apply -f postgres.yaml
```

## OpenShift-Hinweis

Standard-PostgreSQL-Images (z.B. `postgres:17-alpine`) laufen auf OpenShift oft nicht, da sie als Root laufen. Verwende stattdessen OpenShift-kompatible Images:

- `quay.io/sclorg/postgresql-16-c9s`
- Red Hat zertifizierte Images aus dem OperatorHub

## Skalierung & Replikation

Ein StatefulSet allein reicht nicht für echte Hochverfügbarkeit. PostgreSQL benötigt zusätzlich:

- Konfiguration von Primary und Replica(s)
- Streaming Replication oder logische Replikation
- Failover-Mechanismen

Das muss manuell konfiguriert oder über einen Operator automatisiert werden.

## Empfehlung: PostgreSQL Operator

Für produktive Systeme empfiehlt sich ein Operator, der all diese Aspekte automatisiert:

- Automatische Replikation und Failover
- Backups und Point-in-Time Recovery
- Automatisierte Updates
- TLS-Verschlüsselung

Bekannte Operatoren:

- [Zalando Postgres Operator](https://github.com/zalando/postgres-operator)
- [Crunchy Postgres Operator](https://github.com/CrunchyData/postgres-operator)
- [CloudNativePG](https://cloudnative-pg.io/)

## Zusammenfassung

- PostgreSQL braucht stabile Identität → **StatefulSet**
- Pro Pod ein eigenes Volume
- Mehrere Pods auf einem PV → **Datenkorruption**
- Einfaches Deployment + PVC nur für Single-Instanz in Dev/Test
- Produktion → **Operator verwenden**
