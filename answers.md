# answers.md – Helm Assignment Explanations

## Helm Lifecycle Explanation

Helm is a package manager for Kubernetes.
It works by rendering Kubernetes YAML templates (under `templates/`) using values from `values.yaml`, and then applying them to the Kubernetes cluster.

A typical Helm lifecycle includes:

1. **helm install**
   - Installs a chart into the cluster as a new release.
   - Creates Kubernetes resources such as Deployments, Services, ConfigMaps, and Secrets.

2. **helm upgrade**
   - Updates an existing release by re-rendering templates with new values.
   - Applies changes to the cluster.
   - Creates a new revision.

3. **helm history**
   - Displays all revisions of the release, including timestamps and statuses.

4. **helm rollback**
   - Restores a previous revision by re-deploying the old manifests.

5. **helm uninstall**
   - Removes all Kubernetes resources created by the release.

---

## Why `helm upgrade --install` is preferred

I used `helm upgrade --install` because it is idempotent:

- If the release does not exist, Helm installs it.
- If the release already exists, Helm upgrades it.

This is preferred in real DevOps environments because it works well in automation and CI/CD pipelines.
It allows the same command to be executed repeatedly without manual checks.

---

## Explanation of Chart Files

### Chart.yaml
`Chart.yaml` contains chart metadata such as:

- `name` – the chart name
- `version` – the chart version (changes when the chart is updated)
- `appVersion` – the application version (usually the container version)

---

### values.yaml
`values.yaml` contains the default configuration values for the chart.

It allows me to change:
- image repository and tag
- service port and type
- replica count
- enable/disable specific resources (Deployment, DaemonSet, Job, CronJob)
- ConfigMap and Secret content

This makes the chart reusable across environments.

---

## Explanation of Each Template / Resource

### Deployment (`deployment.yaml`)
The Deployment runs the main application as a long-running workload.

I used a Deployment because:
- it supports rolling updates
- it ensures the desired number of replicas is running
- it automatically recreates pods if they fail

The number of replicas is controlled by:

- `.Values.deployment.replicaCount`

---

### Service (`service.yaml`)
The Service exposes the application internally inside the cluster.

It uses selectors to forward traffic to the correct pods.

This allows stable networking even if pod IPs change.

---

### DaemonSet (`daemonset.yaml`)
A DaemonSet runs one pod on each node in the cluster.

This is useful for:
- monitoring agents
- log collectors
- node-level tools

In my chart it is disabled by default using:

- `.Values.daemonset.enabled`

---

### Job (`job.yaml`)
A Job is used for one-time execution tasks.

It creates pods that run until completion.

In my chart it is disabled by default using:

- `.Values.job.enabled`

---

### CronJob (`cronjob.yaml`)
A CronJob is used to run Jobs on a schedule.

The schedule is configured using:

- `.Values.cronjob.schedule`

In my chart it is disabled by default using:

- `.Values.cronjob.enabled`

---

### ConfigMap (`configmap.yaml`)
A ConfigMap stores non-sensitive configuration data.

In my chart, the ConfigMap values are defined in:

- `.Values.configmap.data`

Example usage:
- message text
- configuration values
- feature flags

---

### Secret (`secret.yaml`)
A Secret stores sensitive data such as API keys or tokens.

In my chart, the Secret values are defined in:

- `.Values.secret.stringData`

This is preferred for sensitive information because Kubernetes treats Secrets differently than ConfigMaps.

---

## Part 3 – Image Version Upgrade Explanation

I upgraded the image tag by changing it in `values.yaml`.

Running `helm upgrade` caused Kubernetes to perform a rolling update:
- a new pod was created with the new image
- the old pod was terminated after the new pod became ready

The assignment suggested tag `0.3.0`, but in my environment it was not available (manifest not found),
therefore I used a valid available tag (`latest`) to complete the required upgrade process.

---

## Part 4 – Helm History & Rollback Explanation

`helm history` displays all revisions of a Helm release.
Each `helm upgrade` creates a new revision.

`helm rollback` allows restoring a previous revision.
This is useful when an upgrade introduces a bug or a bad configuration.

---

## Part 5 – ConfigMap and Secret Mounting

I injected the ConfigMap and Secret into the Deployment and DaemonSet using two methods:

### 1. envFrom
- `configMapRef` loads ConfigMap keys as environment variables
- `secretRef` loads Secret keys as environment variables

### 2. volumes + volumeMounts
- ConfigMap is mounted under `/config`
- Secret is mounted under `/secrets` (read-only)

This allows changing configuration without rebuilding the container image.

Note:
The container image `hashicorp/http-echo` is minimal and does not include common Linux utilities such as `ls`,
therefore `kubectl exec ... ls` fails.
Verification was done by inspecting the rendered Helm manifest instead.
