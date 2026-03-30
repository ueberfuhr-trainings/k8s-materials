---
layout: default
title: kubectl/oc Cheat Sheet
---

# kubectl / oc — Cheat Sheet

## kubectl vs. oc

**kubectl** ist das offizielle CLI für Kubernetes. **oc** ist das CLI für OpenShift und ist **vollständig kompatibel** mit kubectl — jeder `kubectl`-Befehl funktioniert auch mit `oc`. Zusätzlich bietet `oc` OpenShift-spezifische Befehle (z.B. `oc new-project`, `oc login`).

> In den Beispielen unten wird `oc` verwendet. Du kannst `oc` jederzeit durch `kubectl` ersetzen, sofern Du mit einem reinen Kubernetes-Cluster arbeitest.

## Verbindung & Kontext

```bash
# Bei OpenShift einloggen
oc login https://api.mein-cluster:6443

# Aktuellen Kontext anzeigen
oc config current-context

# Verfügbare Kontexte anzeigen
oc config get-contexts

# Kontext wechseln
oc config use-context <context-name>

# Aktuellen Benutzer anzeigen
oc whoami
```

## Projects / Namespaces

```bash
# Neues Project erstellen (OpenShift)
oc new-project mein-projekt --display-name="Mein Projekt"

# Namespace erstellen (Kubernetes)
kubectl create namespace mein-projekt

# Aktuelles Project/Namespace wechseln
oc project mein-projekt

# Alle Projects/Namespaces anzeigen
oc get projects        # OpenShift
kubectl get namespaces  # Kubernetes
```

## Ressourcen anzeigen

```bash
# Pods anzeigen
oc get pods
oc get pods -o wide                  # mit zusätzlichen Details (Node, IP)
oc get pods -w                       # Watch-Modus (live)

# Alle Ressourcentypen anzeigen
oc get all

# Bestimmte Ressource anzeigen
oc get deployments
oc get services
oc get configmaps
oc get secrets
oc get routes                        # nur OpenShift
oc get ingress                       # Kubernetes

# Ressourcen über alle Namespaces
oc get pods --all-namespaces
oc get pods -A                       # Kurzform

# YAML einer Ressource ausgeben
oc get deployment mein-deployment -o yaml
```

## Ressourcen erstellen & anwenden

```bash
# Ressource aus YAML-Datei erstellen/aktualisieren
oc apply -f deployment.yaml

# Alle YAML-Dateien in einem Ordner anwenden
oc apply -f ./manifeste/

# Ressource direkt erstellen
oc create deployment mein-app --image=mein-image:latest
```

## Ressourcen bearbeiten & löschen

```bash
# Ressource bearbeiten (öffnet Editor)
oc edit deployment mein-deployment

# Ressource löschen
oc delete pod mein-pod
oc delete -f deployment.yaml

# Alle Pods in einem Namespace löschen
oc delete pods --all
```

## Details & Debugging

```bash
# Detaillierte Informationen (Events, Conditions)
oc describe pod <pod-name>
oc describe deployment <deployment-name>
oc describe node <node-name>

# Logs eines Pods anzeigen
oc logs <pod-name>
oc logs <pod-name> -f               # Follow (live)
oc logs <pod-name> -c <container>   # Bestimmter Container im Pod
oc logs <pod-name> --previous       # Logs des vorherigen Containers (nach Restart)

# Shell in einem laufenden Pod öffnen
oc exec -it <pod-name> -- /bin/sh

# Befehl in einem Pod ausführen
oc exec <pod-name> -- env

# Port-Forwarding (lokaler Zugriff auf Pod)
oc port-forward <pod-name> 8080:8080
```

## Deployments verwalten

```bash
# Deployment skalieren
oc scale deployment mein-deployment --replicas=3

# Rolling Restart (alle Pods neu starten)
oc rollout restart deployment mein-deployment

# Rollout-Status überwachen
oc rollout status deployment mein-deployment

# Rollout-Historie anzeigen
oc rollout history deployment mein-deployment

# Rollback zur vorherigen Version
oc rollout undo deployment mein-deployment
```

## ConfigMaps & Secrets

```bash
# ConfigMap aus Literal erstellen
oc create configmap meine-config --from-literal=KEY=value

# ConfigMap aus Datei erstellen
oc create configmap meine-config --from-file=config.properties

# Secret erstellen
oc create secret generic mein-secret --from-literal=PASSWORD=geheim

# Docker-Registry-Secret erstellen (Pull Secret)
oc create secret docker-registry mein-pull-secret \
  --docker-server=docker.io \
  --docker-username=<user> \
  --docker-password=<token>

# Secret-Inhalt anzeigen (Base64-dekodiert)
oc get secret mein-secret -o jsonpath='{.data.PASSWORD}' | base64 -d
```

## Netzwerk & Routes

```bash
# Service erstellen (Expose eines Deployments)
oc expose deployment mein-deployment --port=8080

# Route erstellen (nur OpenShift)
oc expose service mein-service
oc get routes

# Alle Endpunkte eines Services anzeigen
oc get endpoints mein-service
```

## Nützliche Flags

| Flag | Beschreibung |
|------|-------------|
| `-n <namespace>` | Namespace/Project angeben |
| `-o yaml` | Ausgabe als YAML |
| `-o json` | Ausgabe als JSON |
| `-o wide` | Erweiterte Ausgabe (z.B. Node, IP) |
| `-o jsonpath='{...}'` | Bestimmte Felder extrahieren |
| `-w` / `--watch` | Live-Aktualisierung |
| `-l app=mein-app` | Nach Label filtern |
| `--dry-run=client -o yaml` | YAML generieren ohne anzuwenden |

## YAML generieren (statt von Hand schreiben)

```bash
# Deployment-YAML generieren
oc create deployment mein-app --image=mein-image:latest \
  --dry-run=client -o yaml > deployment.yaml

# Service-YAML generieren
oc expose deployment mein-app --port=8080 \
  --dry-run=client -o yaml > service.yaml
```

## Weiterführende Links

* [kubectl Referenz](https://kubernetes.io/docs/reference/kubectl/)
* [kubectl Cheat Sheet (Kubernetes.io)](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
* [oc CLI-Referenz (Red Hat)](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/cli_tools/openshift-cli-oc)
