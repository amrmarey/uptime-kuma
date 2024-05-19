# Uptime Kuma

## Description
To run Uptime Kuma with an Nginx reverse proxy and Let's Encrypt TLS/HTTPS on Kubernetes, you can use Kubernetes manifests and Helm charts. Here's a step-by-step guide:

## Table of Contents
- [Step 1: Create Kubernetes Manifests](#step-1-create-kubernetes-manifests)
  - [1.1 Uptime Kuma Deployment and Service](#11-uptime-kuma-deployment-and-service)
  - [1.2 Persistent Volume Claim](#12-persistent-volume-claim)
  - [1.3 Nginx Ingress Controller](#13-nginx-ingress-controller)
- [Step 2: Create Cert-Manager Issuer](#step-2-create-cert-manager-issuer)
- [Step 3: Create Ingress for Uptime Kuma](#step-3-create-ingress-for-uptime-kuma)
- [Step 4: Deploy the Resources](#step-4-deploy-the-resources)
- [Summary](#summary)

## Step 1: Create Kubernetes Manifests

### 1.1 Uptime Kuma Deployment and Service

Create a file named `uptime-kuma.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: uptime-kuma
  labels:
    app: uptime-kuma
spec:
  replicas: 1
  selector:
    matchLabels:
      app: uptime-kuma
  template:
    metadata:
      labels:
        app: uptime-kuma
    spec:
      containers:
      - name: uptime-kuma
        image: louislam/uptime-kuma:1
        ports:
        - containerPort: 3001
        volumeMounts:
        - name: uptime-kuma-data
          mountPath: /app/data
      volumes:
      - name: uptime-kuma-data
        persistentVolumeClaim:
          claimName: uptime-kuma-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: uptime-kuma
  labels:
    app: uptime-kuma
spec:
  type: ClusterIP
  ports:
  - port: 3001
    targetPort: 3001
  selector:
    app: uptime-kuma
