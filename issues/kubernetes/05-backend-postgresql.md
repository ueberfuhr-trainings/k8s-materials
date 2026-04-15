---
layout: default
title: "Übung 5: Backend auf PostgreSQL umschalten"
---

# Backend auf PostgreSQL umschalten

Als Entwickler möchte ich das Backend von der eingebauten H2-Datenbank auf die PostgreSQL-Datenbank umschalten, damit die Daten persistent in einer echten Datenbank gespeichert werden.

## 🎯 Lernziele

* Du kannst ein bestehendes Deployment auf ein anderes Image-Tag umstellen.
* Du kannst Umgebungsvariablen aus verschiedenen Quellen (Secrets, ConfigMaps, statische Werte) kombinieren.
* Du verstehst, wie Services als interne DNS-Namen für die Kommunikation zwischen Pods genutzt werden.

## ✅ Definition of Done

* [ ] Du hast das Backend-Deployment auf das `latest`-Tag umgestellt.
* [ ] Du hast die Datenbank-Umgebungsvariablen konfiguriert (`DB_URL`, `DB_USER`, `DB_PASSWORD`).
* [ ] `DB_USER` und `DB_PASSWORD` werden aus dem PostgreSQL-Secret bezogen.
* [ ] Das Backend läuft erfolgreich und nutzt die PostgreSQL-Datenbank.
* [ ] Du kannst Rezepte erstellen und sie bleiben auch nach einem Neustart des Backend-Pods erhalten (solange der DB-Pod läuft).

## 🪜 Arbeitsschritte

### 1. Image-Tag ändern

Ändere im Backend-Deployment das Image-Tag von `latest-dev` zurück auf `latest`:

```yaml
image: ralfueberfuhr/recipes-backend:latest
```

### 2. Umgebungsvariablen konfigurieren

Füge die Datenbank-Konfiguration zum Backend-Deployment hinzu. Verwende den Service-Namen der PostgreSQL als Hostname in der JDBC-URL:

```yaml
env:
  - name: DB_URL
    value: "jdbc:postgresql://postgres:5432/recipes"
  - name: DB_USER
    valueFrom:
      secretKeyRef:
        name: postgres-secret
        key: POSTGRES_USER
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: postgres-secret
        key: POSTGRES_PASSWORD
```

> **Hinweis:** Der Hostname `postgres` in der JDBC-URL entspricht dem Service-Namen aus Übung 4. Kubernetes stellt sicher, dass der Service-Name innerhalb des Clusters als DNS-Name aufgelöst wird.

### 3. Deployment anwenden und prüfen

```bash
kubectl apply -f backend-deployment.yaml
kubectl get pods
kubectl logs <backend-pod-name>
```

In den Logs sollte die erfolgreiche Verbindung zur PostgreSQL sichtbar sein. Prüfe die Readiness-Probe:

```bash
kubectl describe pod <backend-pod-name>
```

### 4. Funktionstest

Erstelle ein Rezept über die API und prüfe, ob es nach einem Pod-Neustart noch vorhanden ist:

```bash
curl -s -X POST https://<backend-ingress-url>/recipes \
  -H "Content-Type: application/json" \
  -d '{"name":"Test","servings":2,"duration":10,"ingredients":[{"name":"Tomate","quantity":1,"unit":"pieces"}],"preparation":"Kochen."}' | jq

kubectl rollout restart deployment/recipes-backend
# Warten, bis der neue Pod läuft...
curl -s https://<backend-ingress-url>/recipes | jq
```

## 📚 Selbstlernmaterial

* [Kubernetes DNS für Services](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
* [Umgebungsvariablen aus Secrets beziehen](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data)
* [Recipes-Backend Docker Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-backend) — Dokumentation der Datenbank-Konfiguration

## 🤔 Reflexionsfragen

* Brauchen wir für die Datenbank einen Ingress?
* Ist es sinnvoll, die `DB_URL` auch im  `postgres-secret` bereitzustellen?
