# Cert Manager Kubernetes Tutorial (Let's Encrypt & Nginx Ingress & ACME | YAML & HELM | 3 Examples)

[YouTube Tutorial]()

## Prerequisites

- [Kubernetes](https://kubernetes.io/)
- [Helm 3](https://helm.sh/)

## Intro
- Clean Documentation
- What is certificate authority?
- How certificates singed using private key (Flow csr -> cer)
- What is PKI(Public key infrastructure)?
- Cert manager components
- Difference between Issuer vs ClusterIssuer (Namespaced/K8s secrets location)
- 1,2 use cases for Internal services such as Grafana, example 3 for publicly facing
- How DNS works and how to resolve private DNS such as route53

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
helm install cert-083 jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.5.3 \
  --values cert-manager-values.yaml
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

## SelfSigned (Example 1)
## CA (Example 2)
- use case for grafana since it's going to be used internally only - protect password with https
- can i show how to use wireshark to snif passwords???? man in the middle
- use case for private dns zones (no need VPN)
- create own hosted zone in route53 (no need VPN)

## Deploy Nginx Ingress Controller
```bash
helm repo add ingress-nginx \
  https://kubernetes.github.io/ingress-nginx
```
```bash
helm repo update
```
```bash
helm install ing-083 ingress-nginx/ingress-nginx \
  --namespace ingress \
  --version 4.0.1 \
  --values nginx-ingress-values.yaml \
  --create-namespace
```

## Deploy Grafana on Kubernetes
```bash
kubectl apply -f grafana
```

## ACME (Example 3)

## Install Grafana on Kubernetes
- id: 11001
- `grafana-dashboard.json`
## Monitor Cert Manager with Prometheus and Grafana
- Port forward Prometheus to localhost
```bash
kubectl port-forward svc/prometheus-operated 9090 -n monitoring
```

## Clean Up
- Remove Helm repo
```bash
helm repo remove jetstack
helm repo remove ingress-nginx
```
- Delete Helm release
```bash
helm delete lesson-083 -n cert-manager
```
- List all cert-manager resources
```bash
kubectl get Issuers,ClusterIssuers,Certificates,CertificateRequests,Orders,Challenges -A
```
