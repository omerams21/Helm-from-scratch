# helm-from-scratch
Assignment 6

## Part 1 Validation

I validated the Helm chart templates using:

```bash
helm lint charts/myapp
helm template testrel charts/myapp


## Part 2 – Deploy the Chart

I deployed the chart using `helm upgrade --install` into a dedicated namespace to keep the output clean:

```bash
helm upgrade --install myapp ./charts/myapp -n myapp --create-namespace |& tee outputs/helm-install.txt


## Part 3 – Image Version Upgrade

The assignment suggested upgrading the image tag (example: `0.3.0`).  
In our environment, the tag `0.3.0` was not available (manifest not found), so we upgraded from `0.2.3` to a valid tag: `latest`.

Upgrade command:

```bash
helm upgrade --install myapp ./charts/myapp -n myapp |& tee outputs/helm-upgrade.txt


## Part 4 – Helm History & Rollback

We checked the Helm release history:

```bash
helm history myapp -n myapp |& tee outputs/helm-history.txt


## Part 5 – ConfigMap and Secret Usage

The chart stores application configuration in a ConfigMap and sensitive data in a Secret:

- ConfigMap: `myapp-myapp-cm`
- Secret: `myapp-myapp-secret`

Both are injected into the Deployment (and also implemented in the DaemonSet template) using:

- `envFrom` (loads all keys as environment variables)
- `volumes` + `volumeMounts` (mounts them as files)

Mounted paths:

- ConfigMap mounted to `/config`
- Secret mounted to `/secrets` (read-only)

After upgrading the release, we verified that the Deployment manifest contains the required configuration injection:

```bash
helm get manifest myapp -n myapp | grep -nE "envFrom|configMapRef|secretRef|volumeMounts|mountPath|volumes|secretName"

