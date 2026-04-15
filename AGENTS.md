# Informationen für KI-Agenten

## Projektkontext

Dieses Verzeichnis enthält Übungsmaterialien für einen Schulungskurs zu **Kubernetes und OpenShift**.
Die Teilnehmer lernen anhand einer Rezeptverwaltungs-Anwendung (Backend + Frontend), wie Container
in Kubernetes/OpenShift deployed, konfiguriert und betrieben werden.

## Container-Images

### Backend: `ralfueberfuhr/recipes-backend`

- **Technologie:** Quarkus (Java 21)
- **Docker Hub:** https://hub.docker.com/r/ralfueberfuhr/recipes-backend
- **Quellcode:** [k8s-sample-backend](https://github.com/ueberfuhr-trainings/k8s-sample-backend)
- **Dokumentation:** [Docker README](https://github.com/ueberfuhr-trainings/k8s-sample-backend/blob/main/src/main/docker/README.md)
- **Tags:**
  - `latest` — benötigt PostgreSQL (Umgebungsvariablen: `DB_URL`, `DB_USER`, `DB_PASSWORD`)
  - `latest-dev` — verwendet eingebaute H2 InMemory-Datenbank, keine DB-Konfiguration nötig
- **Ports:** 8080 (REST API), 9000 (Management/Health Checks)
- **Health Checks:** `/q/health/live` (Liveness), `/q/health/ready` (Readiness) auf Port 9000
- **CORS:** konfigurierbar über `CORS_ORIGINS`, `CORS_METHODS`, `CORS_HEADERS`, `CORS_EXPOSED_HEADERS`
- **REST API:** `/recipes` (GET, POST), `/recipes/{id}` (GET, PUT, DELETE)

### Frontend: `ralfueberfuhr/recipes-frontend`

- **Technologie:** Angular, ausgeliefert über NGINX
- **Docker Hub:** https://hub.docker.com/r/ralfueberfuhr/recipes-frontend
- **Quellcode:** [k8s-sample-frontend](https://github.com/ueberfuhr-trainings/k8s-sample-frontend)
- **Dokumentation:** [Docker README](https://github.com/ueberfuhr-trainings/k8s-sample-frontend/blob/main/recipes-app/docker/README.md)
- **Tags:** `latest`
- **Port:** 8080 (NGINX)
- **Konfiguration:** `API_BASE_URL` — URL zum Backend (Default: `http://localhost:3000`)

## Übungsstruktur

Die Übungen im `issues/`-Ordner folgen einem Template (`issue_template.md`) mit:
- Einleitungstext als User Story
- Lernziele
- Definition of Done (Checkliste)
- Arbeitsschritte mit YAML-Beispielen
- Selbstlernmaterial (Links)
- Reflexionsfragen

### Plattform-Varianten

Die Übungen existieren in zwei Varianten, jeweils in einem eigenen Unterverzeichnis:

- **`issues/openshift/`** — OpenShift-Variante: verwendet `oc` CLI und Routes (`route.openshift.io/v1`)
- **`issues/kubernetes/`** — Kubernetes-Variante: verwendet `kubectl` CLI und Ingress (`networking.k8s.io/v1`)

Die Übungsinhalte sind ansonsten identisch. Bei Änderungen an einer Übung müssen **beide Varianten** angepasst werden. Die wesentlichen Unterschiede sind:

| Aspekt | OpenShift | Kubernetes |
|--------|-----------|------------|
| CLI | `oc` | `kubectl` |
| Externer Zugriff | Route | Ingress |
| Texte/Titel | "in OpenShift deployen" | "in Kubernetes deployen" |

## Reihenfolge der Übungen

1. **Backend-Deployment** — Deployment, Service, Route/Ingress mit YAML
2. **Frontend-Deployment** — Deployment mit Umgebungsvariable `API_BASE_URL`
3. **ConfigMaps** — Backend-URL in ConfigMap auslagern
4. **PostgreSQL-Setup** — PostgreSQL-Deployment, Secrets, ConfigMaps für Init-SQL
5. **Backend auf PostgreSQL umschalten** — `latest`-Tag mit DB-Konfiguration aus Secrets
6. **Liveness- und Readiness-Probes** — Health Checks für Backend, Frontend und Datenbank
7. **Helm Charts** — Anwendung als Helm Charts paketieren

## Hinweise für die Generierung

- Übungen sind in deutscher Sprache verfasst
- Die Übungen bauen aufeinander auf
- PostgreSQL wird ohne PVC deployed (Persistent Volumes sind in der Schulungsumgebung nicht erlaubt)
- Bei neuen oder geänderten Übungen immer beide Plattform-Varianten (OpenShift und Kubernetes) pflegen
