---
layout: default
title: Helm Charts
---

# Helm Charts

Helm ist der **Paketmanager für Kubernetes**. Ähnlich wie `apt` für Debian oder `npm` für Node.js bündelt Helm Kubernetes-Ressourcen in wiederverwendbare Pakete — sogenannte **Charts**.

## Warum Helm?

### Das Problem ohne Helm

Wer eine Anwendung auf Kubernetes deployt, erstellt dafür mehrere YAML-Manifeste: Deployment, Service, ConfigMap, Secret, Route, PersistentVolumeClaim usw. Dabei ergeben sich schnell Herausforderungen:

- **Redundanz:** Dieselbe Anwendung wird in Dev, Staging und Prod deployed — mit leicht unterschiedlicher Konfiguration. Man pflegt 3 Sätze nahezu identischer YAML-Dateien.
- **Versionierung:** Welche Version der Manifeste läuft gerade in Produktion? Ein Rollback erfordert, die richtigen YAML-Dateien wiederzufinden.
- **Abhängigkeiten:** Eine Anwendung braucht eine Datenbank. Beide müssen zusammen deployed werden, in der richtigen Reihenfolge, mit aufeinander abgestimmter Konfiguration.
- **Weitergabe:** Ein Team will seine Anwendung anderen zur Verfügung stellen. Dafür müsste es eine Sammlung loser YAML-Dateien plus eine Installationsanleitung liefern.

### Was Helm löst

| Ohne Helm | Mit Helm |
|-----------|----------|
| Viele lose YAML-Dateien | Ein Chart = ein Paket |
| Werte sind hart kodiert | Parametrisiert über `values.yaml` |
| Kein Überblick über Releases | `helm list` zeigt installierte Releases |
| Rollback = manuelles YAML tauschen | `helm rollback` auf frühere Revision |
| Kein Paketformat | Charts sind teilbar, versioniert, signierbar |
| Abhängigkeiten manuell | `Chart.yaml` deklariert Dependencies |

## Kernkonzepte

### Chart

Ein Chart ist ein **Paket aus Templates und Standardwerten**, das eine Anwendung beschreibt.

#### Chart-Struktur

```
my-app/
├── Chart.yaml            # Metadaten und Abhängigkeiten (erforderlich)
├── Chart.lock            # Exakte Versionen der aufgelösten Dependencies
├── values.yaml           # Standardwerte (überschreibbar beim Install)
├── values.schema.json    # JSON Schema zur Validierung der Values
├── .helmignore           # Dateien, die beim Verpacken ignoriert werden
├── LICENSE               # Lizenz des Charts
├── README.md             # Dokumentation des Charts
├── templates/            # Kubernetes-Manifeste als Go-Templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── _helpers.tpl      # Wiederverwendbare Template-Snippets
│   ├── NOTES.txt         # Hinweistext nach Install/Upgrade
│   └── tests/            # Helm-Tests
├── charts/               # Subcharts (Dependencies)
└── crds/                 # Custom Resource Definitions
```

Detaillierte Beschreibung aller Dateien, Beispiele für `Chart.yaml`, `NOTES.txt`, `_helpers.tpl`, CRDs und mehr: **[Aufbau eines Helm Charts](helm-chart-struktur.html)**

### Release

Ein **Release** ist eine konkrete Installation eines Charts in einem Cluster. Dasselbe Chart kann mehrfach installiert werden — z.B. als `my-app-dev` und `my-app-prod`. Jedes Release hat eine eigene Revisionshistorie für Upgrades und Rollbacks.

### Values

**Values** parametrisieren ein Chart. Sie werden in `values.yaml` definiert und können beim Installieren überschrieben werden:

```yaml
# values.yaml
image:
  repository: ralfueberfuhr/recipes-backend
  tag: latest-dev
replicas: 1
```

```yaml
# templates/deployment.yaml (Auszug)
spec:
  replicas: {{ .Values.replicas }}
  template:
    spec:
      containers:
        - name: backend
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
```

Beim Installieren können Werte überschrieben werden:

```bash
helm install my-release ./my-app --set replicas=3
# oder
helm install my-release ./my-app -f prod-values.yaml
```

### Repository — klassisch und OCI

Es gibt **zwei Wege**, um Charts zu verteilen:

#### Klassische Chart-Repositories (Legacy)

Ein klassisches Repository ist ein HTTP-Server, der eine `index.yaml` mit den verfügbaren Charts bereitstellt. Dafür braucht man einen separaten Server (ChartMuseum, GitHub Pages, Nexus o.ä.).

```
┌─────────────────────────────┐
│     Chart Repository        │
│                             │
│  index.yaml                 │
│  ├── my-app-1.0.0.tgz      │
│  ├── my-app-1.1.0.tgz      │
│  └── postgresql-15.2.0.tgz │
└─────────────────────────────┘
```

Viele bestehende Charts (z.B. Bitnami) werden noch über klassische Repositories verteilt.

#### OCI-Registries (ab Helm 3.8 — empfohlen)

> **Ab Helm 3.8 können Charts direkt in Container-Registries gespeichert werden.** Das ist der empfohlene Weg für neue Setups und wird das klassische Repository-Modell langfristig ablösen.

**Was ist OCI in diesem Kontext?**

**OCI** steht für **Open Container Initiative** — eine Organisation, die offene Standards für Container-Formate und -Registries definiert. Die OCI spezifiziert unter anderem die **OCI Distribution Specification** — ein Protokoll, über das Registries beliebige Artefakte speichern und verteilen können, nicht nur Container-Images.

Helm nutzt genau dieses Protokoll: **Helm Charts werden als OCI-Artefakte** in jeder OCI-kompatiblen Registry gespeichert — also in denselben Registries, die auch Container-Images hosten (Docker Hub, GitHub Container Registry, Quay.io, Harbor, ACR, ECR usw.).

**Wichtige Abgrenzung:**
- **OCI als Organisation** definiert Standards (Image-Format, Runtime, Distribution)
- **OCI Distribution Spec** ist das Protokoll, das Helm nutzt
- Ein Helm Chart ist **kein Container-Image**, aber es wird über dasselbe Protokoll und in derselben Registry gespeichert
- Man braucht **keine separate Infrastruktur** für Charts — die bestehende Container-Registry reicht

#### Vergleich

| | Klassisches Repository | OCI-Registry |
|---|---|---|
| **Protokoll** | Eigenes Helm-Protokoll (HTTP + `index.yaml`) | OCI Distribution Spec |
| **Infrastruktur** | Separater Chart-Server nötig | Bestehende Container-Registry |
| **Suche** | `helm search repo` | Registry-UI oder Artifact Hub |
| **Helm-Version** | Alle | Ab 3.8 (stabil) |
| **Status** | Legacy — funktioniert, aber nicht mehr empfohlen | **Empfohlen für neue Setups** |

#### Architektur im Überblick

```
                    Entwickler
                       │
                 helm push / pull
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
 ┌──────────────────┐    ┌──────────────────┐
 │  Klassisches     │    │  OCI-Registry    │
 │  Chart-Repo      │    │  (z.B. GHCR)     │
 │  (Legacy)        │    │                  │
 │  index.yaml      │    │  my-app:1.0.0    │  ← Chart als OCI-Artefakt
 │  my-app-1.0.0.tgz│    │  postgres:15.2   │  ← neben Container-Images
 └──────────────────┘    └──────────────────┘
          │                         │
          └────────────┬────────────┘
                       ▼
               Kubernetes-Cluster
                  helm install
```

## Helm und OpenShift

OpenShift unterstützt Helm vollständig:

- **Helm CLI** funktioniert identisch wie bei Standard-Kubernetes (`helm install`, `helm upgrade` etc.)
- **Web Console** hat eine integrierte Helm-Ansicht: installierte Releases anzeigen, Werte ändern, Upgrades durchführen
- **Developer Catalog** listet Helm Charts aus konfigurierten Repositories — Teams können Charts direkt aus der Web Console installieren
- **Integrierte Registry** (Quay.io / interne Registry) unterstützt OCI-Artefakte und damit Helm Charts

Helm-Templates können OpenShift-spezifische Ressourcen (Routes, SecurityContextConstraints) enthalten — sie sind gewöhnliche Kubernetes-YAML-Templates.

## Vorteile

- **Wiederverwendbarkeit:** Ein Chart, viele Umgebungen. Unterschiede nur in den Values.
- **Versionierung und Rollback:** Jedes Release hat eine Revisionshistorie. Rollback auf eine frühere Version mit einem Befehl.
- **Abhängigkeitsverwaltung:** Charts können andere Charts als Dependency deklarieren (z.B. Backend-Chart hängt von PostgreSQL-Chart ab).
- **Ökosystem:** Tausende öffentlicher Charts (Bitnami, Artifact Hub) für Standardsoftware — PostgreSQL, Redis, NGINX, Prometheus etc.
- **Standardisierung:** Ein einheitliches Format, das Teams, CI/CD-Pipelines und Cluster-Admins verstehen.

## Herausforderungen

- **Lernkurve:** Go-Templating (`{{ if }}`, `{{ range }}`, `{{ include }}`) ist mächtig, aber die Syntax gewöhnungsbedürftig — besonders bei verschachtelten Templates und Whitespace-Kontrolle.
- **Debugging:** Fehler in Templates erzeugen oft kryptische Fehlermeldungen. `helm template` und `helm lint` helfen, aber die Fehlersuche bleibt aufwändiger als bei purem YAML.
- **Komplexität bei großen Charts:** Umbrella-Charts mit vielen Subcharts können schwer überschaubar werden. Values-Overrides über mehrere Ebenen sind fehleranfällig.
- **Kein GitOps out of the box:** Helm verwaltet Releases imperativ (`helm install`, `helm upgrade`). Für GitOps-Workflows braucht man zusätzliche Tools (ArgoCD, Flux), die Helm-Charts deklarativ aus Git deployen.
- **Drift:** Wenn jemand nach dem `helm install` manuell Ressourcen ändert (`oc edit`), weicht der Cluster-Zustand vom Chart ab. Helm erkennt das erst beim nächsten `helm upgrade`.

## Erste Schritte

### Voraussetzung: Cluster-Zugang (kubeconfig)

Helm bringt **keinen eigenen Cluster-Zugang** mit. Es nutzt die kubeconfig-Datei — dasselbe Konzept, das ursprünglich von `kubectl` stammt und von allen Kubernetes-Tools übernommen wurde (`oc`, `helm`, Lens, k9s usw.).

**In der Praxis heißt das:** Wer sich bereits mit `oc login` am Cluster angemeldet hat, kann Helm sofort nutzen — ohne weitere Konfiguration. `oc login` schreibt die Zugangsdaten automatisch nach `~/.kube/config`, und Helm liest sie von dort.

```bash
# Nach erfolgreichem Login...
oc login https://api.cluster.example.com:6443

# ...funktioniert Helm sofort:
helm list
```

#### Die Umgebungsvariable KUBECONFIG

`KUBECONFIG` ist eine **kubectl-Konvention**, die Helm lediglich übernommen hat. Sie wird nur gebraucht, wenn man eine andere Datei als den Standardpfad `~/.kube/config` verwenden will:

```bash
# Nur nötig, wenn die kubeconfig NICHT unter ~/.kube/config liegt:
export KUBECONFIG=/pfad/zu/meiner/kubeconfig.yaml
```

> **Mehrere Cluster?** Man kann mehrere Kontexte in einer kubeconfig-Datei pflegen und mit `kubectl config use-context <name>` zwischen ihnen wechseln. Alternativ: mehrere Dateien mit `KUBECONFIG=/pfad/a:/pfad/b` (Doppelpunkt-getrennt) zusammenführen.

#### Aufbau der kubeconfig-Datei

Die Datei enthält drei Bereiche:

```yaml
apiVersion: v1
kind: Config
current-context: my-cluster            # Aktiver Kontext

clusters:                              # Cluster-Verbindungen
  - name: my-cluster
    cluster:
      server: https://api.cluster.example.com:6443
      certificate-authority-data: LS0t...   # CA-Zertifikat (Base64)

users:                                 # Zugangsdaten
  - name: developer
    user:
      token: sha256~abc123...          # Bearer Token (z.B. von oc login)

contexts:                              # Kombination aus Cluster + User + Namespace
  - name: my-cluster
    context:
      cluster: my-cluster
      user: developer
      namespace: my-project
```

- **clusters** — Adressen und Zertifikate der Kubernetes-/OpenShift-Cluster
- **users** — Authentifizierung (Token, Client-Zertifikat oder OIDC)
- **contexts** — Verknüpfung von Cluster, User und optionalem Default-Namespace

### 1. Helm installieren

```bash
# macOS
brew install helm

# Linux (Snap)
sudo snap install helm --classic

# Oder: Binary von https://github.com/helm/helm/releases
```

### 2. Ein Chart erstellen

```bash
helm create my-app
```

Das erzeugt eine vollständige Chart-Struktur mit Beispiel-Templates. Für den Einstieg kann man die Templates durch eigene YAML-Manifeste ersetzen und schrittweise parametrisieren.

### 3. Chart testen (ohne zu installieren)

```bash
# Templates rendern und prüfen
helm template my-release ./my-app

# Mit eigenen Values
helm template my-release ./my-app -f prod-values.yaml

# Syntaxprüfung
helm lint ./my-app
```

### 4. Chart installieren

```bash
helm install my-release ./my-app
```

### 5. Release verwalten

```bash
# Installierte Releases auflisten
helm list

# Werte ändern und upgraden
helm upgrade my-release ./my-app --set replicas=3

# Rollback auf vorherige Revision
helm rollback my-release 1

# Release entfernen
helm uninstall my-release
```

## Weiterführende Links

- [Helm Cheat Sheet](helm-cheatsheet.html) — Die wichtigsten Helm-Befehle (klassisch und OCI)
- [Aufbau eines Helm Charts](helm-chart-struktur.html) — Alle Dateien und Verzeichnisse im Detail
- [Helm-Dokumentation](https://helm.sh/docs/)
- [Artifact Hub — öffentliche Charts](https://artifacthub.io/)
- [Helm Best Practices](https://helm.sh/docs/chart_best_practices/)
- [Go Template-Syntax](https://helm.sh/docs/chart_template_guide/)
