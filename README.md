# uptime-kuma
uptime-kuma

To run Uptime Kuma with an Nginx reverse proxy and Let's Encrypt TLS/HTTPS on Kubernetes, you can use Kubernetes manifests and Helm charts. Here's a step-by-step guide:
Step 1: Create Kubernetes Manifests
1.1 Uptime Kuma Deployment and Service

Create a file named uptime-kuma.yaml:

yaml

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

1.2 Persistent Volume Claim

Create a file named uptime-kuma-pvc.yaml:

yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: uptime-kuma-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

1.3 Nginx Ingress Controller

It's recommended to use a Helm chart to install the Nginx Ingress Controller and Cert-Manager.

bash

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add jetstack https://charts.jetstack.io
helm repo update

# Install Nginx Ingress Controller
helm install nginx-ingress ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace

# Install Cert-Manager
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.8.0 --set installCRDs=true

Step 2: Create Cert-Manager Issuer

Create a file named letsencrypt-issuer.yaml:

yaml

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-production
spec:
  acme:
    email: your-email@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-production
    solvers:
    - http01:
        ingress:
          class: nginx

Replace your-email@example.com with your email address.
Step 3: Create Ingress for Uptime Kuma

Create a file named uptime-kuma-ingress.yaml:

yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: uptime-kuma
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-production"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: your-domain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: uptime-kuma
            port:
              number: 3001
  tls:
  - hosts:
    - your-domain.com
    secretName: uptime-kuma-tls

Replace your-domain.com with your actual domain name.
Step 4: Deploy the Resources

Apply the manifests in the following order:

bash

kubectl apply -f uptime-kuma-pvc.yaml
kubectl apply -f uptime-kuma.yaml
kubectl apply -f letsencrypt-issuer.yaml
kubectl apply -f uptime-kuma-ingress.yaml

Summary

This setup involves:

    Deploying Uptime Kuma on Kubernetes with a Persistent Volume Claim for data storage.
    Installing Nginx Ingress Controller and Cert-Manager using Helm.
    Creating a ClusterIssuer for Let's Encrypt.
    Configuring an Ingress resource for Uptime Kuma to handle HTTPS traffic with TLS certificates managed by Cert-Manager.

This will ensure Uptime Kuma is accessible over HTTPS with automatic certificate renewal.
