# Kubernetes Secret Syncer Operator

A lightweight Kubernetes Operator that automatically synchronizes Secrets across all namespaces. 

Ideal for **Wildcard SSL Certificates**, API Keys, and global credentials that need to be available to every application in your cluster.

##  Features
- **Auto-Sync:** Instantly copies any secret labeled `syncer=true` to all other namespaces.
- **Auto-Update:** Updates all copies when the original secret changes (e.g., certificate renewal).
- **Namespace Watch:** Automatically populates new namespaces with synced secrets as soon as they are created.
- **Cleanup:** Deleting the source secret automatically removes the copies.

##  Installation

Add the Helm repository:

```bash
helm repo add my-syncer [https://ankurkumar11.github.io/secretsyncer-operator/](https://ankurkumar11.github.io/secretsyncer-operator/)
helm repo update

Install the Operator:

helm install secret-operator my-syncer/secret-syncer \
  --namespace operators \
  --create-namespace


Basic Usage
To sync any secret, simply add the label syncer=true to it.

kubectl create secret generic my-api-key --from-literal=key=super-secret
kubectl label secret my-api-key syncer=true
The secret my-api-key will immediately appear in all other namespaces.


Cert-Manager (Wildcard Certs)
This is the primary use case. To sync a generic Wildcard Certificate managed by cert-manager:

Add the secretTemplate block to your Certificate YAML:


apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: wildcard-cert
  namespace: default
spec:
  secretName: wildcard-tls
  commonName: "*.example.com"
  dnsNames:
  - "example.com"
  - "*.example.com"
  issuerRef:
    name: letsencrypt-prod-azure-dns
    kind: ClusterIssuer
  
  # --- THIS TRIGGERS THE SYNC ---
  secretTemplate:
    labels:
      syncer: "true"


When cert-manager renews the certificate, the Operator detects the change and updates the TLS secret in every namespace instantly.

🧹 Uninstallation

helm uninstall secret-operator -n operators