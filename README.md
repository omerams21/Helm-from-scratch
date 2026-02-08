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

