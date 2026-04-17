---
layout: default
title: "Übung 6: Persistent Volumes"
---

# Persistenten Speicher für PostgreSQL einrichten

Als Entwickler möchte ich die PostgreSQL-Datenbank mit einem PersistentVolume betreiben, damit die Daten auch nach einem Pod-Neustart oder -Umzug erhalten bleiben.

## 🎯 Lernziele

* Du verstehst den Unterschied zwischen `emptyDir` und PersistentVolumes.
* Du kannst einen PersistentVolumeClaim (PVC) erstellen und in einem Deployment verwenden.
* Du weißt, was eine StorageClass ist und wie dynamisches Provisioning funktioniert.
* Du kannst überprüfen, ob Daten nach einem Pod-Neustart erhalten bleiben.

## ✅ Definition of Done

* [ ] Du hast einen PersistentVolumeClaim für die PostgreSQL-Daten erstellt.
* [ ] Du hast das PostgreSQL-Deployment so angepasst, dass es den PVC statt `emptyDir` verwendet.
* [ ] Du kannst Daten in die Datenbank schreiben, den Pod neu starten und die Daten sind danach noch vorhanden.

## 🪜 Arbeitsschritte

### 1. Aktuelle Situation verstehen

In Übung 4 haben wir für das PostgreSQL-Deployment ein `emptyDir`-Volume verwendet:

```yaml
volumes:
  - name: data
    emptyDir: {}
```

Ein `emptyDir`-Volume ist an die Lebensdauer des **Pods** gebunden — wird der Pod gelöscht oder neu gestartet, sind die Daten weg. Für eine Datenbank ist das nicht akzeptabel.

### 2. Verfügbare StorageClasses prüfen

Prüfe, welche StorageClasses im Cluster verfügbar sind:

```bash
kubectl get storageclass
```

In Minikube ist standardmäßig die StorageClass `standard` vorhanden, die dynamisches Provisioning unterstützt. Das bedeutet: wenn ein PVC erstellt wird, legt Kubernetes automatisch ein passendes PV an.

### 3. PersistentVolumeClaim erstellen

Erstelle eine Datei `postgres-pvc.yaml`:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

**Erklärung:**

* `ReadWriteOnce` — das Volume kann von einem einzelnen Node gelesen und beschrieben werden (ausreichend für eine einzelne PostgreSQL-Instanz).
* `storage: 1Gi` — wir fordern 1 GiB Speicher an.
* Da wir keine `storageClassName` angeben, wird die Default-StorageClass verwendet.

```bash
kubectl apply -f postgres-pvc.yaml
```

Prüfe den Status des PVCs:

```bash
kubectl get pvc postgres-data
```

Der Status sollte `Bound` sein, sobald ein PV zugewiesen wurde.

### 4. PostgreSQL-Deployment anpassen

Ersetze im PostgreSQL-Deployment das `emptyDir`-Volume durch eine Referenz auf den PVC:

```yaml
volumes:
  - name: init-sql
    configMap:
      name: postgres-init-sql
  - name: data
    persistentVolumeClaim:
      claimName: postgres-data
```

Wende das geänderte Manifest an:

```bash
kubectl apply -f postgres-deployment.yaml
```

### 5. Persistenz testen

Überprüfe, ob die Daten einen Pod-Neustart überleben:

**a) Daten in die Datenbank schreiben:**

Erstelle über das Frontend oder die Backend-API ein neues Rezept.

```bash
# Alternativ direkt über die API:
curl -X POST http://<backend-url>/recipes \
  -H "Content-Type: application/json" \
  -d '{"name":"Testrezept","servings":2,"duration":30,"difficulty":"EASY","preparation":"Nur ein Test."}'
```

**b) Pod löschen (Kubernetes erstellt automatisch einen neuen):**

```bash
kubectl delete pod -l app=postgres
```

**c) Warten, bis der neue Pod läuft:**

```bash
kubectl get pods -l app=postgres -w
```

**d) Prüfen, ob die Daten noch da sind:**

```bash
curl http://<backend-url>/recipes | jq
```

Das Testrezept sollte noch vorhanden sein.

### 6. PV und PVC inspizieren

Schau dir die erstellten Ressourcen genauer an:

```bash
# PVC anzeigen
kubectl get pvc postgres-data -o yaml

# Zugehöriges PV anzeigen
kubectl get pv
```

Beachte die Felder `spec.volumeName` im PVC (Verweis auf das PV) und `persistentVolumeReclaimPolicy` im PV (was passiert mit dem PV, wenn der PVC gelöscht wird).

## 📚 Selbstlernmaterial

* [Kubernetes-Ressourcen: Speicher](../../docs/kubernetes-resources.html) — PersistentVolume, PersistentVolumeClaim und StorageClass
* [PostgreSQL auf Kubernetes – Best Practices](../../docs/postgres-as-statefulset.html) — StatefulSet, Operatoren und produktionsreife Setups
* [Kubernetes: Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
* [Kubernetes: Storage Classes](https://kubernetes.io/docs/concepts/storage/storage-classes/)
* [Kubernetes: Dynamic Volume Provisioning](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)

## 🤔 Reflexionsfragen

* Was ist der Unterschied zwischen einem PersistentVolume (PV) und einem PersistentVolumeClaim (PVC)? Warum gibt es diese Trennung?
* Was bedeuten die Access Modes `ReadWriteOnce`, `ReadOnlyMany` und `ReadWriteMany`? Welcher ist für PostgreSQL geeignet und warum?
* Was passiert mit den Daten, wenn der PVC gelöscht wird? Welche Rolle spielt die `persistentVolumeReclaimPolicy`?
* Warum ist für eine produktive PostgreSQL-Instanz ein StatefulSet mit `volumeClaimTemplates` besser geeignet als ein Deployment mit einem manuell erstellten PVC?
* Welche Vorteile bietet dynamisches Provisioning gegenüber manuell angelegten PersistentVolumes?
