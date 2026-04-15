---
layout: default
title: "Übung 3: ConfigMaps verwenden"
---

# Konfiguration mit ConfigMaps auslagern

Als Entwickler möchte ich die Backend-URL für das Frontend in einer ConfigMap verwalten, damit die Konfiguration unabhängig vom Deployment geändert werden kann.

## 🎯 Lernziele

* Du verstehst das Konzept von ConfigMaps und deren Einsatzzweck.
* Du kannst eine ConfigMap mit `kubectl` oder YAML erstellen.
* Du kannst Umgebungsvariablen eines Containers aus einer ConfigMap beziehen.
* Du verstehst die Trennung von Konfiguration und Deployment.

## ✅ Definition of Done

* [ ] Du hast eine ConfigMap mit der Backend-URL erstellt.
* [ ] Du hast das Frontend-Deployment so angepasst, dass `API_BASE_URL` aus der ConfigMap gelesen wird.
* [ ] Das Frontend funktioniert weiterhin korrekt.
* [ ] Du kannst die URL in der ConfigMap ändern, ohne das Deployment-YAML anzufassen.

## 🪜 Arbeitsschritte

### 1. ConfigMap erstellen

Erstelle die ConfigMap direkt per CLI:

```bash
kubectl create configmap frontend-config \
  --from-literal=API_BASE_URL="http://<deine-backend-ingress-url>"
```

Prüfe das Ergebnis:

```bash
kubectl get configmap frontend-config -o yaml
```

### 2. Frontend-Deployment anpassen

Ändere die Umgebungsvariable im Frontend-Deployment, sodass der Wert aus der ConfigMap bezogen wird:

```yaml
env:
  - name: API_BASE_URL
    valueFrom:
      configMapKeyRef:
        name: frontend-config
        key: API_BASE_URL
```

Wende das geänderte Deployment an:

```bash
kubectl apply -f frontend-deployment.yaml
```

### 3. Funktion prüfen

Prüfe, ob das Frontend weiterhin korrekt mit dem Backend kommuniziert. Ändere die URL in der ConfigMap testweise und starte den Pod neu, um zu sehen, wie die Konfiguration wirksam wird:

```bash
kubectl rollout restart deployment/recipes-frontend
```

### 4. *(Optional)* ConfigMap als YAML verwalten

Du hast die ConfigMap bisher per CLI erstellt. Überlege, wie du sie stattdessen als eigenständige YAML-Datei definieren und mit `kubectl apply -f` anlegen könntest. Schau dir dazu die Ausgabe von `kubectl get configmap frontend-config -o yaml` an — was davon brauchst du, was kannst du weglassen?

## 📚 Selbstlernmaterial

* [Kubernetes ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)
* [Umgebungsvariablen aus ConfigMaps beziehen](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data)
* [The Twelve-Factor App: Config](https://12factor.net/config)

## 🤔 Reflexionsfragen

* Was ist der Vorteil einer ConfigMap gegenüber hart kodierten Umgebungsvariablen im Deployment?
* Wann wird eine Änderung an einer ConfigMap wirksam? Muss der Pod neu gestartet werden?
* Für welche Art von Daten ist eine ConfigMap geeignet — und für welche nicht?
