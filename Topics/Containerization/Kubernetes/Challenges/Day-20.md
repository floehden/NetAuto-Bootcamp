# **Day 20: ArgoCD with Helm Charts**

## **Concepts:**
* Deploying Helm charts with ArgoCD.
* Managing Helm values via Git (in `values.yaml` or directly in `Application` spec).
* Overriding chart values from ArgoCD `Application` spec (`helm.values` or `helm.valueFiles`).
* **Loading Custom Images into Kind:** Essential for testing Helm charts that use custom images.

# **Examples (Code):**
```bash
# Use your custom Helm chart from Day 16 (my-custom-app/)
# Create a simple custom image for testing
cat <<EOF > my-custom-image/Dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EOF
echo "<h1>Hello from my Custom Helm App in Kind!</h1>" > my-custom-image/index.html

# Build and load custom image into Kind
docker build -t my-custom-helm-image:v1 my-custom-image/
kind load docker-image my-custom-helm-image:v1 --name k8s-cluster-1

# Modify your my-custom-app/values.yaml to use this image
# image:
#   repository: my-custom-helm-image
#   tag: v1
#   pullPolicy: Never # Crucial for local Kind images

# Create a Git repository for your Helm chart
# Example repo: https://github.com/<your-username>/argocd-helm-app
# Place your 'my-custom-app' chart folder inside this repo (e.g., at the root)

# Create an ArgoCD Application for the Helm chart (argocd-app-helm.yaml)
cat <<EOF > argocd-app-helm.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
    name: my-helm-gitops-app
    namespace: argocd
spec:
    project: default
    source:
    repoURL: https://github.com/<your-username>/argocd-helm-app.git # Your repo with Helm chart
    targetRevision: HEAD
    path: my-custom-app # Path to your Helm chart within the repo
    helm:
        valueFiles:
        - values.yaml # Path to values.yaml within the chart's directory
        values: | # Override specific values directly in Application CR
        replicaCount: 1
        appMessage: "Hello from ArgoCD via Helm! (Override)"
        ingress:
            enabled: true
            hosts:
            - host: helm-app.kind.local
                paths:
                - path: /
                    pathType: Prefix
            className: "nginx" # Specify your IngressClass
    destination:
    server: https://kubernetes.default.svc
    namespace: default
    syncPolicy:
    automated:
        prune: true
        selfHeal: true
EOF
```

```bash
kubectl apply -f argocd-app-helm.yaml -n argocd
# Observe in ArgoCD UI, it will detect the Helm chart and deploy.
kubectl get deployment,svc,ingress -l app.kubernetes.io/instance=my-custom-app-release
kubectl logs <my-helm-gitops-app-release-pod> # Check appMessage from override

# Make a change to values.yaml in Git (e.g., replicaCount: 2) and push
# Or, modify `appMessage` in the `helm.values` section of `argocd-app-helm.yaml` and re-apply
# Observe ArgoCD syncing the change.
```

## **Challenge:** 
Take your custom "Hello World" Helm chart from Day 16. Build a custom Docker image for it and load it into your Kind cluster. Configure ArgoCD to deploy this Helm chart, ensuring it uses your custom image. Modify a parameter (e.g., `appMessage` or `replicaCount`) directly within the ArgoCD `Application` manifest and verify ArgoCD updates the application.
