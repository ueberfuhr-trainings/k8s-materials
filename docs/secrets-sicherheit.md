---
layout: default
title: Sind Secrets in Kubernetes/OpenShift sicher?
---

# Sind Secrets in Kubernetes/OpenShift sicher?

**Kurzantwort: Ja – aber mit wichtigen Einschränkungen.**

Kubernetes- und OpenShift-Secrets sind für die Verwaltung sensibler Daten wie Passwörter, API-Keys oder TLS-Zertifikate gedacht. Sie sind jedoch **nicht automatisch vollständig abgesichert**. Allein das Anlegen eines Secrets bedeutet noch keine Verschlüsselung der Daten.

---

## Der häufigste Irrtum: Base64 ≠ Verschlüsselung

Secrets werden **Base64-kodiert** abgelegt – das ist reine Kodierung, **keine** Verschlüsselung.

Wer die Berechtigung hat, das Secret zu lesen, kann den Wert trivial dekodieren:

```bash
# OpenShift
oc get secret db-credentials -o yaml
echo "cGFzc3dvcnQxMjM=" | base64 -d      # -> passwort123

# Kubernetes
kubectl get secret db-credentials -o jsonpath='{.data.password}' | base64 -d
```

Base64 dient nur dazu, beliebige Binärdaten (z.B. Zertifikate) in einem YAML-Feld transportierbar zu machen – nicht dem Schutz.

---

## Was ist (mit korrekter Konfiguration) sicher?

- **Zugriffskontrolle (RBAC):** Nur Benutzer oder Service Accounts mit den passenden Rechten können ein Secret lesen. Das ist die *wichtigste* Schutzschicht.
- **Übertragung:** Die Kommunikation zwischen den Cluster-Komponenten (API Server, kubelet, etcd) läuft in der Regel über **TLS**.
- **Im Pod:** Als **Volume gemountete** Secrets werden im flüchtigen Speicher (`tmpfs`) bereitgestellt und nicht dauerhaft auf die Festplatte des Nodes geschrieben.

---

## Was ist *nicht* automatisch sicher?

### Speicherung in etcd

Der gesamte Cluster-Zustand – und damit auch Secrets – wird in **etcd** gespeichert.

- **Ohne** zusätzliche Konfiguration liegen Secrets dort **unverschlüsselt** (nur Base64-kodiert).
- Wer Zugriff auf das etcd-Backend oder ein etcd-Backup hat, kann die Werte auslesen.
- Kubernetes und OpenShift unterstützen **Encryption at Rest**: Ist sie aktiviert, werden Secrets vor dem Schreiben in etcd verschlüsselt (z.B. mit AES-CBC oder über einen KMS-Provider).

### Secrets als Umgebungsvariablen

Werden Secrets als **Umgebungsvariablen** in den Container gereicht, sind sie leichter ungewollt sichtbar:

- Sie erscheinen häufig in Logs, Crash-Dumps oder bei `oc describe pod` / Debug-Ausgaben.
- Kindprozesse erben die Umgebung.
- Als **Datei gemountet** (Volume) sind sie besser gekapselt und lassen sich zur Laufzeit rotieren, ohne den Pod neu zu starten.

### Secrets im Git / in Images

- Secrets gehören **nicht** im Klartext (oder Base64) in ein Git-Repository oder ein Container-Image.
- Für GitOps: Werkzeuge wie **Sealed Secrets**, **SOPS** oder ein **External Secrets Operator** verwenden, damit im Repo nur Verschlüsseltes bzw. eine Referenz liegt.

---

## Best Practices für produktive Umgebungen

- ✅ **Encryption at Rest** für etcd aktivieren.
- ✅ **RBAC restriktiv** konfigurieren (Least Privilege) – Zugriff nur für die Service Accounts, die das Secret wirklich brauchen.
- ✅ Secrets **als Datei mounten** statt als Umgebungsvariable, wo möglich.
- ✅ **Secret-Rotation** etablieren (regelmäßiger Wechsel, kurze Gültigkeiten).
- ✅ Secrets **nicht** im Klartext in Git oder Images ablegen (Sealed Secrets / SOPS / External Secrets).
- ✅ Für besonders kritische Daten einen **externen Secret Manager** nutzen – z.B. **HashiCorp Vault** oder den **Secrets Store CSI Driver** – für dynamische oder zentral verwaltete Secrets.

---

## Kubernetes vs. OpenShift – gibt es Unterschiede?

Das Secret-Objekt selbst ist in beiden identisch (`kind: Secret`). OpenShift ergänzt vor allem:

- **RBAC und Security Context Constraints (SCCs)** sind standardmäßig restriktiver.
- Encryption at Rest lässt sich über die OpenShift-Konfiguration (APIServer-Ressource) aktivieren.
- Zusätzliche integrierte Secret-Typen (z.B. für Image-Pull, Builds, Routes/TLS).

---

## Fazit

Kubernetes-/OpenShift-Secrets sind **sicher genug für die meisten Anwendungsfälle – vorausgesetzt, der Cluster ist korrekt konfiguriert**. Entscheidend sind:

- aktivierte **Encryption at Rest**,
- saubere **RBAC**-Berechtigungen,
- der bewusste Umgang mit **Env-Variablen vs. gemounteten Dateien**,
- und ggf. ein **externes Secret-Management** für besonders sensible Informationen.

> **Merksatz für die Schulung:** Ein Secret ist standardmäßig *versteckt, aber nicht verschlüsselt*. Sicherheit entsteht durch RBAC + Encryption at Rest, nicht durch das Secret-Objekt allein.

## Weiterführend

- [Kubernetes: Managing Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [Kubernetes: Encrypting Secret Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
- [OpenShift: Encrypting etcd data](https://docs.openshift.com/container-platform/latest/security/encrypting-etcd.html)
