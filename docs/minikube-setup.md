---
layout: default
title: Minikube-Setup für die Schulung
---

# Minikube-Setup für die Schulung

Diese Anleitung beschreibt, wie Du einen lokalen Kubernetes-Cluster mit Minikube einrichtest und für die Übungen vorbereitest.

## 1. Minikube installieren

### macOS

```bash
brew install minikube
```

### Windows

```bash
choco install minikube
```

Alternativ: [Installer herunterladen](https://minikube.sigs.k8s.io/docs/start/)

### Linux

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

## 2. kubectl installieren

Falls `kubectl` noch nicht installiert ist:

### macOS

```bash
brew install kubectl
```

### Windows

```bash
choco install kubernetes-cli
```

### Linux

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install kubectl /usr/local/bin/kubectl
```

## 3. Minikube starten

Starte einen lokalen Cluster:

```bash
minikube start
```

Prüfe, ob der Cluster läuft:

```bash
kubectl cluster-info
kubectl get nodes
```

## 4. Minikube konfigurieren

### Ingress-Controller aktivieren

Für die Übungen benötigen wir einen Ingress Controller. Minikube liefert den NGINX Ingress Controller als Addon mit:

```bash
minikube addons enable ingress
```

Prüfe, ob der Ingress Controller läuft:

```bash
kubectl get pods -n ingress-nginx
```

### Kubernetes-Dashboard aktivieren

Minikube bringt ein Dashboard-Addon mit, das eine grafische Übersicht über den Cluster bietet:

```bash
minikube addons enable dashboard
```

Das Dashboard kannst Du jederzeit im Browser öffnen:

```bash
minikube dashboard
```

Der Befehl startet einen Proxy und öffnet das Dashboard automatisch im Browser. Mit `Ctrl+C` wird der Proxy wieder beendet.

### Namespace erstellen und als Default setzen

Erstelle einen eigenen Namespace für die Schulung:

```bash
kubectl create namespace schulung
```

Setze diesen Namespace als Default für den aktuellen Kontext, damit Du nicht bei jedem Befehl `-n schulung` angeben musst:

```bash
kubectl config set-context --current --namespace=schulung
```

Prüfe, ob der Namespace korrekt gesetzt ist:

```bash
kubectl config view --minify --output 'jsonpath={..namespace}'
```

Die Ausgabe sollte `schulung` sein. Ab jetzt arbeiten alle `kubectl`-Befehle automatisch in diesem Namespace.

## 5. Zusammenfassung

Nach diesen Schritten ist Deine lokale Umgebung bereit:

- [x] Minikube und kubectl sind installiert
- [x] Der Cluster läuft lokal
- [x] Der Ingress Controller und das Dashboard sind aktiviert
- [x] Der Namespace `schulung` ist erstellt und als Default gesetzt

Du kannst jetzt mit [Übung 1: Backend-Deployment](../issues/kubernetes/01-backend-deployment.html) starten.

## Nützliche Befehle

| Befehl | Beschreibung |
|--------|-------------|
| `minikube start` | Cluster starten |
| `minikube stop` | Cluster stoppen (Zustand bleibt erhalten) |
| `minikube delete` | Cluster vollständig löschen |
| `minikube dashboard` | Kubernetes-Dashboard im Browser öffnen |
| `minikube tunnel` | LoadBalancer-Services lokal erreichbar machen |
| `minikube service <name> --url` | URL eines Services anzeigen |

## Selbstlernmaterial

* [Minikube-Dokumentation](https://minikube.sigs.k8s.io/docs/)
* [kubectl installieren](https://kubernetes.io/docs/tasks/tools/)
* [Kubernetes Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
