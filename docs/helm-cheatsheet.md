---
layout: default
title: Helm Cheat Sheet
---

# Helm Cheat Sheet

Kurzreferenz für die wichtigsten Helm-Befehle. Wo sich Befehle zwischen klassischen Repositories und OCI-Registries unterscheiden, sind beide Varianten aufgeführt.

> **OCI-Support** ist ab Helm 3.8 stabil verfügbar. Für ältere Versionen stehen nur die klassischen Repository-Befehle zur Verfügung.

## Repository-Verwaltung

### Klassisch (alle Helm-Versionen)

```bash
# Repository hinzufügen
helm repo add bitnami https://charts.bitnami.com/bitnami

# Repositories aktualisieren
helm repo update

# Repositories auflisten
helm repo list

# Repository entfernen
helm repo remove bitnami
```

### OCI (ab Helm 3.8)

OCI-Registries brauchen kein `helm repo add` — man referenziert Charts direkt über die Registry-URL.

```bash
# Bei einer OCI-Registry anmelden
helm registry login ghcr.io
helm registry login docker.io

# Abmelden
helm registry logout ghcr.io
```

## Charts suchen

### Klassisch

```bash
# In hinzugefügten Repos suchen
helm search repo postgresql

# Alle Versionen anzeigen
helm search repo postgresql --versions

# Auf Artifact Hub suchen
helm search hub postgresql
```

### OCI

```bash
# OCI-Registries haben keinen eingebauten Suchbefehl.
# Stattdessen: Registry-UI oder Artifact Hub nutzen.
```

## Charts herunterladen und inspizieren

### Klassisch

```bash
# Chart herunterladen (als .tgz)
helm pull bitnami/postgresql

# Chart herunterladen und entpacken
helm pull bitnami/postgresql --untar

# Chart-Infos anzeigen
helm show chart bitnami/postgresql

# Standard-Values anzeigen
helm show values bitnami/postgresql
```

### OCI (ab Helm 3.8)

```bash
# Chart aus OCI-Registry herunterladen
helm pull oci://ghcr.io/myorg/my-app --version 1.0.0

# Chart herunterladen und entpacken
helm pull oci://ghcr.io/myorg/my-app --version 1.0.0 --untar

# Chart-Infos anzeigen
helm show chart oci://ghcr.io/myorg/my-app --version 1.0.0

# Standard-Values anzeigen
helm show values oci://ghcr.io/myorg/my-app --version 1.0.0
```

## Charts veröffentlichen

### Klassisch

```bash
# Chart als .tgz verpacken
helm package ./my-app

# Upload zum Repository hängt vom Repo-Typ ab (z.B. ChartMuseum API, GitHub Pages, Nexus)
```

### OCI (ab Helm 3.8)

```bash
# Chart als .tgz verpacken
helm package ./my-app

# Chart in OCI-Registry pushen
helm push my-app-1.0.0.tgz oci://ghcr.io/myorg
```

## Charts installieren

```bash
# Aus lokalem Verzeichnis
helm install my-release ./my-app

# Aus klassischem Repo
helm install my-release bitnami/postgresql

# Aus OCI-Registry (ab Helm 3.8)
helm install my-release oci://ghcr.io/myorg/my-app --version 1.0.0

# Mit eigenen Values
helm install my-release ./my-app -f prod-values.yaml

# Einzelne Werte überschreiben
helm install my-release ./my-app --set replicas=3 --set image.tag=1.2.3

# In bestimmtem Namespace
helm install my-release ./my-app -n my-namespace

# Namespace automatisch erstellen
helm install my-release ./my-app -n my-namespace --create-namespace

# Dry-Run (nur rendern, nicht installieren)
helm install my-release ./my-app --dry-run
```

## Releases verwalten

```bash
# Installierte Releases auflisten
helm list

# Alle Namespaces
helm list -A

# Release-Status anzeigen
helm status my-release

# Aktuelle Values eines Release anzeigen
helm get values my-release

# Alle generierten Manifeste eines Release anzeigen
helm get manifest my-release

# Release-Historie anzeigen
helm history my-release
```

## Upgrade und Rollback

```bash
# Release upgraden (lokales Chart)
helm upgrade my-release ./my-app

# Release upgraden (mit geänderten Values)
helm upgrade my-release ./my-app -f prod-values.yaml

# Upgrade mit Install-Fallback (installiert, falls noch nicht vorhanden)
helm upgrade --install my-release ./my-app

# Rollback auf vorherige Revision
helm rollback my-release

# Rollback auf bestimmte Revision
helm rollback my-release 2
```

## Release entfernen

```bash
# Release deinstallieren (entfernt alle Kubernetes-Ressourcen)
helm uninstall my-release

# In bestimmtem Namespace
helm uninstall my-release -n my-namespace
```

## Chart entwickeln

```bash
# Neues Chart-Gerüst erstellen
helm create my-app

# Chart-Syntax prüfen
helm lint ./my-app

# Templates lokal rendern (ohne Cluster)
helm template my-release ./my-app

# Templates mit eigenen Values rendern
helm template my-release ./my-app -f prod-values.yaml

# Dependencies (Subcharts) herunterladen
helm dependency update ./my-app

# Dependencies auflisten
helm dependency list ./my-app
```

## Befehlsübersicht: Klassisch vs. OCI

| Aktion | Klassisch | OCI (ab 3.8) |
|--------|-----------|---------------|
| Repo einrichten | `helm repo add NAME URL` | `helm registry login REGISTRY` |
| Suchen | `helm search repo KEYWORD` | Registry-UI / Artifact Hub |
| Herunterladen | `helm pull REPO/CHART` | `helm pull oci://REGISTRY/CHART --version X` |
| Veröffentlichen | Repo-spezifisch | `helm push CHART.tgz oci://REGISTRY` |
| Installieren | `helm install REL REPO/CHART` | `helm install REL oci://REGISTRY/CHART --version X` |
| Chart-Info | `helm show chart REPO/CHART` | `helm show chart oci://REGISTRY/CHART --version X` |
