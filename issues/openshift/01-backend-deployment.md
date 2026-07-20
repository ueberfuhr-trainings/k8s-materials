---
layout: default
title: "Übung 1: Backend-Deployment"
---

# Backend in OpenShift deployen

Als Entwickler möchte ich das Recipes-Backend als Container in OpenShift deployen, damit es über eine öffentliche URL erreichbar ist.

## 🎯 Lernziele

* Du kannst ein Deployment mit einem YAML-Manifest erstellen und auf den Cluster anwenden.
* Du kannst den Status eines Pods prüfen.
* Du kannst einen Service und eine Route erstellen, um eine Anwendung im Cluster und von außen erreichbar zu machen.

## ✅ Definition of Done

* [ ] Du hast ein Deployment für das Backend erstellt und angewendet.
* [ ] Der Backend-Pod läuft (Status `Running`).
* [ ] Du hast einen Service und eine Route erstellt.
* [ ] Du kannst die REST-API über die Route im Browser oder mit `curl` aufrufen.

## 🪜 Arbeitsschritte

### 1. Deployment erstellen

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
      containers:
        - name: recipes-backend
          image: ralfueberfuhr/recipes-backend:latest-dev
          ports:
            - containerPort: 8080
```

> **Hinweis:** Wir verwenden das Tag `latest-dev`. Dieses enthält eine eingebaute H2 InMemory-Datenbank und benötigt keine externe Datenbank — ideal für den ersten Schritt.

Wende das Manifest an:

```bash
oc apply -f backend-deployment.yaml
```

### 2. Pod-Status prüfen

Prüfe, ob der Pod läuft:

```bash
oc get pods
```

Nach kurzer Zeit sollte der Pod im Status `Running` sein.

> **Troubleshooting:** Bleibt der Pod bei `ErrImagePull` oder `ImagePullBackOff` stehen, liegt das am Rate Limiting von Docker Hub für anonyme Zugriffe. Erstelle in diesem Fall ein **Pull Secret** und ergänze es im Deployment:
>
> ```bash
> oc create secret docker-registry dockerhub-pull-secret \
>   --docker-server=docker.io \
>   --docker-username=<dein-dockerhub-username> \
>   --docker-password=<dein-dockerhub-token>
> ```
>
> ```yaml
>     spec:
>       imagePullSecrets:
>         - name: dockerhub-pull-secret
>       containers:
>         - name: recipes-backend
> ```
>
> Danach erneut `oc apply -f backend-deployment.yaml` ausführen.

### 3. Service erstellen

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

### 4. Route erstellen

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
curl -i https://<route-url>/recipes
```

## 📚 Selbstlernmaterial

* [Kubernetes Deployment-Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [Kubernetes Service-Dokumentation](https://kubernetes.io/docs/concepts/services-networking/service/)
* [OpenShift Routes](https://docs.openshift.com/container-platform/latest/networking/routes/route-configuration.html)
* [YAML-Syntax-Referenz](https://kubernetes.io/docs/reference/kubernetes-api/) (z.B. zu [`Deployment`](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/))
* [Recipes-Backend Docker Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-backend) — Dokumentation der Umgebungsvariablen und Ports

## 🤔 Reflexionsfragen

* Wie ist der Zusammenhang zwischen einem Deployment und einem Pod?
* Wozu verwenden wir einen Service?
* Welche Rolle spielt die Route in OpenShift?
