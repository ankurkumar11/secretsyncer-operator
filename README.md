
# Kubernetes Secret Syncer Operator

A lightweight, Python-based Kubernetes Operator that automatically synchronizes Secrets across all namespaces in your cluster.

Ideal for distributing **Wildcard SSL Certificates**, **API Keys**, and **Global Registry Credentials** to every application automatically.

---

## 🏗 Architecture

The operator watches for specific events and propagates changes instantly:

```mermaid
graph LR
    A[Source Secret] -- labeled syncer=true --> B(Operator)
    B -- copy --> C[Namespace A]
    B -- copy --> D[Namespace B]
    B -- copy --> E[New Namespace]
````

> **Note:** If you delete the source secret, the operator automatically cleans up the copies to prevent orphaned credentials.

-----

## ✨ Features

  * 🚀 **Auto-Sync:** Instantly copies any secret labeled `syncer=true` to all existing namespaces.
  * 🔄 **Auto-Update:** Watcher detects changes (e.g., Certificate Renewal) and updates all copies immediately.
  * 👁️ **Namespace Watch:** Automatically populates **newly created namespaces** with synced secrets.
  * 🧹 **Smart Cleanup:** Deleting the source secret automatically removes the copies.

-----

## 📦 Installation

We recommend installing via **Helm** for the best experience.

### 1\. Add the Repository

```bash
helm repo add my-syncer [https://ankurkumar11.github.io/secretsyncer-operator/](https://ankurkumar11.github.io/secretsyncer-operator/)
helm repo update
```

### 2\. Install the Operator

```bash
helm install secret-operator my-syncer/secret-syncer \
  --namespace operators \
  --create-namespace
```

-----

## 🛠 Usage

### Scenario A: Basic Usage (Manual)

To sync any secret, simply add the label `syncer=true` to it.

```bash
# 1. Create a secret
kubectl create secret generic my-api-key --from-literal=key=super-secret

# 2. Label it to trigger the sync
kubectl label secret my-api-key syncer=true
```

> **Result:** The secret `my-api-key` will immediately be created in all other namespaces.

<br>

### Scenario B: Cert-Manager (Wildcard Certificates)

This is the primary use case. It allows you to manage a single Wildcard Certificate in `default` and have it available for Ingress resources in every namespace.

Add the `secretTemplate` block to your Certificate YAML:

```yaml
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
  
  # ------------------------------
  # THIS TRIGGERS THE SYNC OPERATOR
  # ------------------------------
  secretTemplate:
    labels:
      syncer: "true"
```

> **How it works:** When `cert-manager` renews the certificate (every 60-90 days), it updates the `wildcard-tls` secret. The Operator detects this update and instantly patches the new certificate data into every namespace.

-----

## 🧹 Uninstallation

To remove the operator and stop syncing:

```bash
helm uninstall secret-operator -n operators
```

*(Note: Secrets already created by the operator will remain until manually deleted, or until the source secret is deleted before uninstallation.)*

```
```
