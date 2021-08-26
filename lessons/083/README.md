# Cert Manager Kubernetes Tutorial (Let's Encrypt & Nginx Ingress & ACME | YAML & HELM | 3 Examples)

[YouTube Tutorial]()

## Prerequisites

- [Kubernetes](https://kubernetes.io/)
- [Helm 3](https://helm.sh/)

## Install Cert Manager Helm & YAML

- Review default helm [values](https://github.com/jetstack/cert-manager/blob/master/deploy/charts/cert-manager/values.yaml)

- Add the Jetstack Helm repository

```bash
helm repo add jetstack \
    https://charts.jetstack.io
```

- Update your local Helm chart repository cache

```bash
helm repo update
```

- Create `values.yaml` file to override default variables

- Search for cert-manager

```bash
helm search repo cert-manager
```

- Generate YAML files

```bash
helm template lesson-083 jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.5.3 \
  --values values.yaml \
  --output-dir helm-generated-yaml
```

## Clean Up
- Remove Helm repo
```bash
helm repo remove jetstack
```
