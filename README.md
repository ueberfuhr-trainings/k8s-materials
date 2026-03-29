# Kubernetes/OpenShift - Kursmaterialien

Dieses Verzeichnis enthält die Übungsmaterialien für den Kurs **Kubernetes/OpenShift**.

## Struktur

- `index.md` — Startseite der GitHub Pages mit Übersicht und Links zu allen Übungen
- `issues/` — Übungen als GitHub-Issue-Vorlagen (Markdown), bereit zum Kopieren
- `_config.yml` — Jekyll-Konfiguration für GitHub Pages
- `_layouts/` — Layout-Templates für GitHub Pages

## Verwendete Container-Images

Die Übungen basieren auf zwei Container-Images, die über Docker Hub bereitgestellt werden:

| Image | Beschreibung | Docker Hub |
|-------|-------------|------------|
| `ralfueberfuhr/recipes-backend` | Quarkus-basiertes REST-Backend zur Rezeptverwaltung | [Link](https://hub.docker.com/r/ralfueberfuhr/recipes-backend) |
| `ralfueberfuhr/recipes-frontend` | Angular-basiertes Web-Frontend zur Rezeptverwaltung | [Link](https://hub.docker.com/r/ralfueberfuhr/recipes-frontend) |

### Backend-Tags

- `latest` — Produktions-Image, erwartet eine PostgreSQL-Datenbank
- `latest-dev` — Entwicklungs-Image mit eingebauter H2 InMemory-Datenbank (kein externer DB-Service nötig)

### Frontend-Tags

- `latest` — Produktions-Image, konfigurierbar über `API_BASE_URL`

## Quellen

Die Container-Images werden aus folgenden GitHub-Repositories gebaut:

- [k8s-sample-backend](https://github.com/ueberfuhr-trainings/k8s-sample-backend) — Quarkus-Anwendung (Java 21)
- [k8s-sample-frontend](https://github.com/ueberfuhr-trainings/k8s-sample-frontend) — Angular-Anwendung

Die Docker-Dokumentation (inkl. Umgebungsvariablen, Health Checks, SQL-Schema) findet sich jeweils im `docker/`- bzw. `src/main/docker/`-Ordner der Repositories.
