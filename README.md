# 🚢 Deploying Harbor Container Registry on Kubernetes

This guide walks you through installing **Harbor** (a cloud-native container registry) on a Kubernetes cluster using Helm. The steps match the following article:  
🔗 [Harbor Container Registry – DevGenius](https://blog.devgenius.io/harbor-container-registry-f0d5ef7d899a)

---

## 📋 Prerequisites

- A running Kubernetes cluster (tested with 1.24+)
- Helm installed (`v3+`)
- A domain name (e.g., `harbor.example.com`) and a TLS certificate (or use self-signed cert)
- An Ingress Controller installed (like NGINX)

---

## 1️⃣ Add Harbor Helm Repo

```bash
helm repo add harbor https://helm.goharbor.io
helm repo update
```

---

## 2️⃣ Create a Namespace for Harbor

```bash
kubectl create namespace harbor
```

---

## 3️⃣ Generate TLS Certificates (Optional: for production use real certs)

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -out tls.crt \
  -keyout tls.key \
  -subj "/CN=harbor.local"

kubectl create secret tls harbor-tls \
  --cert=tls.crt \
  --key=tls.key \
  -n harbor
```

> Replace `harbor.local` with your actual domain name.

---

## 4️⃣ Create a Custom `values.yaml` File

Create a file named `values.yaml`:

```yaml
expose:
  type: ingress
  tls:
    enabled: true
    secretName: harbor-tls
    commonName: harbor.local

externalURL: https://harbor.local

harborAdminPassword: "Harbor12345"

persistence:
  persistentVolumeClaim:
    registry:
      storageClass: "standard"
      accessMode: ReadWriteOnce
      size: 5Gi
    jobservice:
      storageClass: "standard"
      accessMode: ReadWriteOnce
      size: 1Gi
    database:
      storageClass: "standard"
      accessMode: ReadWriteOnce
      size: 1Gi
    redis:
      storageClass: "standard"
      accessMode: ReadWriteOnce
      size: 1Gi
    trivy:
      storageClass: "standard"
      accessMode: ReadWriteOnce
      size: 5Gi
```

> Adjust `storageClass` and sizes based on your environment.  
> Replace `harbor.local` with your actual domain.

---

## 5️⃣ Install Harbor with Helm

```bash
helm install harbor harbor/harbor \
  -n harbor \
  -f values.yaml
```

---

## 6️⃣ Verify Deployment

Check all pods are running:

```bash
kubectl get pods -n harbor
```

Check the services:

```bash
kubectl get svc -n harbor
```

---

## 7️⃣ Access Harbor UI

- Update your local `/etc/hosts` if you're using self-signed certs:

```bash
sudo echo "YOUR_HARBOR_IP harbor.local" >> /etc/hosts
```

- Visit `https://harbor.local` in your browser.
- Login with:
  - **Username:** `admin`
  - **Password:** `Harbor12345`

---

✅ **Done!** Your Harbor registry is now running on Kubernetes.
