---
layout: default
title: "Übung 9: Helm — Backend & Frontend"
---

# Backend (und Frontend) als Helm Charts mit Abhängigkeiten

Als Entwickler möchte ich das Backend – und optional das Frontend – als Helm Charts paketieren und über Chart-Abhängigkeiten mit der Datenbank verknüpfen, damit sich die Anwendung mit einem einzigen Befehl installieren lässt.

> **Voraussetzung:** [Übung 8](08-helm-charts.html) ist abgeschlossen — das Chart `recipes-db` existiert und ist als Release `recipes-db` installiert.

## 🎯 Lernziele

* Du kannst weitere Charts erstellen und bestehende Manifeste in Templates überführen.
* Du kannst **Chart-Abhängigkeiten** definieren (`dependencies`, `helm dependency update`).
* Du kannst ein „Umbrella"-Release installieren, das mehrere Charts mitbringt.

## ✅ Definition of Done

* [ ] Es existiert ein Chart `recipes-backend` mit einer Abhängigkeit auf `recipes-db`.
* [ ] `helm dependency update` wurde ausgeführt (Dependency liegt im `charts/`-Ordner).
* [ ] Backend **und** Datenbank sind über **ein** Helm-Release installiert und die Anwendung ist erreichbar.
* [ ] *(Optional)* Es existiert ein Chart `recipes-frontend` mit Abhängigkeit auf `recipes-backend`, über das die gesamte Anwendung installiert wird.

## 🪜 Arbeitsschritte

### 1. Backend-Chart erstellen

Erzeuge im Ordner `helm-charts` das Backend-Chart und überführe deine Backend-Manifeste (Deployment mit DB-Umgebungsvariablen, Service, Ingress) analog zu Übung 8 in die Templates:

```bash
cd helm-charts
helm create recipes-backend
```

* `Chart.yaml` anpassen (Name, Version, Beschreibung).
* Beispiel-Templates löschen, deine Backend-Manifeste in `templates/` ablegen.
* Einige Werte (Image-Tag, Replicas) in `values.yaml` auslagern und `helm lint` / `helm template` zum Prüfen nutzen.

### 2. Abhängigkeit auf die Datenbank definieren

Das Backend braucht die Datenbank. Definiere das als lokale Abhängigkeit in `recipes-backend/Chart.yaml`:

```yaml
dependencies:
  - name: recipes-db
    version: "0.1.0"
    repository: "file://../recipes-db"
```

Löse die Abhängigkeit auf — Helm kopiert das DB-Chart nach `recipes-backend/charts/`:

```bash
helm dependency update ./recipes-backend
```

### 3. Standalone-Datenbank aus Übung 8 entfernen

Damit die Datenbank nicht doppelt läuft (einmal als Release `recipes-db`, einmal als Abhängigkeit des Backends), das eigenständige Release aus Übung 8 deinstallieren:

```bash
helm uninstall recipes-db
```

### 4. Backend + Datenbank in einem Release installieren

```bash
helm install recipes ./recipes-backend
```

Beobachten und testen (`minikube tunnel` muss laufen):

```bash
kubectl get pods -w
helm status recipes
kubectl get ingress recipes-backend
curl -i http://<backend-ingress-url>/recipes
```

Backend und Datenbank kommen jetzt aus **einem** Release.

### 5. *(Optional)* Frontend ergänzen

Baue analog ein Chart `recipes-frontend` (Deployment mit `API_BASE_URL`, Service, Ingress) und definiere darin die Abhängigkeit auf das Backend:

```yaml
# recipes-frontend/Chart.yaml
dependencies:
  - name: recipes-backend
    version: "0.1.0"
    repository: "file://../recipes-backend"
```

```bash
helm dependency update ./recipes-frontend
```

Das Frontend-Chart ist jetzt das oberste „Umbrella"-Chart (Frontend → Backend → DB). Ersetze das bisherige Release dadurch:

```bash
helm uninstall recipes
helm install recipes ./recipes-frontend
```

Ein einziger Befehl installiert damit die **gesamte** Anwendung. Prüfen:

```bash
kubectl get pods -w
kubectl get ingress
```

## 📚 Selbstlernmaterial

* [Helm Dependency Management](https://helm.sh/docs/helm/helm_dependency/)
* [Helm Charts](../../docs/helm.html) — Konzepte, OCI-Support, Architektur
* [Aufbau eines Helm Charts](../../docs/helm-chart-struktur.html) — Alle Dateien und Verzeichnisse im Detail
* [Helm Cheat Sheet](../../docs/helm-cheatsheet.html) — Die wichtigsten Helm-Befehle

## 🤔 Reflexionsfragen

* Warum wird die Datenbank als Abhängigkeit des Backends definiert und nicht andersherum?
* Was passiert bei `helm install` mit den Ressourcen aus den abhängigen Charts?
* Warum mussten wir das eigenständige `recipes-db`-Release deinstallieren, bevor wir das Umbrella-Release installiert haben?
* Wie würdest du unterschiedliche Konfigurationen für Dev und Prod abbilden (z.B. eigene `values`-Dateien)?
* Was wäre der Vorteil, die Charts in eine OCI-Registry zu pushen, statt sie nur lokal zu verwenden?
