# **Day 22: FluxCD: A Kubernetes-Native GitOps Tool**

## **Concepts:**
* Introduction to FluxCD: A CNCF graduated project, Kubernetes-native GitOps.
* Flux Toolkit components: Source Controller (fetches from Git), Kustomize Controller (applies Kustomize), Helm Controller (applies Helm releases), Notification Controller.
* Flux's pull-based reconciliation (cluster-side agent pulls changes).
* Git source management (`GitRepository` CRD).

## **Examples (Code):**
```bash
# Switch back to your first Kind cluster for Flux setup
kubectl config use-context kind-k8s-cluster-1

# Install FluxCD CLI (if not already installed)
# brew install fluxcd/flux/flux

# Bootstrap Flux onto your cluster (replace with your Git repo)
# This will install Flux components and configure them to watch your repo.
# Create an *empty* Git repository first (e.g., flux-gitops-repo) on GitHub.
flux bootstrap github \
  --owner=<your-github-username> \
  --repository=<your-flux-repo-name> \
  --branch=main \
  --path=clusters/k8s-cluster-1 # This path will store cluster-specific configs
  --personal # Use if authenticating with a personal access token for GitHub
# Follow prompts to generate a deploy key and add to GitHub.

# Verify Flux installation
flux get sources git
flux get kustomizations
kubectl get pods -n flux-system

# Create a simple Nginx manifest in your Git repo under `clusters/k8s-cluster-1/`
# Commit and push this file to your Flux Git repository
# clusters/k8s-cluster-1/nginx-app.yaml
cat <<EOF > clusters/k8s-cluster-1/nginx-app.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flux-nginx-deployment
  labels:
    app: flux-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flux-nginx
  template:
    metadata:
      labels:
        app: flux-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21.6
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: flux-nginx-service
spec:
  selector:
    app: flux-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
EOF
```

```bash
# Commit and push this file to your Flux Git repository (to the 'clusters/k8s-cluster-1' path)

# Observe Flux deploying the application
flux get kustomizations # Check status of the Kustomization created by bootstrap
kubectl get deployment flux-nginx-deployment
kubectl get service flux-nginx-service

# Make a change in Git (e.g., replicas: 3), commit and push
# Observe Flux detecting and applying the change
```

## **Challenge:** 
Bootstrap FluxCD to your `k8s-cluster-1` Kind cluster, linking it to a new Git repository (`flux-app-repo`). Create a simple Nginx Deployment and Service in this repository under the path specified during bootstrap. Push the changes and verify Flux deploys the application. Make a change in Git and confirm Flux updates the cluster.
