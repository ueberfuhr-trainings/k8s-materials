---
layout: default
title: "Übung 7: Helm Charts"
---

# Anwendung als Helm Charts paketieren

Als Entwickler möchte ich die gesamte Rezepte-Anwendung (Datenbank, Backend, Frontend) als Helm Charts paketieren, damit die Anwendung mit einem einzigen Befehl installiert werden kann.

## 🎯 Lernziele

* Du kannst ein Helm Chart mit `helm create` erzeugen und an deine Bedürfnisse anpassen.
* Du kannst bestehende Kubernetes-Manifeste in ein Chart überführen.
* Du verstehst, wie man Werte in `values.yaml` auslagert und in Templates referenziert.
* Du kannst die gesamte Anwendung mit `helm install` deployen.

## ✅ Definition of Done

* [ ] Du hast drei Helm Charts erstellt: `recipes-db`, `recipes-backend`, `recipes-frontend`.
* [ ] Jedes Chart enthält die notwendigen Kubernetes-Ressourcen als Templates.
* [ ] Mindestens einige Werte (z.B. Image-Tag, Replicas) sind in `values.yaml` ausgelagert.
* [ ] Die Charts referenzieren sich gegenseitig als lokale Dependencies.
* [ ] Die gesamte Anwendung lässt sich mit `helm install` über das Frontend-Chart deployen.

## 🪜 Arbeitsschritte

### 1. Bisherige Ressourcen aufräumen

**Wichtig:** Bevor wir mit Helm deployen, müssen alle manuell erstellten Ressourcen entfernt werden, damit es keine Konflikte gibt:

```bash
# Deployments
oc delete deployment recipes-backend recipes-frontend postgres

# Services
oc delete service recipes-backend recipes-frontend postgres

# Routes
oc delete route recipes-backend recipes-frontend

# ConfigMaps und Secrets
oc delete configmap frontend-config postgres-config postgres-init-sql
oc delete secret postgres-secret

# Prüfen, ob alles weg ist
oc get all
oc get configmap
oc get secret
```

### 2. Chart-Grundgerüste erzeugen

Erstelle einen gemeinsamen Ordner und darin drei Charts:

```bash
mkdir helm-charts
cd helm-charts

helm create recipes-db
helm create recipes-backend
helm create recipes-frontend
```

`helm create` erzeugt ein vollständiges Beispiel-Chart. Schau Dir die generierten Dateien gern an.

### 3. Chart.yaml anpassen

Passe die `Chart.yaml` jedes Charts an. Beispiel für `recipes-db/Chart.yaml`:

```yaml
apiVersion: v2
name: recipes-db
description: PostgreSQL-Datenbank für die Rezepte-Anwendung
version: 0.1.0
appVersion: "17"
type: application
```

### 4. YAML-Manifeste übernehmen

Kopiere deine bestehenden YAML-Manifeste in den `templates/`-Ordner des jeweiligen Charts. Bereits bestehende YAML-Dateien kannst Du löschen bzw. überschreiben.

### 5. Charts prüfen

Prüfe jedes Chart auf Syntaxfehler:

```bash
helm lint ./recipes-db
helm lint ./recipes-backend
helm lint ./recipes-frontend
```

Du kannst dir auch die gerenderten Templates anschauen, ohne etwas zu installieren:

```bash
helm template test-release ./recipes-db
```

### 6. Werte in values.yaml auslagern

Jetzt kommt der eigentliche Mehrwert: Ersetze hart kodierte Werte in den Templates durch Referenzen auf `values.yaml`. Beginne mit ein paar wichtigen Werten — nicht alles auf einmal.

**Beispiel für `recipes-db`:**

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

Und in `recipes-db/templates/secret.yaml`:

{% raw %}
```yaml
# Vorher:
data:
  POSTGRES_USER: cmVjaXBlcw==
  POSTGRES_PASSWORD: cmVjaXBlcy1zZWNyZXQtcHc=

# Nachher (mit stringData statt data — Helm kann Klartext verwenden):
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

### 7. Charts als Pakete erzeugen (optional)

Du kannst jedes Chart als `.tgz`-Archiv verpacken:

```bash
helm package ./recipes-db
helm package ./recipes-backend
helm package ./recipes-frontend
```

Das erzeugt Dateien wie `recipes-db-0.1.0.tgz`. So könnten Charts in ein Repository oder eine OCI-Registry gepusht werden.

### 8. Abhängigkeiten definieren

Das Frontend braucht das Backend, und das Backend braucht die Datenbank. Definiere das über lokale `file://`-Referenzen in der `Chart.yaml`.

**`recipes-backend/Chart.yaml`** :

```yaml
dependencies:
  - name: recipes-db
    version: "0.1.0"
    repository: "file://../recipes-db"
```

**`recipes-frontend/Chart.yaml`**:

```yaml
dependencies:
  - name: recipes-backend
    version: "0.1.0"
    repository: "file://../recipes-backend"
```

### 9. Abhängigkeiten auflösen

Lade die Dependencies herunter — die Reihenfolge ist wichtig, weil die Charts aufeinander aufbauen:

```bash
# 1. DB hat keine Dependencies — nichts zu tun
helm dependency update ./recipes-db

# 2. Backend hängt von DB ab
helm dependency update ./recipes-backend

# 3. Frontend hängt von Backend ab (das wiederum DB mitbringt)
helm dependency update ./recipes-frontend
```

Nach jedem Befehl erscheint eine `Chart.lock`-Datei und im `charts/`-Ordner das jeweilige Dependency-Archiv.

### 10. Alles installieren

Ein einziger Befehl installiert jetzt die gesamte Anwendung — Frontend, Backend und Datenbank:

```bash
helm install recipes ./recipes-frontend
```

Beobachte den Fortschritt:

```bash
oc get pods -w
helm status recipes
```

Prüfe, ob die Anwendung funktioniert:

```bash
oc get routes
curl -s https://<backend-route-url>/recipes | jq
```

## 📚 Selbstlernmaterial

* [Helm Charts](../../docs/helm.html) — Konzepte, OCI-Support, Architektur
* [Aufbau eines Helm Charts](../../docs/helm-chart-struktur.html) — Alle Dateien und Verzeichnisse im Detail
* [Helm Cheat Sheet](../../docs/helm-cheatsheet.html) — Die wichtigsten Helm-Befehle
* [Helm Template-Guide](https://helm.sh/docs/chart_template_guide/)
* [Helm Dependency Management](https://helm.sh/docs/helm/helm_dependency/)

## 🤔 Reflexionsfragen

* Welche Werte lohnt es sich, in die `values.yaml` auszulagern? Welche können hart kodiert bleiben?
* Welche Vorteile hat die Nutzung von Helm Charts gegenüber den einzelnen YAML-Manifesten, die wir zuvor verwendet haben?
* Warum mussten wir die bisherigen Ressourcen löschen, bevor wir mit Helm installiert haben?
* Was wäre der Vorteil, die Charts in eine OCI-Registry zu pushen, statt sie nur lokal zu verwenden?
* Wie würdest du unterschiedliche Konfigurationen für Dev und Prod abbilden?
