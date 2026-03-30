---
layout: default
title: Was ist Kubernetes?
---

# Was ist Kubernetes?

Kubernetes (oft als **K8s** abgekürzt) ist eine Open-Source-Plattform zur **Container-Orchestrierung**, die ursprünglich von Google entwickelt und 2014 als Open-Source-Projekt veröffentlicht wurde. Heute wird Kubernetes von der [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/) verwaltet.

Kubernetes automatisiert die Bereitstellung, Skalierung und Verwaltung von containerisierten Anwendungen über Cluster von Maschinen hinweg.

## Kernkonzepte

### Cluster-Architektur

Ein Kubernetes-Cluster besteht aus:

- **Control Plane (Master)** — verwaltet den Cluster, plant Workloads und überwacht den Zustand
- **Worker Nodes** — führen die eigentlichen Container aus

```
┌──────────────────────────────────────────────────────┐
│                        Kubernetes Cluster            │
│                                                      │
│      ┌───────────────────────────────────────┐       │
│      │          Control Plane (Master)       │       │
│      │                                       │       │
│      │  ┌───────────┐  ┌──────────────────┐  │       │
│      │  │ API Server│  │    Scheduler     │  │       │
│      │  └───────────┘  └──────────────────┘  │       │
│      │  ┌───────────┐  ┌──────────────────┐  │       │
│      │  │   etcd    │  │Controller Manager│  │       │
│      │  └───────────┘  └──────────────────┘  │       │
│      └───────────────────────────────────────┘       │
│                          │                           │
│            ┌─────────────┼─────────────┐             │
│            ▼             ▼             ▼             │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐  │
│  │ Worker Node 1│ │ Worker Node 2│ │ Worker Node 3│  │
│  │              │ │              │ │              │  │
│  │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌──────────┐ │  │
│  │ │  kubelet │ │ │ │  kubelet │ │ │ │  kubelet │ │  │
│  │ └──────────┘ │ │ └──────────┘ │ │ └──────────┘ │  │
│  │ ┌──────────┐ │ │ ┌──────────┐ │ │ ┌──────────┐ │  │
│  │ │kube-proxy│ │ │ │kube-proxy│ │ │ │kube-proxy│ │  │
│  │ └──────────┘ │ │ └──────────┘ │ │ └──────────┘ │  │
│  │              │ │              │ │              │  │
│  │ ┌────┐┌────┐ │ │ ┌────┐       │ │ ┌────┐┌────┐ │  │
│  │ │Pod ││Pod │ │ │ │Pod │       │ │ │Pod ││Pod │ │  │
│  │ └────┘└────┘ │ │ └────┘       │ │ └────┘└────┘ │  │
│  └──────────────┘ └──────────────┘ └──────────────┘  │
└──────────────────────────────────────────────────────┘
```

**Control Plane-Komponenten:**

| Komponente | Aufgabe |
|------------|---------|
| **API Server** | Zentrale Schnittstelle — alle Befehle (`kubectl`, `oc`) gehen hierüber |
| **Scheduler** | Entscheidet, auf welchem Knoten ein neuer Pod läuft |
| **Controller Manager** | Überwacht den Ist-Zustand und stellt den Soll-Zustand her (z.B. Replica-Anzahl) |
| **etcd** | Verteilter Key-Value-Store — speichert den gesamten Cluster-Zustand |

**Worker-Node-Komponenten:**

| Komponente | Aufgabe |
|------------|---------|
| **kubelet** | Agent auf jedem Knoten — startet und überwacht Pods |
| **kube-proxy** | Netzwerk-Proxy — leitet Traffic an die richtigen Pods weiter |

### Ressourcen

Kubernetes verwaltet Anwendungen über deklarative Ressourcen wie Pods, Deployments, Services, ConfigMaps und mehr. Eine ausführliche Übersicht findest Du unter [Kubernetes-Ressourcen](kubernetes-resources.html).

## Docker vs. Kubernetes — wofür was?

Docker und Kubernetes lösen unterschiedliche Probleme und ergänzen sich:

| | Docker (inkl. Compose) | Kubernetes |
|---|---|---|
| **Zweck** | Container bauen und lokal ausführen | Container über mehrere Maschinen hinweg orchestrieren |
| **Skalierung** | Manuell (z.B. `replicas` in Compose) | Automatisch (Horizontal Pod Autoscaler, Replica Sets) |
| **Hochverfügbarkeit** | Nicht vorgesehen — bei Ausfall eines Hosts ist alles weg | Pods werden automatisch auf gesunde Knoten verschoben |
| **Rollouts** | Manuell, mit Downtime | Rolling Updates, Blue-Green-Deployments, Canary Releases |
| **Netzwerk** | Einfaches Bridge-Netzwerk | Cluster-weites Netzwerk mit Service Discovery und DNS |
| **Konfiguration** | `.env`-Dateien, `docker-compose.yml` | ConfigMaps, Secrets, deklarative YAML-Manifeste |
| **Health Checks** | `HEALTHCHECK` im Dockerfile (begrenzt) | Liveness-, Readiness- und Startup-Probes mit automatischem Neustart |
| **Speicher** | Bind Mounts, Volumes | Persistent Volumes mit Storage Classes und dynamischer Provisionierung |
| **Geeignet für** | Entwicklung, CI, einfache Deployments | Produktion, Microservices, hohe Verfügbarkeit |

### Wann Docker (Compose)?

- **Lokale Entwicklung** — schnell einen Stack aus mehreren Services starten
- **CI/CD-Pipelines** — Container bauen und testen
- **Einfache Deployments** — einzelner Host, keine Hochverfügbarkeit nötig

### Wann Kubernetes?

- **Produktion** — wenn Hochverfügbarkeit, automatische Skalierung und Selbstheilung gebraucht werden
- **Microservices** — viele Services, die unabhängig deployed und skaliert werden
- **Multi-Team** — Namespaces, RBAC und Resource Quotas für organisatorische Trennung
- **Hybrid/Multi-Cloud** — einheitliche Plattform über verschiedene Infrastrukturen hinweg

### Zusammenspiel

Docker und Kubernetes sind keine Konkurrenten:

1. **Docker** baut die Container-Images (Dockerfile → `docker build`)
2. Die Images werden in eine **Registry** gepusht (Docker Hub, ghcr.io, etc.)
3. **Kubernetes** zieht die Images und orchestriert deren Ausführung im Cluster

> **Hinweis:** Kubernetes selbst verwendet seit Version 1.24 nicht mehr die Docker-Engine als Container-Runtime, sondern setzt auf **containerd** oder **CRI-O**. Docker-Images funktionieren aber weiterhin, da sie dem OCI-Standard entsprechen.

## Kubernetes lokal ausprobieren

Mit [Minikube](https://minikube.sigs.k8s.io/) kannst Du einen lokalen Kubernetes-Cluster auf Deinem Rechner starten — ideal zum Lernen und Experimentieren. Minikube läuft auf Linux, macOS und Windows und unterstützt verschiedene Virtualisierungs-Backends (Docker, VirtualBox, Hyper-V etc.).

```bash
minikube start
kubectl get nodes
```

## Weiterführende Links

* [Offizielle Kubernetes-Dokumentation](https://kubernetes.io/docs/home/)
* [Kubernetes Concepts](https://kubernetes.io/docs/concepts/)
* [Interactive Tutorials](https://kubernetes.io/docs/tutorials/)
* [Minikube — Lokaler Kubernetes-Cluster](https://minikube.sigs.k8s.io/docs/start/)
