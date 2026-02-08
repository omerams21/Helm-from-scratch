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

## Part 2 – Explanation

`helm upgrade --install` ensures the release is created if it doesn't exist, or upgraded if it already exists.
Deploying into a dedicated namespace (`-n myapp --create-namespace`) keeps the Kubernetes output isolated from other workloads, so `kubectl get` commands show only resources created by this chart.

## Part 3 – Explanation

Upgrading the image tag changes the Deployment manifest rendered by Helm.
Running `helm upgrade` applies the updated manifest and Kubernetes performs a rolling update (new pod is created with the new image while the old pod terminates).

In our case, the example tag `0.3.0` was not available in the registry (manifest not found), so we used a valid tag (`latest`) to complete the required image upgrade.
