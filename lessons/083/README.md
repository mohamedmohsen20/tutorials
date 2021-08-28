# Cert Manager Kubernetes Tutorial (Let's Encrypt & Nginx Ingress & ACME | YAML & HELM | 5 Examples)

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
```bash
kubectl apply -f prometheus/0-crd
kubectl apply -f prometheus/1-prometheus-operator
kubectl apply -f prometheus/2-prometheus
kubectl get pods -n monitoring
```

## Deploy Cert Manager Helm & YAML

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
helm template cert-083 jetstack/cert-manager \
  --namespace cert-manager \
  --version v1.5.3 \
  --values cert-manager-values.yaml \
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

## Generate Self Signed Certificate (Example 1)
- Create SelfSigned ClusterIssuer `example-1/0-self-signed-Issuer.yaml`

- Generate Self Signed Certificate `example-1/1-ca-certificate.yaml`
```bash
kubectl apply -f example-1
```

- Get certificate
```bash
kubectl get certificate -n cert-manager
```

- Get secrets
```bash
kubectl get secrets -n cert-manager
```

- Get x509 certificate
```bash
kubectl get secrets devopsbyexample-io-key-pair -o yaml -n cert-manager
```

- Decode base64 certificte
```bash
echo "base64" | base64 -d -o ca.crt
```

- Decode CA certificate
```bash
openssl x509 -in ca.crt -text -noout
```

## Generate TLS Certificate using CA (Example 2)
- Create CA ClusterIssuer `example-2/0-ca-issuer.yaml`

- Create `staging` namespace `example-2/1-staging-ns.yaml`

- Create certificate for blog.devopsbyexample.io `example-2/example-2/2-blog-certificate.yaml`
```bash
kubectl apply -f example-2
```

- Get certificate
```bash
kubectl get certificate -n staging
```

- Get secrets
```bash
kubectl get secrets -n staging
```

- Get x509 certificate
```bash
kubectl get secrets blog-devopsbyexample-io-key-pair -o yaml -n staging
```

- Decode base64 certificte
```bash
echo "base64" | base64 -d -o blog-devopsbyexample-io.crt
```

- Decode CA certificate
```bash
openssl x509 -in blog-devopsbyexample-io.crt -text -noout
```

## Deploy Nginx Ingress Controller
- Add ingress-nginx Helm repo
```bash
helm repo add ingress-nginx \
  https://kubernetes.github.io/ingress-nginx
```

- Update Helm repos
```bash
helm repo update
```

- Search for Nginx Helm Chart
```bash
helm search repo ingress-nginx
```

- Install Nginx Ingress
```bash
helm install ing-083 ingress-nginx/ingress-nginx \
  --namespace ingress \
  --version 4.0.1 \
  --values nginx-ingress-values.yaml \
  --create-namespace
```

## Deploy Grafana on Kubernetes

- use case for grafana since it's going to be used internally only - protect password with https
- can i show how to use wireshark to snif passwords???? man in the middle
- use case for private dns zones (no need VPN)
- create own hosted zone in route53 (no need VPN)
```bash
kubectl apply -f grafana
```

- Install wireshark
```bash
brew install wireshark
```
```bash
sudo tshark -i en1 -f "dst net 3.208.164.186"
sudo tshark -i en1 -f "tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)"
sudo tshark -i en1 -x -f "tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)" > test.pcap
sudo tshark -i en1 -f "host 3.208.164.186 and port 80"
sudo tshark -i en1 -f "host 3.208.164.186 and port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420"
sudo tshark -i en1 -f "host 3.208.164.186 and port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354"
sudo tshark -i en1 -x -f "host 3.208.164.186 and port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354" > login.pcap
sudo tshark -i en1 -x -f "host 3.208.164.186 and port 443 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354" > login-https.pcap
sudo tshark -i en1 -x -f "host 3.208.164.186 and port 443"

http://seclists.org/tcpdump/2004/q4/95
apture HTTP GET requests. This looks for the bytes 'G', 'E', 'T', and ' ' (hex values 47, 45, 54, and 20) just after the TCP header. "tcp[12:1] & 0xf0) >> 2" figures out the TCP header length. From Jefferson Ogata via the tcpdump-workers mailing list.

wireshark Capture filters
tcpdump -A -r stackoverflow.cap > stackoverflow.txt
```

kubectl get secrets devopsbyexample-io-key-pair -o yaml -n cert-manager


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
- Remove wireshark
```bash
brew remove wireshark
```
- Remove `devopsbyexample.io` CA
