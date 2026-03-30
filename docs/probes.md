---
layout: default
title: Health Checks (Probes)
---

# Health Checks (Probes)

Kubernetes überwacht die Gesundheit von Containern über **Probes** — regelmäßige Prüfungen, die Kubernetes automatisch durchführt. Basierend auf dem Ergebnis entscheidet Kubernetes, ob ein Container Traffic empfangen darf, neu gestartet werden muss oder noch Zeit zum Starten braucht.

> **Wichtig:** Probes werden pro **Container** konfiguriert, nicht pro Pod. Ein Pod mit mehreren Containern (z.B. App + Sidecar) kann für jeden Container eigene Probes definieren.

> **Hinweis zu Docker `HEALTHCHECK`:** Kubernetes ignoriert die `HEALTHCHECK`-Anweisung aus dem Dockerfile vollständig. Diese wird nur von der Docker-Engine selbst ausgewertet (z.B. bei `docker run` oder Docker Compose). In Kubernetes müssen Health Checks immer als Probes im Pod-Spec definiert werden.

## Die drei Probe-Typen

### Startup Probe

Prüft, ob der Container **erfolgreich gestartet** ist. Solange die Startup Probe nicht erfolgreich war, werden Liveness und Readiness Probes **nicht ausgeführt**.

**Zweck:** Schutz von Containern mit langer Startzeit. Ohne Startup Probe könnte die Liveness Probe einen noch startenden Container fälschlicherweise als tot betrachten und neu starten — was zu einer Endlosschleife führen kann.

**Bei Fehlschlag:** Der Container wird neu gestartet (wie bei Liveness).

### Liveness Probe

Prüft, ob der Container **noch lebt**. Eine fehlschlagende Liveness Probe bedeutet: der Container hängt, ist in einem Deadlock oder kann sich nicht mehr erholen.

**Bei Fehlschlag:** Kubernetes **startet den Container neu** (im selben Pod).

**Typische Szenarien:** Deadlocks, Endlosschleifen, Speicherlecks, die den Prozess blockieren.

### Readiness Probe

Prüft, ob der Container **bereit ist, Traffic zu empfangen**. Ein Container kann am Leben sein (Liveness OK), aber noch nicht bereit — z.B. weil er noch Caches aufbaut oder auf eine Datenbankverbindung wartet.

**Bei Fehlschlag:** Der Pod wird **aus dem Service entfernt** — er erhält keinen Traffic mehr, wird aber **nicht** neu gestartet. Sobald die Probe wieder erfolgreich ist, wird der Pod dem Service wieder hinzugefügt.

## Zusammenspiel der Probes

```
Container startet
       │
       ▼
┌──────────────┐    fehlgeschlagen     Container wird
│ Startup Probe│──────────────────────► neu gestartet
└──────┬───────┘
       │ erfolgreich
       ▼
┌──────────────┐    fehlgeschlagen     Container wird
│Liveness Probe│──────────────────────► neu gestartet
└──────┬───────┘
       │ parallel
       ▼
┌──────────────┐    fehlgeschlagen     Pod wird aus
│Readiness Pr. │──────────────────────► Service entfernt
└──────────────┘    (kein Neustart)
```

## Probe-Mechanismen

Kubernetes unterstützt drei Mechanismen für Probes:

| Mechanismus | Beschreibung | Beispiel |
|-------------|-------------|---------|
| **httpGet** | HTTP-GET-Request auf einen Endpunkt. Erfolgreich bei Statuscode 200–399 | Health-Endpunkt einer Web-Anwendung |
| **tcpSocket** | Prüft, ob ein TCP-Port offen ist | Datenbank-Port |
| **exec** | Führt ein Kommando im Container aus. Erfolgreich bei Exit-Code 0 | `pg_isready` für PostgreSQL |

## Konfigurationsparameter

```yaml
livenessProbe:
  httpGet:
    path: /q/health/live
    port: 9000
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 3
  successThreshold: 1
```

| Parameter | Beschreibung | Default |
|-----------|-------------|---------|
| `initialDelaySeconds` | Wartezeit nach Container-Start vor der ersten Prüfung | 0 |
| `periodSeconds` | Intervall zwischen den Prüfungen | 10 |
| `timeoutSeconds` | Maximale Wartezeit auf Antwort | 1 |
| `failureThreshold` | Anzahl aufeinanderfolgender Fehlschläge, bevor die Aktion ausgelöst wird | 3 |
| `successThreshold` | Anzahl aufeinanderfolgender Erfolge, bevor der Container als gesund gilt | 1 |

> **Tipp:** Wähle `initialDelaySeconds` und `failureThreshold` nicht zu aggressiv. Zu kurze Werte führen dazu, dass Container bei kurzfristiger Last unnötig neu gestartet werden. Zu lange Werte verzögern die Erkennung tatsächlicher Probleme.

## Beispiele für verschiedene Anwendungstypen

| Anwendung | Startup Probe | Liveness Probe | Readiness Probe |
|-----------|--------------|----------------|-----------------|
| **NGINX (statisches Frontend)** | — (startet schnell) | `httpGet` auf `/` Port 8080 | `httpGet` auf `/` Port 8080 |
| **Quarkus-Backend** | `httpGet` auf `/q/health/started` Port 9000 | `httpGet` auf `/q/health/live` Port 9000 | `httpGet` auf `/q/health/ready` Port 9000 (prüft inkl. DB-Verbindung) |
| **Spring Boot-Backend** | `httpGet` auf `/actuator/health/liveness` Port 8080 | `httpGet` auf `/actuator/health/liveness` Port 8080 | `httpGet` auf `/actuator/health/readiness` Port 8080 (prüft inkl. DB-Verbindung) |
| **PostgreSQL** | `exec`: `pg_isready -U <user>` | `exec`: `pg_isready -U <user>` | `exec`: `pg_isready -U <user>` |

**Erläuterungen:**

- **NGINX** startet sehr schnell, daher ist keine Startup Probe nötig. Liveness und Readiness können identisch sein, da NGINX entweder funktioniert oder nicht.
- **Quarkus** bietet über SmallRye Health separate Endpunkte auf einem dedizierten Management-Port (9000). Die Readiness-Probe prüft zusätzlich externe Abhängigkeiten wie die Datenbankverbindung.
- **Spring Boot** bietet über Actuator ähnliche Endpunkte. Seit Spring Boot 2.3 werden Liveness und Readiness als separate Health Groups unterstützt.
- **PostgreSQL** hat keine HTTP-Endpunkte, daher wird `exec` mit dem mitgelieferten Tool `pg_isready` verwendet.

## Weiterführende Links

* [Kubernetes: Liveness, Readiness und Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
* [Kubernetes: Pod Lifecycle](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
* [Quarkus SmallRye Health](https://quarkus.io/guides/smallrye-health)
* [Spring Boot Actuator Health](https://docs.spring.io/spring-boot/reference/actuator/endpoints.html#actuator.endpoints.health)
