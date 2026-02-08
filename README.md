# Helm From Scratch - Assignment

This repository contains a Helm chart built from scratch as part of a Kubernetes/Helm assignment.

The chart deploys a lightweight HTTP server using the image:

- `hashicorp/http-echo`

It demonstrates the usage of common Kubernetes resources using Helm templating.

---

## Repository Structure

```
helm-from-scratch/
├── charts/
│   └── myapp/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── daemonset.yaml
│           ├── cronjob.yaml
│           ├── job.yaml
│           ├── configmap.yaml
│           └── secret.yaml
├── README.md
├── answers.md
├── outputs/
│   ├── helm-install.txt
│   ├── helm-upgrade.txt
│   ├── helm-history.txt
│   └── helm-rollback.txt
```

---

## Purpose of Each Resource

### Chart.yaml
Stores chart metadata such as chart name, chart version, and app version.

### values.yaml
Stores default configuration values (image, service settings, replica count, and enable/disable flags).

### Deployment
Runs the main application as a long-running workload and supports rolling updates.

### Service
Exposes the application inside the cluster using a ClusterIP service.

### DaemonSet
Template exists to demonstrate running a pod on every node.
Disabled by default and can be enabled using `daemonset.enabled=true`.

### Job
Template exists to demonstrate a one-time workload execution.
Disabled by default and can be enabled using `job.enabled=true`.

### CronJob
Template exists to demonstrate scheduled job execution.
Disabled by default and can be enabled using `cronjob.enabled=true`.

### ConfigMap
Stores non-sensitive configuration values such as message text.

### Secret
Stores sensitive configuration such as API tokens.

---

## Part 1 – Create Helm Chart

I created a Helm chart under `charts/myapp` and implemented the required templates.

Validation commands:

```bash
helm lint charts/myapp
helm template testrel charts/myapp
```

---

## Part 2 – Deploy the Chart

I deployed the chart into a dedicated namespace to keep the output clean and isolated:

```bash
helm upgrade --install myapp ./charts/myapp -n myapp --create-namespace |& tee outputs/helm-install.txt
kubectl get all -n myapp |& tee -a outputs/helm-install.txt
kubectl get configmap -n myapp |& tee -a outputs/helm-install.txt
kubectl get secret -n myapp |& tee -a outputs/helm-install.txt
```

Output saved to:

- `outputs/helm-install.txt`

---

## Part 3 – Image Version Upgrade

The assignment suggested upgrading the image tag (example: `0.3.0`).
In my environment, the tag `0.3.0` was not available in the registry (manifest not found).

Therefore, I upgraded the image tag from `0.2.3` to a valid available tag: `latest`.

Upgrade command:

```bash
helm upgrade --install myapp ./charts/myapp -n myapp |& tee outputs/helm-upgrade.txt
```

Verification commands:

```bash
kubectl rollout status deployment/myapp-myapp -n myapp --timeout=120s |& tee -a outputs/helm-upgrade.txt
kubectl get deployment myapp-myapp -n myapp -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}' |& tee -a outputs/helm-upgrade.txt
kubectl get pods -n myapp |& tee -a outputs/helm-upgrade.txt
```

Output saved to:

- `outputs/helm-upgrade.txt`

---

## Part 4 – Helm History & Rollback

I checked the release history:

```bash
helm history myapp -n myapp |& tee outputs/helm-history.txt
```

Then I rolled back the release to revision 1:

```bash
helm rollback myapp 1 -n myapp |& tee outputs/helm-rollback.txt
```

Verification after rollback:

```bash
kubectl rollout status deployment/myapp-myapp -n myapp --timeout=120s |& tee -a outputs/helm-rollback.txt
kubectl get deployment myapp-myapp -n myapp -o jsonpath='{.spec.template.spec.containers[0].image}{"\n"}' |& tee -a outputs/helm-rollback.txt
kubectl get pods -n myapp |& tee -a outputs/helm-rollback.txt
```

Outputs saved to:

- `outputs/helm-history.txt`
- `outputs/helm-rollback.txt`

---

## Part 5 – ConfigMap and Secret Usage

I stored non-sensitive configuration in a ConfigMap and sensitive data in a Secret.

Both resources are injected into the Deployment and DaemonSet using:

- `envFrom` (loads all keys as environment variables)
- `volumes + volumeMounts` (mounts them as files)

Mounted paths:

- ConfigMap mounted to `/config`
- Secret mounted to `/secrets` (read-only)

Verification (by inspecting Helm rendered manifest):

```bash
helm get manifest myapp -n myapp | grep -nE "envFrom|configMapRef|secretRef|volumeMounts|mountPath|volumes|secretName"
```

Note: `hashicorp/http-echo` is a minimal container image and does not include tools like `ls`,
therefore `kubectl exec ... ls` is not supported.

---

## Outputs Folder

All required command outputs are saved under the `outputs/` directory:

- `outputs/helm-install.txt`
- `outputs/helm-upgrade.txt`
- `outputs/helm-history.txt`
- `outputs/helm-rollback.txt`
