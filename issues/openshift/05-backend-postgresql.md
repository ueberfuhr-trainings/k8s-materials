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

* [ ] Du hast für die PostgreSQL aus Übung 4 einen **Service** (Port 5432) erstellt.
* [ ] Du hast das Backend-Deployment auf das `latest`-Tag umgestellt.
* [ ] Du hast die Datenbank-Umgebungsvariablen konfiguriert (`DB_URL`, `DB_USER`, `DB_PASSWORD`).
* [ ] `DB_USER` und `DB_PASSWORD` werden aus dem PostgreSQL-Secret bezogen.
* [ ] Das Backend läuft erfolgreich und nutzt die PostgreSQL-Datenbank.
* [ ] Du kannst Rezepte erstellen und sie bleiben auch nach einem Neustart des Backend-Pods erhalten (solange der DB-Pod läuft).

## 🪜 Arbeitsschritte

### 1. PostgreSQL über einen Service erreichbar machen

Damit das Backend die Datenbank über einen **stabilen Namen** statt über eine wechselnde Pod-IP erreicht, braucht die PostgreSQL aus Übung 4 einen **Service**. Erstelle ihn mit folgenden Anforderungen:

* Er wählt die PostgreSQL-Pods über ihr Label aus (`app: postgres`).
* Er ist clusterintern unter Port **5432** erreichbar.
* Sein Name wird gleich als DNS-Hostname in der JDBC-URL verwendet — wähle ihn passend (z.B. `postgres`).

### 2. Image-Tag ändern

Ändere im Backend-Deployment das Image-Tag von `latest-dev` zurück auf `latest`:

```yaml
image: ralfueberfuhr/recipes-backend:latest
```

### 3. Umgebungsvariablen konfigurieren

Füge dem Backend-Deployment folgende Umgebungsvariablen hinzu:

* **`DB_URL`** — JDBC-URL zur Datenbank: `jdbc:postgresql://postgres:5432/recipes` (Hostname = Service-Name aus Schritt 1)
* **`DB_USER`** — aus dem `postgres-secret`, Schlüssel `POSTGRES_USER`
* **`DB_PASSWORD`** — aus dem `postgres-secret`, Schlüssel `POSTGRES_PASSWORD`

Überlege selbst, wie du einen statischen Wert (`DB_URL`) und Werte aus einem Secret (`DB_USER`, `DB_PASSWORD`) im Deployment angibst.

> **Hinweis:** Der Hostname `postgres` in der JDBC-URL entspricht dem Service-Namen aus Schritt 1. Kubernetes löst den Service-Namen clusterintern als DNS-Namen auf.

### 4. Deployment anwenden und prüfen

```bash
oc apply -f backend-deployment.yaml
oc get pods
oc logs <backend-pod-name>
```

In den Logs sollte die erfolgreiche Verbindung zur PostgreSQL sichtbar sein. Prüfe die Readiness-Probe:

```bash
oc describe pod <backend-pod-name>
```

### 5. Funktionstest

Erstelle ein Rezept über die API und prüfe, ob es nach einem Pod-Neustart noch vorhanden ist:

```bash
curl -i -X POST https://<backend-route>/recipes \
  -H "Content-Type: application/json" \
  -d '{"name":"Test","servings":2,"duration":10,"ingredients":[{"name":"Tomate","quantity":1,"unit":"pieces"}],"preparation":"Kochen."}'

oc rollout restart deployment/recipes-backend
# Warten, bis der neue Pod läuft...
curl -i https://<backend-route>/recipes
```

## 📚 Selbstlernmaterial

* [Kubernetes DNS für Services](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
* [Umgebungsvariablen aus Secrets beziehen](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-using-secret-data)
* [Recipes-Backend Docker Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-backend) — Dokumentation der Datenbank-Konfiguration

## 🤔 Reflexionsfragen

* Brauchen wir für die Datenbank eine Route?
* Ist es sinnvoll, die `DB_URL` auch im  `postgres-secret` bereitzustellen?
