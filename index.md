---
layout: default
title: Kubernetes/OpenShift - Schulungsunterlagen
---

# Kubernetes/OpenShift

Willkommen zu den Übungen für den Kurs **Kubernetes/OpenShift**!

In diesen Übungen deployt und konfiguriert Ihr eine Rezeptverwaltungs-Anwendung bestehend aus einem Backend und einem Frontend in Kubernetes bzw. OpenShift.

## Zum Einstieg

* [Docker Grundlagen-Check](reflections/docker-grundlagen.html) — Quiz zur Auffrischung der Container-Basics

## Dokumentation

* [Was ist Kubernetes?](docs/kubernetes.html) — Kernkonzepte und Abgrenzung zu Docker
* [Kubernetes-Ressourcen](docs/kubernetes-resources.html) — Pods, Deployments, Services, ConfigMaps, Probes und mehr
* [Was ist OpenShift?](docs/openshift.html) — Unterschiede zu Kubernetes, Sicherheit, Installationsoptionen
* [kubectl/oc Cheat Sheet](docs/cli-cheatsheet.html) — Die wichtigsten CLI-Befehle für Kubernetes und OpenShift
* [Health Checks (Probes)](docs/probes.html) — Startup, Liveness und Readiness Probes mit Beispielen
* [PostgreSQL auf Kubernetes – Best Practices](docs/postgres-as-statefulset.html) — StatefulSet, Operatoren und produktionsreife Setups
* [Helm Charts](docs/helm.html) — Paketmanagement für Kubernetes: Konzepte, OCI-Support, Architektur
* [Aufbau eines Helm Charts](docs/helm-chart-struktur.html) — Alle Dateien und Verzeichnisse im Detail
* [Helm Cheat Sheet](docs/helm-cheatsheet.html) — Die wichtigsten Helm-Befehle (klassisch und OCI)

## Container-Images

| Image                            | Docker Hub                                                     | Beschreibung                 |
|----------------------------------|----------------------------------------------------------------|------------------------------|
| `ralfueberfuhr/recipes-backend`  | [Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-backend)  | REST-Backend (Quarkus)       |
| `ralfueberfuhr/recipes-frontend` | [Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-frontend) | Web-Frontend (Angular/NGINX) |

## Übungen (OpenShift)

1. [Backend-Deployment](issues/openshift/01-backend-deployment.html) — Deployment, Service und Route für das Backend erstellen
2. [Frontend-Deployment](issues/openshift/02-frontend-deployment.html) — Frontend deployen und mit dem Backend verbinden
3. [Kreuzworträtsel](reflections/crossword-tag1.html) / [Quiz](reflections/quiz-tag1.html) — Wiederholung
3. [ConfigMaps verwenden](issues/openshift/03-configmaps.html) — Konfiguration in ConfigMaps auslagern
4. [PostgreSQL-Datenbank](issues/openshift/04-postgresql.html) — PostgreSQL deployen und mit Secrets/ConfigMaps konfigurieren
5. [Backend auf PostgreSQL umschalten](issues/openshift/05-backend-postgresql.html) — Backend mit externer Datenbank betreiben
6. [Persistent Volumes](issues/openshift/06-persistent-volumes.html) — Persistenten Speicher für die PostgreSQL-Datenbank einrichten
7. [Liveness- und Readiness-Probes](issues/openshift/07-probes.html) — Health Checks für Backend, Frontend und Datenbank
8. [Helm Charts](issues/openshift/08-helm-charts.html) — Anwendung als Helm Charts paketieren und mit einem Befehl deployen

## Übungen (Kubernetes)

> **Vorbereitung:** [Minikube-Setup für die Schulung](docs/minikube-setup.html) — Minikube installieren, Cluster starten und Namespace `schulung` einrichten

1. [Backend-Deployment](issues/kubernetes/01-backend-deployment.html) — Deployment, Service und Ingress für das Backend erstellen
2. [Frontend-Deployment](issues/kubernetes/02-frontend-deployment.html) — Frontend deployen und mit dem Backend verbinden
3. [Kreuzworträtsel](reflections/crossword-tag1.html) / [Quiz](reflections/quiz-tag1.html) — Wiederholung
3. [ConfigMaps verwenden](issues/kubernetes/03-configmaps.html) — Konfiguration in ConfigMaps auslagern
4. [PostgreSQL-Datenbank](issues/kubernetes/04-postgresql.html) — PostgreSQL deployen und mit Secrets/ConfigMaps konfigurieren
5. [Backend auf PostgreSQL umschalten](issues/kubernetes/05-backend-postgresql.html) — Backend mit externer Datenbank betreiben
6. [Persistent Volumes](issues/kubernetes/06-persistent-volumes.html) — Persistenten Speicher für die PostgreSQL-Datenbank einrichten
7. [Liveness- und Readiness-Probes](issues/kubernetes/07-probes.html) — Health Checks für Backend, Frontend und Datenbank
8. [Helm Charts](issues/kubernetes/08-helm-charts.html) — Anwendung als Helm Charts paketieren und mit einem Befehl deployen
