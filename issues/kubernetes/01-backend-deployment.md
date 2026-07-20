---
layout: default
title: "Übung 1: Backend-Deployment"
---

# Backend in Kubernetes deployen

Als Entwickler möchte ich das Recipes-Backend als Container in Kubernetes deployen, damit es über eine öffentliche URL erreichbar ist.

## 🎯 Lernziele

* Du kannst ein Deployment mit einem YAML-Manifest erstellen und auf den Cluster anwenden.
* Du kannst den Status eines Pods prüfen.
* Du kannst einen Service und einen Ingress erstellen, um eine Anwendung im Cluster und von außen erreichbar zu machen.

## ✅ Definition of Done

* [ ] Du hast ein Deployment für das Backend erstellt und angewendet.
* [ ] Der Backend-Pod läuft (Status `Running`).
* [ ] Du hast einen Service und einen Ingress erstellt.
* [ ] Du kannst die REST-API über den Ingress im Browser oder mit `curl` aufrufen.

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
kubectl apply -f backend-deployment.yaml
```

### 2. Pod-Status prüfen

Prüfe, ob der Pod läuft:

```bash
kubectl get pods
```

Nach kurzer Zeit sollte der Pod im Status `Running` sein.

> **Troubleshooting:** Bleibt der Pod bei `ErrImagePull` oder `ImagePullBackOff` stehen, liegt das am Rate Limiting von Docker Hub für anonyme Zugriffe. Erstelle in diesem Fall ein **Pull Secret** und ergänze es im Deployment:
>
> ```bash
> kubectl create secret docker-registry dockerhub-pull-secret \
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
> Danach erneut `kubectl apply -f backend-deployment.yaml` ausführen.

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
kubectl apply -f backend-service.yaml
```

### 4. Ingress erstellen

Erstelle eine Datei `backend-ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: recipes-backend
spec:
  rules:
    - host: recipes-backend.127.0.0.1.nip.io
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: recipes-backend
                port:
                  number: 8080
```

> **Hinweis:** Wir verwenden hier [nip.io](https://nip.io) — einen DNS-Dienst, der Hostnamen automatisch auf die eingebettete IP-Adresse auflöst. `recipes-backend.127.0.0.1.nip.io` wird also zu `127.0.0.1` aufgelöst. Dein Cluster benötigt einen [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) — bei Minikube ist dieser bereits über das Addon `ingress` aktiviert (siehe [Minikube-Setup](../../docs/minikube-setup.html)).

```bash
kubectl apply -f backend-ingress.yaml
kubectl get ingress recipes-backend
```

### 5. Ingress testen

Damit der Ingress lokal erreichbar ist, muss `minikube tunnel` in einem **separaten Terminal** laufen:

```bash
minikube tunnel
```

Teste dann die API:

```bash
curl -i http://recipes-backend.127.0.0.1.nip.io/recipes
```

Die URL `http://recipes-backend.127.0.0.1.nip.io/recipes` kannst Du auch direkt im Browser öffnen.

## 📚 Selbstlernmaterial

* [Kubernetes Deployment-Dokumentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* [Kubernetes Service-Dokumentation](https://kubernetes.io/docs/concepts/services-networking/service/)
* [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* [YAML-Syntax-Referenz](https://kubernetes.io/docs/reference/kubernetes-api/) (z.B. zu [`Deployment`](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/))
* [Recipes-Backend Docker Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-backend) — Dokumentation der Umgebungsvariablen und Ports

## 🤔 Reflexionsfragen

* Wie ist der Zusammenhang zwischen einem Deployment und einem Pod?
* Wozu verwenden wir einen Service?
* Welche Rolle spielt der Ingress in Kubernetes?
