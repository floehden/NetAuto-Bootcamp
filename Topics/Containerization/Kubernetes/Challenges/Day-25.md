# **Day 25: GitOps for Multi-Cluster with ArgoCD ApplicationSets**

## * **Concepts:**
* **ArgoCD ApplicationSets:** A powerful ArgoCD controller for automating the creation and management of Applications across multiple clusters or namespaces.
* Generators: List, Cluster, Git, Matrix, SCM Provider (GitHub/GitLab), Pull Request.
* Templating Applications for multiple targets.
* **Multi-Cluster GitOps Workflow:** Single Git repo defines apps for multiple clusters.

## **Examples (Code):**
```bash
# Ensure ArgoCD is running on k8s-cluster-1 (from Day 19)
kubectl config use-context kind-k8s-cluster-1

# Ensure k8s-cluster-2 is registered with ArgoCD (from Day 21)
# Check `argocd cluster list` via argocd CLI or UI

# Create a new Git repository for multi-cluster apps
# Example repo: https://github.com/<your-username>/argocd-multicluster-apps
# Add a simple Nginx Deployment and Service YAML to this repo (e.g., in a `nginx-app/` directory)
# nginx-app/deployment.yaml (use a generic image like nginx:alpine)
# nginx-app/service.yaml
```

```yaml
# argocd-applicationset-example.yaml (in your ArgoCD Git repo, or apply directly)
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
    name: common-nginx-apps
    namespace: argocd
spec:
    generators:
    - clusters: {} # Generates an application for each registered cluster
    template:
    metadata:
        name: '{{ .name }}-nginx-app' # Name of the generated ArgoCD Application
        labels:
        argocd.argoproj.io/managed-by: common-nginx-apps
    spec:
        project: default
        source:
        repoURL: https://github.com/<your-username>/argocd-multicluster-apps.git # Your new Git repo
        targetRevision: HEAD
        path: nginx-app # Path to the Nginx app in the repo
        destination:
        server: '{{ .server }}' # Target cluster server from generator
        namespace: default # Namespace to deploy into on each cluster
        syncPolicy:
        automated:
            prune: true
            selfHeal: true
EOF
```

```bash
kubectl apply -f argocd-applicationset-example.yaml -n argocd
# Observe in ArgoCD UI: New Applications will be created for each registered cluster (k8s-cluster-1-nginx-app, k8s-cluster-2-nginx-app).
# Check if pods are running on both Kind clusters:
kubectl get pods -n default --context kind-k8s-cluster-1 -l app=nginx
kubectl get pods -n default --context kind-k8s-cluster-2 -l app=nginx

# Modify `nginx-app/deployment.yaml` in Git (e.g., change replicas from 1 to 2), commit and push.
# Observe both cluster's applications syncing in ArgoCD UI and scaling up.
```

## **Challenge:** 
Using ArgoCD ApplicationSets, deploy your "Hello World" Helm chart (from Day 20) to both `k8s-cluster-2` and `k8s-cluster-3`. Ensure each deployment is customized slightly (e.g., `appMessage` includes the cluster name). Verify deployments on all clusters via `kubectl`.

