---
layout: default
title: Kubernetes/OpenShift - Schulungsunterlagen
---

# Kubernetes/OpenShift

Willkommen zu den Übungen für den Kurs **Kubernetes/OpenShift**!

In diesen Übungen deployt und konfiguriert Ihr eine Rezeptverwaltungs-Anwendung bestehend aus einem Backend und einem Frontend in OpenShift.

## Dokumentation

* [Was ist Kubernetes?](docs/kubernetes.html) — Kernkonzepte und Abgrenzung zu Docker
* [Kubernetes-Ressourcen](docs/kubernetes-resources.html) — Pods, Deployments, Services, ConfigMaps, Probes und mehr
* [Was ist OpenShift?](docs/openshift.html) — Unterschiede zu Kubernetes, Sicherheit, Installationsoptionen
* [kubectl/oc Cheat Sheet](docs/cli-cheatsheet.html) — Die wichtigsten CLI-Befehle für Kubernetes und OpenShift

## Container-Images

| Image                            | Docker Hub                                                     | Beschreibung                 |
|----------------------------------|----------------------------------------------------------------|------------------------------|
| `ralfueberfuhr/recipes-backend`  | [Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-backend)  | REST-Backend (Quarkus)       |
| `ralfueberfuhr/recipes-frontend` | [Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-frontend) | Web-Frontend (Angular/NGINX) |

## Übungen

1. [Backend-Deployment](issues/01-backend-deployment.html) — Deployment, Service und Route für das Backend erstellen
2. [Frontend-Deployment](issues/02-frontend-deployment.html) — Frontend deployen und mit dem Backend verbinden
3. [ConfigMaps verwenden](issues/03-configmaps.html) — Konfiguration in ConfigMaps auslagern
4. [PostgreSQL-Datenbank](issues/04-postgresql.html) — PostgreSQL deployen und mit Secrets/ConfigMaps konfigurieren
5. [Backend auf PostgreSQL umschalten](issues/05-backend-postgresql.html) — Backend mit externer Datenbank betreiben
6. [Liveness- und Readiness-Probes](issues/06-probes.html) — Health Checks für Backend, Frontend und Datenbank
