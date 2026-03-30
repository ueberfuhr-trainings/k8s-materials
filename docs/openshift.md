---
layout: default
title: Was ist OpenShift?
---

# Was ist OpenShift?

**Red Hat OpenShift** ist eine Enterprise-Kubernetes-Plattform, die Kubernetes um zusätzliche Funktionen für Entwickler und Betrieb erweitert. OpenShift basiert auf Kubernetes, fügt aber eine Reihe von Werkzeugen, Sicherheitsfunktionen und Abstraktionen hinzu, die den Betrieb in Unternehmen erleichtern.

## Container-Engine

Seit Version 4 verwendet OpenShift **CRI-O** als Container-Runtime (statt der Docker-Engine in früheren Versionen). CRI-O ist eine schlanke, stabile Container-Laufzeit, die speziell für Kubernetes entwickelt wurde und sich auf die sichere und zuverlässige Ausführung von Containern konzentriert.

## OpenShift vs. Kubernetes

OpenShift enthält Kubernetes vollständig, ergänzt es aber um wichtige Funktionen:

| Bereich | Kubernetes | OpenShift |
|---------|-----------|-----------|
| **Web-Konsole** | Kubernetes Dashboard (einfach) | Umfangreiche Web-Konsole mit Entwickler- und Administrator-Perspektive |
| **Routing** | Ingress (erfordert zusätzlichen Ingress Controller) | **Routes** — integriertes, einfach zu konfigurierendes Routing mit TLS-Terminierung |
| **CI/CD** | Keine integrierte Lösung | Integrierte **Pipelines** (basierend auf Tekton), **Builds** direkt aus Source Code (S2I) |
| **Sicherheit** | Grundlegende RBAC | Erweiterte Sicherheit: **Security Context Constraints (SCCs)**, standardmäßig kein Root |
| **Image Registry** | Keine integrierte Registry | Integrierte **Image Registry** mit Image Streams |
| **Operatoren** | Operator-Support vorhanden | **OperatorHub** — kuratierter Marketplace für Operatoren |
| **Monitoring** | Manuell einzurichten (Prometheus, Grafana) | **Integriertes Monitoring** und Alerting (vorkonfiguriert) |
| **Logging** | Manuell einzurichten (EFK-Stack) | Integrierte **Log-Aggregation** |
| **Support** | Community-Support | **Enterprise-Support** von Red Hat mit langfristiger Stabilität und Sicherheitspatches |
| **Updates** | Manuell, komplex | Automatisierte **Cluster-Updates** über den Operator Lifecycle Manager |

### Routes vs. Ingress

Eine der auffälligsten Unterschiede im Alltag: OpenShift verwendet **Routes** statt Ingress-Ressourcen. Routes bieten:

- Einfache TLS-Terminierung (`tls.termination: edge`)
- Automatische Generierung von Hostnamen
- Integriertes Traffic-Management

### Security Context Constraints (SCCs)

OpenShift erzwingt standardmäßig strengere Sicherheitsrichtlinien als Kubernetes:

- Container laufen **nicht als Root** (daher das `nginx-unprivileged`-Image in unseren Übungen)
- Container erhalten eine **zufällige UID** aus einem vordefinierten Bereich
- Dateisysteme sind standardmäßig **read-only**

### Operatoren

Operatoren sind Kubernetes-native Anwendungen, die die Verwaltung komplexer Anwendungen automatisieren. Sie kapseln anwendungsspezifisches Wissen und bewährte Methoden und ermöglichen die automatisierte Bereitstellung, Konfiguration, Skalierung und Lebenszyklusverwaltung. OpenShift stellt mit dem **OperatorHub** einen kuratierten Marketplace bereit, über den Operatoren direkt installiert werden können.

## Hochverfügbarkeit und Fehlertoleranz

OpenShift nutzt die Kubernetes-Mechanismen für Hochverfügbarkeit und erweitert sie:

- **Replica Sets** und **Pod-Autoskalierung** für Anwendungsverfügbarkeit
- **Multi-Master-Konfigurationen** für Redundanz auf der Control-Plane-Ebene
- Automatisierte **Rolling Updates** und **Rolling Deployments** für Minimierung von Ausfallzeiten
- **Failover-Funktionen** bei Ausfall einzelner Knoten

## Installationsoptionen

OpenShift kann auf verschiedene Arten installiert werden:

### Public Cloud (Managed)

**OpenShift Cloud Services** — schlüsselfertige Anwendungsplattform, die von Red Hat und dem Cloud-Anbieter gemeinsam verwaltet wird:

| Service | Cloud-Anbieter |
|---------|---------------|
| **ROSA** (Red Hat OpenShift on AWS) | Amazon Web Services |
| **ARO** (Azure Red Hat OpenShift) | Microsoft Azure |
| **RHOIC** (Red Hat OpenShift on IBM Cloud) | IBM Cloud |

Vorteile: Kein eigener Betrieb, automatische Updates, SLA-gesichert.

### Private / On-Premises

**Red Hat OpenShift Platform Plus** — selbstverwaltete Bereitstellung:

- **Bare Metal** — direkt auf eigener Hardware
- **VMware vSphere** — auf virtualisierter Infrastruktur
- **Private Cloud** (OpenStack etc.)
- **Edge** — für verteilte Standorte

Vorteile: Volle Kontrolle, Datenhoheit, Compliance-Anforderungen.

### Lokal (Entwicklung & Lernen)

**[Red Hat OpenShift Local](https://developers.redhat.com/products/openshift-local/overview)** (ehemals CodeReady Containers / CRC):

- Einzelknoten-Installation eines vollständigen OpenShift-Clusters
- Läuft auf dem lokalen Rechner (Linux, macOS, Windows)
- Gedacht für **Lernzwecke und Entwicklung**, nicht für Produktion
- Kostenlos verfügbar über das [Red Hat Developer-Programm](https://developers.redhat.com/)

```bash
# Beispiel: OpenShift Local starten
crc setup
crc start
# Anschließend mit oc einloggen
eval $(crc oc-env)
oc login -u developer https://api.crc.testing:6443
```

## Weiterführende Links

* [Red Hat OpenShift Dokumentation](https://docs.openshift.com/)
* [OpenShift Local herunterladen](https://developers.redhat.com/products/openshift-local/overview)
* [OpenShift vs. Kubernetes (Red Hat)](https://www.redhat.com/en/topics/containers/what-is-openshift)
* [OperatorHub.io](https://operatorhub.io/)
