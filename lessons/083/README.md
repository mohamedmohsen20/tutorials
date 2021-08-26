# Cert Manager Kubernetes Tutorial (Let's Encrypt & Nginx Ingress & ACME | YAML & HELM | 3 Examples)

[YouTube Tutorial]()

## Prerequisites

- [Kubernetes](https://kubernetes.io/)
- [Helm 3](https://helm.sh/)

## Intro
- What is certificate authority?
- What is PKI(Public key infrastructure)?
- Cert manager components

## Deploy Prometheus on Kubernetes
- Install Prometheus
```bash
kubectl apply -f prometheus/0-crd
kubectl apply -f prometheus/1-prometheus-operator
kubectl apply -f prometheus/2-prometheus
kubectl get pods -n monitoring
```

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
- Create `cert-manager` namespace
```bash
kubectl apply -f cert-manager-ns.yaml
```

- Install cert-manager with Helm
```bash
helm install lesson-083 jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.5.3 \
  --values values.yaml
```

- Get Helm releases
```bash
helm list -n cert-manager
```

- Get pods
```bash
kubectl get pods -n cert-manager
```
- [cert-manager](https://cert-manager.io/docs/concepts/)
- [cainjector](https://cert-manager.io/docs/concepts/ca-injector/)
- [webhook](https://cert-manager.io/docs/concepts/webhook/)

## Install Grafana on Kubernetes
## Monitor Cert Manager with Prometheus and Grafana

## Clean Up
- Remove Helm repo
```bash
helm repo remove jetstack
```
- Delete Helm release
```bash
helm delete lesson-083 -n cert-manager
```
