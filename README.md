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

