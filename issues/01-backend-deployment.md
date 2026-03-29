---
layout: default
title: "Übung 1: Backend-Deployment"
---

# Backend in OpenShift deployen

Als Entwickler möchte ich das Recipes-Backend als Container in OpenShift deployen, damit es über eine öffentliche URL erreichbar ist.

## 🎯 Lernziele

* Du kannst ein Deployment mit einem YAML-Manifest erstellen und auf den Cluster anwenden.
* Du kannst Fehler in Pods mit `oc describe pod` und `oc logs` analysieren.
* Du kannst einen Service und eine Route erstellen, um eine Anwendung im Cluster und von außen erreichbar zu machen.

## ✅ Definition of Done

* [ ] Du hast ein Deployment für das Backend erstellt und angewendet.
* [ ] Du hast die Fehlermeldung beim `latest`-Tag analysiert und verstanden.
* [ ] Du hast das Image-Tag auf `latest-dev` geändert und der Pod läuft erfolgreich.
* [ ] Du hast einen Service und eine Route erstellt.
* [ ] Du kannst die REST-API über die Route im Browser oder mit `curl` aufrufen.

## 🪜 Arbeitsschritte

### 1. Pull Secret erstellen

Da die Container-Images von Docker Hub gezogen werden, benötigst Du ein Pull Secret mit Deinen Docker Hub-Zugangsdaten:

```bash
oc create secret docker-registry dockerhub-pull-secret \
  --docker-server=docker.io \
  --docker-username=<dein-dockerhub-username> \
  --docker-password=<dein-dockerhub-token>
```

### 2. Deployment erstellen

Erstelle eine Datei `backend-deployment.yaml` mit folgendem Inhalt:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: recipes-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: recipes-backend
  template:
    metadata:
      labels:
        app: recipes-backend
    spec:
      imagePullSecrets:
        - name: dockerhub-pull-secret
      containers:
        - name: recipes-backend
          image: ralfueberfuhr/recipes-backend:latest
          ports:
            - containerPort: 8080
```

Wende das Manifest an:

```bash
oc apply -f backend-deployment.yaml
```

### 3. Fehler analysieren

Der Pod wird in einen Fehlerzustand laufen, weil das `latest`-Tag eine PostgreSQL-Datenbank erwartet. Untersuche den Fehler:

```bash
oc get pods
oc describe pod <pod-name>
oc logs <pod-name>
```

Du solltest eine Fehlermeldung sehen, die auf eine fehlende Datenbankkonfiguration hinweist.

### 4. Auf `latest-dev` wechseln

Ändere in deinem YAML das Image-Tag von `latest` auf `latest-dev`. Dieses Tag enthält eine eingebaute H2 InMemory-Datenbank und benötigt keine externe Datenbank.

```yaml
          image: ralfueberfuhr/recipes-backend:latest-dev
```

Wende das geänderte Manifest erneut an und prüfe, ob der Pod jetzt läuft:

```bash
oc apply -f backend-deployment.yaml
oc get pods
oc logs <pod-name>
```

### 5. Service erstellen

Erstelle eine Datei `backend-service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: recipes-backend
spec:
  selector:
    app: recipes-backend
  ports:
    - port: 8080
      targetPort: 8080
```

```bash
oc apply -f backend-service.yaml
```

### 6. Route erstellen

Erstelle eine Datei `backend-route.yaml`:

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: recipes-backend
spec:
  to:
    kind: Service
    name: recipes-backend
  port:
    targetPort: 8080
  tls:
    termination: edge
```

```bash
oc apply -f backend-route.yaml
oc get route recipes-backend
```

Teste die API:

```bash
curl -s https://<route-url>/recipes | jq
```

## 📚 Selbstlernmaterial

* [Kubernetes Deployment-Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [Kubernetes Service-Dokumentation](https://kubernetes.io/docs/concepts/services-networking/service/)
* [OpenShift Routes](https://docs.openshift.com/container-platform/latest/networking/routes/route-configuration.html)
* [YAML-Syntax-Referenz](https://kubernetes.io/docs/reference/kubernetes-api/)
* [Recipes-Backend Docker Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-backend) — Dokumentation der Umgebungsvariablen und Ports

## 🤔 Reflexionsfragen

* Warum ist der Pod mit dem `latest`-Tag fehlgeschlagen? Was bedeutet das für den produktiven Einsatz?
* Was ist der Unterschied zwischen einem Deployment und einem Pod? Warum erstellt man nicht direkt Pods?
* Warum braucht man einen Service, obwohl der Pod schon läuft? Was passiert, wenn der Pod neu gestartet wird?
* Welche Rolle spielt die Route in OpenShift? Wie unterscheidet sich das von einem Kubernetes Ingress?
* Was passiert mit den Daten in der H2-Datenbank, wenn der Pod neu gestartet wird?
