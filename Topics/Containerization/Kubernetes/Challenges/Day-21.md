# **Day 21: ArgoCD Advanced Features & Multi-Cluster Preparation**

## **Concepts:**
* Synchronization options: `automated` (auto-sync), `manual` sync, `sync waves` (ordered deployment), `pre/post sync hooks`.
* Rollback and drift detection with ArgoCD.
* ApplicationSets for managing multiple applications (e.g., deploying to multiple clusters/namespaces, or multiple instances of an app).
* RBAC in ArgoCD (users, groups, projects, roles).
* **Preparation for Multi-Cluster:** Understanding `kubeconfig` files and contexts.

## **Examples (Code):**
```bash
# Test manual sync and auto-sync in ArgoCD UI (toggle sync policy)

# Demonstrate drift detection: manually modify a deployed resource
kubectl scale deployment my-helm-gitops-app-release -n default --replicas=5
# Observe in ArgoCD UI: Application becomes 'OutOfSync'. Click 'Sync' to revert or 'Diff' to see changes.

# Rollback in ArgoCD UI: Go to application history, select a previous revision, and click 'Rollback'.

# Reviewing Kubeconfig for Multi-Cluster
ls ~/.kube/config
kubectl config get-contexts
kubectl config current-context

# Example of a `kind-config-cluster2.yaml`
cat <<EOF > kind-config-cluster2.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: k8s-cluster-2
nodes:
- role: control-plane
EOF
```

```bash
# Create a second Kind cluster
kind create cluster --name k8s-cluster-2 --config kind-config-cluster2.yaml
echo "Waiting for cluster k8s-cluster-2 to be ready..."
kubectl cluster-info --context kind-k8s-cluster-2

# List contexts again to see both clusters
kubectl config get-contexts

# Add k8s-cluster-2 to ArgoCD (from ArgoCD UI: Settings -> Clusters -> + Register Cluster)
# It will provide a `kubectl config export` command, copy and run it.
# Alternatively, you can use ArgoCD CLI (requires argocd cli installed)
# argocd cluster add kind-k8s-cluster-2
# Ensure you are targeting the ArgoCD server's port-forwarded address, or its service IP.
```

## **Challenge:** 
Intentionally manually modify a deployed resource (e.g., change the image tag directly with `kubectl edit deployment` for an ArgoCD-managed app). Observe how ArgoCD detects the drift and then use ArgoCD to either revert (sync) or view the difference. Create a second Kind cluster (`k8s-cluster-2`) and add it to your ArgoCD setup. Verify both clusters appear in the ArgoCD UI.

