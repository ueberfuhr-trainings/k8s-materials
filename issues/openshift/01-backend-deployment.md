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
* [ ] Du hast den Pod-Status geprüft und ggf. einen `ErrImagePull`-Fehler mit einem Pull Secret behoben.
* [ ] Du hast die Fehlermeldung beim `latest`-Tag analysiert und verstanden.
* [ ] Du hast das Image-Tag auf `latest-dev` geändert und der Pod läuft erfolgreich.
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
          image: ralfueberfuhr/recipes-backend:latest
          ports:
            - containerPort: 8080
```

Wende das Manifest an:

```bash
oc apply -f backend-deployment.yaml
```

### 2. Pod-Status prüfen

Prüfe den Status des Pods:

```bash
oc get pods
oc describe pod <pod-name>
```

Falls der Pod im Status `ErrImagePull` oder `ImagePullBackOff` steht, liegt das am Rate Limiting von Docker Hub für anonyme Zugriffe. In diesem Fall musst Du ein **Pull Secret** erstellen (siehe Schritt 3). Falls der Pod das Image erfolgreich ziehen konnte, kannst Du Schritt 3 überspringen und direkt mit Schritt 4 fortfahren.

### 3. *(Falls nötig)* Pull Secret erstellen und Deployment ergänzen

Erstelle ein Pull Secret mit Deinen Docker Hub-Zugangsdaten:

```bash
oc create secret docker-registry dockerhub-pull-secret \
  --docker-server=docker.io \
  --docker-username=<dein-dockerhub-username> \
  --docker-password=<dein-dockerhub-token>
```

Ergänze im `backend-deployment.yaml` den Abschnitt `imagePullSecrets` im `spec`-Block des Pod-Templates:

```yaml
    spec:
      imagePullSecrets:
        - name: dockerhub-pull-secret
      containers:
        - name: recipes-backend
```

Wende das geänderte Manifest erneut an:

```bash
oc apply -f backend-deployment.yaml
```

### 4. Fehler analysieren — Datenbank

Der Pod wird nun das Image ziehen können, aber in einen neuen Fehlerzustand laufen, weil das `latest`-Tag eine PostgreSQL-Datenbank erwartet:

```bash
oc get pods
oc describe pod <pod-name>
oc logs <pod-name>
```

Du solltest eine Fehlermeldung sehen, die auf eine fehlende Datenbankkonfiguration hinweist.

### 5. Auf `latest-dev` wechseln

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

### 6. Service erstellen

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

### 7. Route erstellen

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
* [YAML-Syntax-Referenz](https://kubernetes.io/docs/reference/kubernetes-api/) (z.B. zu [`Deployment`](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/))
* [Recipes-Backend Docker Hub](https://hub.docker.com/r/ralfueberfuhr/recipes-backend) — Dokumentation der Umgebungsvariablen und Ports

## 🤔 Reflexionsfragen

* Warum ist der Pod mit dem `latest`-Tag fehlgeschlagen?
* Wie ist der Zusammenhang zwischen einem Deployment und einem Pod?
* Warum braucht man einen Service, obwohl der Pod schon läuft? Welche Antwort erhalten wir, wenn der Pod gestoppt ist?
* Welche Rolle spielt die Route in OpenShift?
* Was passiert mit den Daten in der H2-Datenbank, wenn der Pod neu gestartet wird?
* Was ist die `imagePullPolicy` und welcher Wert gilt standardmäßig? Welche anderen Werte gibt es, und wann ist welcher sinnvoll?
