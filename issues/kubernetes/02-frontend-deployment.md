---
layout: default
title: "Übung 2: Frontend-Deployment"
---

# Frontend in Kubernetes deployen

Als Entwickler möchte ich das Recipes-Frontend in Kubernetes deployen und mit dem Backend verbinden, damit Anwender Rezepte über eine Weboberfläche verwalten können.

## 🎯 Lernziele

* Du kannst ein Frontend-Deployment erstellen und über einen Ingress erreichbar machen.
* Du verstehst, wie Fehlkonfigurationen in den Browser Developer Tools sichtbar werden.
* Du kannst Umgebungsvariablen in einem Deployment-YAML konfigurieren.
* Du verstehst, wie Frontend und Backend über eine Umgebungsvariable verbunden werden.

## ✅ Definition of Done

* [ ] Du hast das Frontend deployed und einen Ingress erstellt.
* [ ] Du hast in den Browser Developer Tools die fehlerhafte Backend-URL identifiziert.
* [ ] Du hast die Umgebungsvariable `API_BASE_URL` im Deployment gesetzt.
* [ ] Das Frontend kann Rezepte vom Backend laden und anzeigen.

## 🪜 Arbeitsschritte

### 1. Frontend deployen

Erstelle ein Deployment, einen Service und einen Ingress für das Frontend — analog zum Backend aus Übung 1. Verwende das Image `ralfueberfuhr/recipes-frontend:latest` mit dem Container-Port `8080`. Verwende als Ingress-Host `recipes-frontend.127.0.0.1.nip.io`.

Wende die Manifeste an und prüfe, ob der Pod läuft.

### 2. Fehler im Browser analysieren

Stelle sicher, dass `minikube tunnel` in einem separaten Terminal läuft (siehe Übung 1). Öffne dann `http://recipes-frontend.127.0.0.1.nip.io` im Browser. Öffne die **Developer Tools** (F12) und wechsle zum **Network**-Tab. Du wirst sehen, dass das Frontend versucht, das Backend unter einer falschen URL zu erreichen (Standard: `http://localhost:3000`).

### 3. Umgebungsvariable setzen

Das Frontend benötigt die Umgebungsvariable `API_BASE_URL`, um die korrekte Backend-URL zu kennen.

Recherchiere in der [Kubernetes-Dokumentation](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/), wie Umgebungsvariablen in einem Container-Spec definiert werden, und setze `API_BASE_URL` auf die Ingress-URL des Backends aus Übung 1 (`http://recipes-backend.127.0.0.1.nip.io`).

Wende das geänderte Manifest an und prüfe im Browser, ob das Frontend jetzt korrekt mit dem Backend kommuniziert.

## 📚 Selbstlernmaterial

* [Kubernetes: Umgebungsvariablen definieren](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/)
* [Recipes-Frontend Docker Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-frontend) — Dokumentation der Umgebungsvariablen
* [Browser Developer Tools (MDN)](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Tools_and_setup/What_are_browser_developer_tools)

## 🤔 Reflexionsfragen

* Warum verwendet das Frontend eine Umgebungsvariable statt einer fest kodierten URL?
* Was passiert, wenn der Backend-Ingress sich ändert — wo musst Du die URL aktualisieren?
* Könnte das Frontend auch den internen Service-Namen (`http://recipes-backend:8080`) statt der Ingress-URL verwenden? Warum oder warum nicht?
