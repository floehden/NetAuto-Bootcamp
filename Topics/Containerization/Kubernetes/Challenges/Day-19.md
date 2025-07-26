# **Day 19: ArgoCD: Declarative GitOps CD**

## **Concepts:**
* Introduction to ArgoCD: A declarative GitOps continuous delivery tool.
* Core components: API Server, Application Controller, Repo Server, Dex, Redis.
* Applications as custom resources (`Application` CRD).
* Synchronization strategies (manual, auto-sync), health checks, pruning, self-healing.

## **Examples (Code):**
```bash
# Install ArgoCD on Kind
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
echo "Waiting for ArgoCD pods to be ready..."
kubectl wait --namespace argocd \
    --for=condition=ready pod \
    --selector=app.kubernetes.io/name=argocd-server \
    --timeout=90s

# Get the ArgoCD UI password
ARGOCD_SERVER_POD=$(kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o jsonpath='{.items[0].metadata.name}')
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

# Expose ArgoCD UI via port-forwarding (for Kind local access)
echo "Port-forwarding ArgoCD server to localhost:8081. Open http://localhost:8081"
kubectl port-forward svc/argocd-server -n argocd 8081:443 --address 0.0.0.0 & # Run in background
# Access ArgoCD UI at https://localhost:8081 (username: admin, password: obtained above). Accept self-signed cert.

# Create a Git repository (e.g., on GitHub) with a simple Nginx Deployment and Service
# Example repo: https://github.com/<your-username>/argocd-example-app
# Create the following files in the root of your repo:
```

```yaml
# gitops-nginx-app/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
name: gitops-nginx-deployment
labels:
app: gitops-nginx
spec:
replicas: 2
selector:
matchLabels:
    app: gitops-nginx
template:
metadata:
    labels:
    app: gitops-nginx
spec:
    containers:
    - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

```yaml
# gitops-nginx-app/service.yaml
apiVersion: v1
kind: Service
metadata:
    name: gitops-nginx-service
spec:
    selector:
    app: gitops-nginx
    ports:
    - protocol: TCP
        port: 80
        targetPort: 80
    type: ClusterIP
EOF
```

```bash
# Commit and push these files to your Git repository

# Create your first ArgoCD Application (argocd-app-nginx.yaml)
cat <<EOF > argocd-app-nginx.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
    name: my-first-gitops-app
    namespace: argocd
spec:
    project: default
    source:
    repoURL: https://github.com/<your-username>/argocd-example-app.git # Replace with your repo
    targetRevision: HEAD
    path: . # Path within the repo where K8s manifests are
    destination:
    server: https://kubernetes.default.svc # Refers to the in-cluster API server
    namespace: default # Namespace to deploy into
    syncPolicy:
    automated:
        prune: true
        selfHeal: true
EOF
```

```bash
kubectl apply -f argocd-app-nginx.yaml -n argocd
# Observe in ArgoCD UI: Application will appear, synchronize, and resources will be created.
kubectl get deployment gitops-nginx-deployment
kubectl get service gitops-nginx-service

# Make a change in Git (e.g., change replicas to 3 in deployment.yaml, commit, push)
# Observe ArgoCD detecting the OutOfSync state and automatically syncing
```

## **Challenge:** 
Create a Git repository with a simple `httpd` Deployment and Service YAML. Use ArgoCD to deploy this application to your Kind cluster. Make a change to the image tag in Git and observe ArgoCD automatically syncing the change. Access the ArgoCD UI via port-forwarding and explore the application's details.

