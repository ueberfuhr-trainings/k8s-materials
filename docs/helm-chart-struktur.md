---
layout: default
title: Aufbau eines Helm Charts
---

# Aufbau eines Helm Charts

Ein Helm Chart ist ein Verzeichnis mit einer festen Struktur. Hier sind alle Dateien und Verzeichnisse, die ein Chart enthalten kann:

```
my-app/
├── Chart.yaml            # Metadaten und Abhängigkeiten (erforderlich)
├── Chart.lock            # Exakte Versionen der aufgelösten Dependencies
├── values.yaml           # Standardwerte (überschreibbar beim Install)
├── values.schema.json    # JSON Schema zur Validierung der Values
├── .helmignore           # Dateien, die beim Verpacken ignoriert werden
├── LICENSE               # Lizenz des Charts
├── README.md             # Dokumentation des Charts (wird auf Artifact Hub angezeigt)
├── templates/            # Kubernetes-Manifeste als Go-Templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl      # Wiederverwendbare Template-Snippets (kein Output)
│   ├── NOTES.txt         # Hinweistext nach Install/Upgrade (Go-Template)
│   └── tests/            # Helm-Tests (Pods, die nach Install laufen)
│       └── test-connection.yaml
├── charts/               # Subcharts (Dependencies, heruntergeladen oder lokal)
│   └── postgresql/
└── crds/                 # Custom Resource Definitions (werden vor Templates installiert)
    └── my-crd.yaml
```

## Dateien im Detail

### Chart.yaml (erforderlich)

Metadaten des Charts — Name, Version, Abhängigkeiten:

```yaml
apiVersion: v2              # v2 = Helm 3 (v1 = Helm 2, veraltet)
name: my-app
version: 1.0.0              # Version des Charts selbst (SemVer)
appVersion: "2.3.1"         # Version der verpackten Anwendung
description: Recipes Backend für Kubernetes
type: application            # "application" oder "library"
dependencies:
  - name: postgresql
    version: "~15.2"
    repository: oci://registry-1.docker.io/bitnamicharts
    condition: postgresql.enabled
```

- **`version`** — die Version des Charts (wird bei `helm package` und in der Registry verwendet)
- **`appVersion`** — die Version der Anwendung, die das Chart installiert (rein informativ)
- **`type`** — `application` (installierbar) oder `library` (nur als Dependency nutzbar, erzeugt keine eigenen Ressourcen)
- **`dependencies`** — andere Charts, die automatisch mit installiert werden

### values.yaml

Standardwerte, die Nutzer beim Installieren überschreiben können:

```yaml
image:
  repository: ralfueberfuhr/recipes-backend
  tag: latest-dev
replicas: 1
service:
  port: 8080
```

In Templates werden diese Werte mit {% raw %}`{{ .Values.image.tag }}`{% endraw %} referenziert.

### values.schema.json

Optionales JSON Schema, das Values beim `helm install` validiert. Erzwingt z.B. dass `replicas` eine Zahl ist oder `image.tag` gesetzt sein muss:

```json
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["image"],
  "properties": {
    "replicas": { "type": "integer", "minimum": 1 },
    "image": {
      "type": "object",
      "required": ["repository", "tag"],
      "properties": {
        "repository": { "type": "string" },
        "tag": { "type": "string" }
      }
    }
  }
}
```

### templates/

Kubernetes-Manifeste als Go-Templates. Jede `.yaml`-Datei in diesem Verzeichnis wird gerendert und auf den Cluster angewendet.

#### templates/NOTES.txt

Text, der nach `helm install` und `helm upgrade` auf der Konsole angezeigt wird. Unterstützt Go-Templating und wird typischerweise genutzt, um dem Nutzer die nächsten Schritte zu zeigen:

{% raw %}
```
Dein Release {{ .Release.Name }} wurde installiert!

So erreichst du die Anwendung:
  export URL=$(kubectl get route {{ .Release.Name }} -o jsonpath='{.spec.host}')
  echo "Öffne https://$URL"
```
{% endraw %}

#### templates/_helpers.tpl

Dateien mit Unterstrich (`_`) erzeugen kein Kubernetes-Manifest. Sie enthalten wiederverwendbare Template-Funktionen, die von anderen Templates mit {% raw %}`{{ include }}`{% endraw %} aufgerufen werden:

{% raw %}
```
{{- define "my-app.fullname" -}}
{{ .Release.Name }}-{{ .Chart.Name }}
{{- end -}}

{{- define "my-app.labels" -}}
app.kubernetes.io/name: {{ .Chart.Name }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion }}
{{- end -}}
```
{% endraw %}

Verwendung in einem Template:

{% raw %}
```yaml
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
```
{% endraw %}

#### templates/tests/

Pods mit der Annotation `helm.sh/hook: test`, die nach der Installation mit `helm test my-release` ausgeführt werden:

{% raw %}
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: {{ .Release.Name }}-test
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: curl-test
      image: curlimages/curl
      command: ['curl', '--fail', 'http://{{ .Release.Name }}:8080/health']
  restartPolicy: Never
```
{% endraw %}

Typische Einsätze: HTTP-Anfrage an den Service, Datenbankverbindung prüfen, Smoke-Test.

### charts/

Subcharts (Dependencies). Werden durch `helm dependency update` aus den in `Chart.yaml` definierten Dependencies heruntergeladen, oder manuell als Verzeichnisse abgelegt.

### crds/ — Custom Resource Definitions

CRDs werden **vor** allen Templates installiert, aber bei `helm upgrade` **nicht** aktualisiert (Sicherheitsmaßnahme). CRD-Updates müssen manuell erfolgen. Bei `helm uninstall` werden CRDs **nicht gelöscht** — das verhindert, dass ein versehentliches Deinstallieren alle Custom Resources im Cluster zerstört.

**Was sind CRDs?** Kubernetes kennt von Haus aus Ressourcen wie `Pod`, `Service` oder `Deployment`. Eine CRD erweitert die Kubernetes-API um **eigene Ressourcentypen**. Zum Beispiel könnte ein Datenbank-Operator einen Typ `PostgresCluster` definieren — danach kann man in YAML-Manifesten `kind: PostgresCluster` verwenden, als wäre es eine eingebaute Ressource.

Beispiel: Ein Chart für einen Monitoring-Operator liefert eine CRD mit, die den Typ `MonitoringRule` einführt:

```yaml
# crds/monitoring-rule-crd.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: monitoringrules.example.com
spec:
  group: example.com
  names:
    kind: MonitoringRule
    plural: monitoringrules
    singular: monitoringrule
    shortNames:
      - mr
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                metric:
                  type: string
                threshold:
                  type: integer
```

Sobald das Chart installiert ist, kennt der Cluster den neuen Typ. In `templates/` können dann konkrete Instanzen davon angelegt werden:

```yaml
# templates/alert-rule.yaml
apiVersion: example.com/v1
kind: MonitoringRule
metadata:
  name: high-cpu-alert
spec:
  metric: cpu_usage_percent
  threshold: 90
```

**Warum `crds/` und nicht `templates/`?** Helm muss die CRD zuerst registrieren, bevor Templates Instanzen davon erstellen können. Dateien in `crds/` werden deshalb immer zuerst angewendet.

### .helmignore

Funktioniert wie `.gitignore`: Dateien und Verzeichnisse, die beim `helm package` nicht ins Archiv sollen:

```
.git/
*.swp
*.tmp
.idea/
.vscode/
```

### Chart.lock

Wird automatisch von `helm dependency update` erzeugt. Enthält die exakt aufgelösten Versionen aller Dependencies (vergleichbar mit `package-lock.json` bei npm). Sollte ins Git eingecheckt werden, damit alle Teammitglieder dieselben Versionen erhalten.
