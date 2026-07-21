---
layout: default
title: "Übung 8: Helm — Datenbank-Chart & erste Schritte"
---

# Datenbank als Helm Chart paketieren

Als Entwickler möchte ich zunächst die Datenbank als Helm Chart paketieren und dabei die Helm-CLI kennenlernen, damit ich ein erstes Deployment über Helm durchführen kann.

## 🎯 Lernziele

* Du kennst die Grundstruktur eines Helm Charts (`helm create`).
* Du kannst bestehende Kubernetes-Manifeste in Chart-Templates überführen.
* Du kannst Werte in `values.yaml` auslagern und in Templates referenzieren.
* Du kannst mit der Helm-CLI ein Chart prüfen (`helm lint`, `helm template`) und installieren (`helm install`, `helm status`, `helm list`).

## ✅ Definition of Done

* [ ] Die bisher manuell erstellten Ressourcen sind entfernt (sauberer Stand für Helm).
* [ ] Es existiert ein Chart `recipes-db` mit den Datenbank-Ressourcen als Templates.
* [ ] Einige Werte (z.B. Image-Tag, DB-Name/User/Passwort) sind in `values.yaml` ausgelagert.
* [ ] Mit einer zweiten Werte-Datei (`values-prod.yaml`) und `helm template -f` hast du gezeigt, dass dasselbe Chart je Umgebung andere Manifeste erzeugt.
* [ ] Die Datenbank ist per `helm install` deployt und der Pod läuft.

## 🪜 Arbeitsschritte

### 1. Bisherige Ressourcen aufräumen

Ab jetzt übernimmt **Helm** das Deployment der Anwendung. Damit es keine Konflikte mit den zuvor manuell erstellten Objekten gibt, räumen wir auf:

```bash
# Deployments, Services, Routes
oc delete deployment recipes-backend recipes-frontend postgres
oc delete service recipes-backend recipes-frontend postgres
oc delete route recipes-backend recipes-frontend

# ConfigMaps und Secrets
oc delete configmap frontend-config postgres-config postgres-init-sql
oc delete secret postgres-secret

# PersistentVolumeClaim (falls in Übung 6 angelegt)
oc delete pvc postgres-data

# Prüfen, ob alles weg ist
oc get all
```

> Fehlermeldungen zu bereits nicht mehr vorhandenen Objekten kannst Du ignorieren.

### 2. Erstes Chart erzeugen

Erstelle einen gemeinsamen Ordner und darin das DB-Chart:

```bash
mkdir helm-charts
cd helm-charts

helm create recipes-db
```

`helm create` erzeugt ein vollständiges Beispiel-Chart. **Schau Dir die generierten Dateien an** (`Chart.yaml`, `values.yaml`, `templates/`) — das ist der beste Einstieg in die Chart-Struktur.

### 3. Chart.yaml anpassen

Passe `recipes-db/Chart.yaml` an:

```yaml
apiVersion: v2
name: recipes-db
description: PostgreSQL-Datenbank für die Rezepte-Anwendung
version: 0.1.0
appVersion: "17"
type: application
```

### 4. Deine DB-Manifeste als Templates übernehmen

Lösche die von `helm create` generierten Beispiel-Templates im Ordner `recipes-db/templates/` und lege stattdessen deine Datenbank-Manifeste aus Übung 4 dort ab (Secret, ConfigMap für den DB-Namen, ConfigMap fürs Init-SQL, Deployment, Service). Die generierte `values.yaml` kannst Du zunächst leeren.

### 5. Chart prüfen (erste Helm-CLI-Schritte)

Prüfe das Chart auf Syntaxfehler und sieh dir die gerenderten Manifeste an, **ohne** etwas zu installieren:

```bash
helm lint ./recipes-db
helm template test-release ./recipes-db
```

### 6. Werte in values.yaml auslagern

Jetzt kommt der eigentliche Mehrwert: Ersetze hart kodierte Werte in den Templates durch Referenzen auf `values.yaml`. Beginne mit ein paar wichtigen Werten — nicht alles auf einmal.

`recipes-db/values.yaml`:

```yaml
image:
  repository: postgres
  tag: "17-alpine"
replicas: 1
database:
  name: recipes
  user: recipes
  password: recipes-secret-pw
```

In `recipes-db/templates/deployment.yaml` ersetzt du dann z.B.:

{% raw %}
```yaml
# Vorher:
image: postgres:17-alpine

# Nachher:
image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
```
{% endraw %}

Und im Secret (Helm kann Klartext über `stringData` verwenden):

{% raw %}
```yaml
stringData:
  POSTGRES_USER: {{ .Values.database.user }}
  POSTGRES_PASSWORD: {{ .Values.database.password }}
```
{% endraw %}

Nach jeder Änderung erneut prüfen:

```bash
helm lint ./recipes-db
helm template test-release ./recipes-db
```

### 7. Parameterisierung sichtbar machen: zweite Umgebung

Der eigentliche Nutzen der Auslagerung zeigt sich, wenn **dieselben Templates** mit **unterschiedlichen Werten** gerendert werden. Lege dafür eine zweite Werte-Datei für eine andere Umgebung an — hier für die Produktion. Sie enthält nur die **Abweichungen** von der Standard-`values.yaml`:

`recipes-db/values-prod.yaml`:

```yaml
image:
  tag: "17"
replicas: 3
database:
  name: recipes_prod
```

Rendere das Chart nun einmal mit den Standardwerten und einmal mit der Prod-Datei und **vergleiche die Ausgabe**:

```bash
# Standard (values.yaml)
helm template test-release ./recipes-db

# Mit Overlay für die Produktion
helm template test-release ./recipes-db -f ./recipes-db/values-prod.yaml
```

Die mit `-f` übergebene Datei **überschreibt** nur die dort gesetzten Werte; alles andere kommt weiterhin aus der `values.yaml`. So erzeugt **ein** Chart je nach Umgebung unterschiedliche Manifeste (anderes Image-Tag, mehr Replicas, anderer DB-Name).

> Beim Installieren funktioniert das genauso: `helm install recipes-db ./recipes-db -f ./recipes-db/values-prod.yaml`.

### 8. Erstes Deployment über Helm

Installiere die Datenbank als Helm-Release:

```bash
helm install recipes-db ./recipes-db
```

Beobachte den Fortschritt und die installierten Releases:

```bash
oc get pods -w
helm list
helm status recipes-db
```

Prüfe die Logs — das Init-SQL sollte ausgeführt worden sein:

```bash
oc logs deployment/postgres
```

## 📚 Selbstlernmaterial

* [Helm Charts](../../docs/helm.html) — Konzepte, OCI-Support, Architektur
* [Aufbau eines Helm Charts](../../docs/helm-chart-struktur.html) — Alle Dateien und Verzeichnisse im Detail
* [Helm Cheat Sheet](../../docs/helm-cheatsheet.html) — Die wichtigsten Helm-Befehle
* [Helm Template-Guide](https://helm.sh/docs/chart_template_guide/)

## 🤔 Reflexionsfragen

* Welche Werte lohnt es sich, in die `values.yaml` auszulagern? Welche können hart kodiert bleiben?
* Welche Vorteile hat ein Helm Chart gegenüber den einzelnen YAML-Manifesten, die wir zuvor verwendet haben?
* Warum mussten wir die bisherigen Ressourcen löschen, bevor wir mit Helm installiert haben?
* Wofür ist `helm template` nützlich, bevor man `helm install` ausführt?
* Wie hilft eine zweite Werte-Datei (`-f values-prod.yaml`) dabei, dieselbe Anwendung in mehreren Umgebungen zu betreiben?
