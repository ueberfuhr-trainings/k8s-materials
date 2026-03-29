---
layout: default
title: "Übung 6: Liveness- und Readiness-Probes"
---

# Liveness- und Readiness-Probes konfigurieren

Als Entwickler möchte ich Liveness- und Readiness-Probes für meine Deployments konfigurieren, damit Kubernetes fehlerhafte Pods automatisch neu startet und nur bereite Pods Traffic erhalten.

## 🎯 Lernziele

* Du verstehst den Unterschied zwischen Liveness- und Readiness-Probes.
* Du kannst Probes in einem Deployment-YAML konfigurieren.
* Du weißt, welche Endpunkte oder Mechanismen sich als Probes für verschiedene Anwendungen eignen.
* Du verstehst die Auswirkung von Probes auf das Verhalten von Services und Rollouts.

## ✅ Definition of Done

* [ ] Du hast Liveness- und Readiness-Probes für das Backend konfiguriert.
* [ ] Du hast mindestens eine Probe für das Frontend konfiguriert.
* [ ] Du hast eine Probe für die PostgreSQL-Datenbank konfiguriert.
* [ ] Du kannst beobachten, wie Kubernetes auf fehlgeschlagene Probes reagiert.

## 🪜 Arbeitsschritte

### 1. Backend-Probes konfigurieren

Das Backend stellt dedizierte Health-Check-Endpunkte bereit. Recherchiere in der [Docker Hub-Dokumentation des Backends](https://hub.docker.com/r/ralfueberfuhr/recipes-backend), welche Endpunkte für Liveness und Readiness zur Verfügung stehen und auf welchem Port diese erreichbar sind.

Recherchiere anschließend in der [Kubernetes-Dokumentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/), wie `livenessProbe` und `readinessProbe` im Container-Spec eines Deployments definiert werden.

Ergänze die Probes im Backend-Deployment und wende das Manifest an:

```bash
oc apply -f backend-deployment.yaml
```

Prüfe, ob die Probes erkannt werden:

```bash
oc describe pod <backend-pod-name>
```

### 2. Frontend-Probes konfigurieren

Das Frontend wird von NGINX ausgeliefert. Überlege, welcher Mechanismus sich hier als Liveness- und/oder Readiness-Probe eignet. Ein einfacher HTTP-Check auf den Root-Pfad (`/`) ist ein guter Ausgangspunkt.

Ergänze die Probes im Frontend-Deployment.

### 3. PostgreSQL-Probes konfigurieren

Für die PostgreSQL-Datenbank kannst Du eine `exec`-basierte Probe verwenden, die prüft, ob die Datenbank Verbindungen annimmt. Recherchiere, wie eine `exec`-Probe konfiguriert wird und welches Kommando sich für PostgreSQL eignet (Tipp: `pg_isready`).

Ergänze die Probes im PostgreSQL-Deployment.

### 4. Verhalten beobachten

Beobachte, wie sich die Probes auf das Verhalten der Pods auswirken:

```bash
oc get pods -w
```

Teste, was passiert, wenn eine Probe fehlschlägt — z.B. indem Du die Datenbank stoppst und beobachtest, wie die Readiness-Probe des Backends reagiert.

## 📚 Selbstlernmaterial

* [Kubernetes: Liveness, Readiness und Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
* [Kubernetes: Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
* [Recipes-Backend Docker Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-backend) — Dokumentation der Health-Check-Endpunkte
* [PostgreSQL `pg_isready`](https://www.postgresql.org/docs/current/app-pg-isready.html)

## 🤔 Reflexionsfragen

* Was ist der Unterschied zwischen einer Liveness-Probe und einer Readiness-Probe? Was passiert jeweils, wenn sie fehlschlägt?
* Warum hat das Backend separate Endpunkte für Liveness und Readiness? Wann könnte die Readiness-Probe fehlschlagen, während die Liveness-Probe noch erfolgreich ist?
* Welche Rolle spielen `initialDelaySeconds`, `periodSeconds` und `failureThreshold`? Was passiert, wenn diese Werte zu aggressiv gewählt werden?
* Warum ist es sinnvoll, auch für das Frontend und die Datenbank Probes zu definieren, obwohl diese keine dedizierten Health-Endpunkte haben?
* Was ist eine Startup-Probe und wann sollte man sie zusätzlich einsetzen?
