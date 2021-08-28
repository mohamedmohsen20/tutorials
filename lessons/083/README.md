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

- Port forward Prometheus
```bash
kubectl port-forward svc/prometheus-operated 9090 -n monitoring
```

- Go to Prometheus targets `http://localhost:9090`

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

- Deploy Grafana using YAML
  - `grafana/0-secret.yaml`
  - `grafana/1-deployment.yaml`
  - `grafana/2-service.yaml`

```bash
kubectl apply -f grafana
```

- Get services in monitoring namespaces
```bash
kubectl get svc -n monitoring
```

- Create `HTTP` ingress
  - `example-3/grafana.yaml`

```bash
kubectl apply -f example-3
```

- Get ingresses
```bash
kubectl get ing -n monitoring
```

- Create `CNAME` record for `grafana.devopsbyexample.io`

- Access `http://grafana.devopsbyexample.io`

- Install wireshark
```bash
brew install wireshark
```

- Get public ip address of Grafana
```bash
dig +short grafana.devopsbyexample.io
```

- Get network interfaces
```bash
ifconfig
```

- Get POST TCP packets
```bash
sudo tshark -i en1 -x -f "host 52.207.88.124 and port 80 and tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x504f5354" > post.pcap
```

```bash
cat post.pcap
```

- Add TLS section to grafana ingress
  - `example-3/grafana.yaml`

```bash
kubectl apply -f example-3
```

- Get certificates in monitoring namespace
```bash
kubectl get certificates -n monitoring
```

- Go to `https://grafana.devopsbyexample.io` to check TLS certs

- Add CA to the keychain

- Login to Grafana

- Listen on port 443


```bash
sudo tshark -i en1 -x -f "host 52.207.88.124 and port 443" > login-https.pcap
```

- Read the TCP packets
```bash
cat login-https.pcap
```

## Secure NGINX ingress with Let's Encrypt & ACME (Example 3)
- Create `letsencrypt-staging` Issuer in `monitoring` namespace
  - `example-4/letsencrypt-staging-issuer.yaml`

- Create `letsencrypt-staging` Issuer in `monitoring` namespace
  - `example-4/letsencrypt-prod-issuer.yaml`

```bash
kubectl apply -f example-4
```

- Get issuers in `monitoring` namespace
```bash
kubectl get issuers -n monitoring
```

- If it's not ready describe to get error message
```bash
kubectl describe issuer letsencrypt-prod -n monitoring
```

- Create Prometheus ingress with `letsencrypt-prod` issuer
  - `example-4/2-prometheus-ing.yaml`

- Watch CRDs
```bash
kubectl get Certificates -n monitoring --watch
kubectl get CertificateRequests -n monitoring --watch
kubectl get Orders -n monitoring --watch
kubectl get Challenges -n monitoring --watch
```

- Apply Prometheus ingress
```bash
kubectl apply -f example-4/2-prometheus-ing.yaml
```

- Describe certificate first
```bash
kubectl get certificate -n monitoring
kubectl describe certificate prometheus-devopsbyexample-io-key-pair -n monitoring
```

- Describe certificate request
```bash
kubectl get certificaterequests -n monitoring
kubectl describe certificaterequests prometheus-devopsbyexample-io-key-pair-<id> -n monitoring
```

- Describe order
```bash
kubectl get orders -n monitoring
kubectl describe orders prometheus-devopsbyexample-io-key-pair-<id> -n monitoring
```

- Describe challenge
```bash
kubectl get challenges -n monitoring
kubectl describe challenges prometheus-devopsbyexample-io-key-pair-<id> -n monitoring
```

- Make sure that all pods are up in monitoring namespace including `cm-acme-http-solver-<id>`
```bash
kubectl get pods -n monitoring
```

- To complete we need to create CNAME record for prometheus
```bash
kubectl get ing -n monitoring
```



## Delegate a Subdomain to Route53
> [Creating a subdomain that uses Amazon Route 53 as the DNS service without migrating the parent domain](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/CreatingNewSubdomain.html)

## Monitor Cert Manager with Prometheus and Grafana

- Create `monitoring.devopsbyexample.io` Route53 hosted zone

- Get Name Servers from `Hosted zone details`

- Add NS records for the subdomain `monitoring`

- Create A record to test route53 zone `test.monitoring.devopsbyexample.io` -> 10.10.10.10

- Wait up to 1 minute and test with dig

```bash
dig +short test.monitoring.devopsbyexample.io
```

## Create IAM Role for Kubernetes Service Account

- Create OpenID Connect provider, use `sts.amazonaws.com` for Audience

- Create IAM policy with Route53 access
  - `CertManagerRoute53Access`
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "route53:GetChange",
      "Resource": "arn:aws:route53:::change/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets",
        "route53:ListResourceRecordSets"
      ],
      "Resource": "arn:aws:route53:::hostedzone/Z00637903L4LNI5ZNF0EQ"
    }
  ]
}
```

- Create IAM role for `cert-manager-acme` cert-manager

- Get service account name for cert-manager
```bash
kubectl get sa -n cert-manager
```

- Update trust for IAM role to allow only `cert-083-cert-manager` service account to assume it
```
aud -> sub
sts.amazonaws.com -> system:serviceaccount:cert-manager:cert-083-cert-manager
```

- Update service account manually, add following line to annotations
```bash
kubectl edit sa -n cert-manager
```
```yaml
eks.amazonaws.com/role-arn: arn:aws:iam::424432388155:role/cert-manager-acme
```

- Modify the cert-manager Deployment with the correct file system permissions, so the ServiceAccount token can be read.
```bash
kubectl get deployment -n cert-manager
kubectl edit deployment cert-083-cert-manager -n cert-manager
```
```yaml
spec:
  template:
    spec:
      securityContext:
        fsGroup: 1001
```
```yaml
- --issuer-ambient-credentials=true
```

- This change can be inclused to Helm Chart (include to `cert-manager-values.yaml` before installing or upgrade Helm release)
```yaml
serviceAccount:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::424432388155:role/cert-manager-acme
securityContext:
  enabled: true
  fsGroup: 1001
extraArgs: 
- --issuer-ambient-credentials=true
```

## Resolve DNS-01 challenge with cert-manager (Example 5)

- Create Issuer to use dns-01 challenge
  - `example-5/0-letsencrypt-staging-dns01-issuer.yaml`

```bash
kubectl apply -f example-5/0-letsencrypt-staging-dns01-issuer.yaml
```

- List Issuers in monitoring namespace
```bash
kubectl get issuers -n monitoring
```

- Describe `letsencrypt-dns01-staging` issuer
```bash
kubectl describe issuer letsencrypt-dns01-staging -n monitoring
```

https://cert-manager.io/docs/configuration/acme/dns01/route53/

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


## Notes
http://seclists.org/tcpdump/2004/q4/95
apture HTTP GET requests. This looks for the bytes 'G', 'E', 'T', and ' ' (hex values 47, 45, 54, and 20) just after the TCP header. "tcp[12:1] & 0xf0) >> 2" figures out the TCP header length. From Jefferson Ogata via the tcpdump-workers mailing list.

- use case for grafana since it's going to be used internally only - protect password with https

- can i show how to use wireshark to snif passwords???? man in the middle

- use case for private dns zones (no need VPN)

- create own hosted zone in route53 (no need VPN)

Nginx Ingress Class: https://github.com/kubernetes/ingress-nginx/blob/1510c06045ece4e199ebec85e7ec90cf15e19747/docs/index.md

--watch-ingress-without-class=true