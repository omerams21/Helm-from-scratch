## Part 1 – Explanation (Helm fundamentals)

### Chart.yaml
`Chart.yaml` holds chart metadata:
- `apiVersion: v2` – Helm 3 chart format
- `name` – chart name
- `version` – chart version (ex: 0.1.0)
- `appVersion` – application version (used here to match the container tag)

### values.yaml
`values.yaml` provides default configurable inputs for templates.
We define:
- `image.repository` + `image.tag` – container image
- `image.command` – arguments passed to the container (http-echo uses args like `-text=...`)
- `service.port/type` – service exposure
- `deployment/daemonset/job/cronjob.enabled` – feature flags to enable resources
- `configmap.data` and `secret.stringData` – data sources rendered into Kubernetes objects

### Templates concept
Helm templates are Kubernetes YAML with placeholders.
Examples:
- `{{ .Values.image.repository }}` reads from `values.yaml`
- `{{ .Release.Name }}` is the release name given at install time
- `include "myapp.fullname" .` uses helper functions to generate consistent names

Each resource template is wrapped with `{{- if ...enabled }}` so we can turn it on/off via values.
