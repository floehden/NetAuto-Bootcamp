# **Day 17: Introduction to Multi-Cluster Kubernetes with Kind**

## **Concepts:**
* Why Multi-Cluster? Use cases (High Availability/Disaster Recovery, Geographic Distribution, Regulatory Compliance, Workload Isolation, Scalability).
* Challenges in Multi-Cluster (Networking, Identity, Observability, Deployment).
* **Managing Multiple Kind Clusters:** Creating, deleting, and switching contexts.
* Deploying independent applications to different Kind clusters.

## **Examples (Code):**

```bash
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

# Ensure you have k8s-cluster-1 and k8s-cluster-2 
# List all available Kind clusters
kind get clusters

# Switch contexts
kubectl config use-context kind-k8s-cluster-1
echo "Current context: $(kubectl config current-context)"
kubectl get nodes

kubectl config use-context kind-k8s-cluster-2
echo "Current context: $(kubectl config current-context)"
kubectl get nodes

# Deploy App A to Cluster 1
# app-a-cluster1-deployment.yaml
cat <<EOF > app-a-cluster1-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-a-cluster1
spec:
  replicas: 1
  selector: { matchLabels: { app: app-a-cluster1 } }
  template:
    metadata: { labels: { app: app-a-cluster1 } }
    spec: { containers: [ { name: app-a, image: nginxdemos/hello:plain-text, ports: [ { containerPort: 80 } ] } ] }
EOF
kubectl apply -f app-a-cluster1-deployment.yaml --context kind-k8s-cluster-1
kubectl get pods -l app=app-a-cluster1 --context kind-k8s-cluster-1

# Deploy App B to Cluster 2
# app-b-cluster2-deployment.yaml
cat <<EOF > app-b-cluster2-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-b-cluster2
spec:
  replicas: 1
  selector: { matchLabels: { app: app-b-cluster2 } }
  template:
    metadata: { labels: { app: app-b-cluster2 } }
    spec: { containers: [ { name: app-b, image: nginx, ports: [ { containerPort: 80 } ] } ] }
EOF
kubectl apply -f app-b-cluster2-deployment.yaml --context kind-k8s-cluster-2
kubectl get pods -l app=app-b-cluster2 --context kind-k8s-cluster-2
```

## **Challenge:** 
Create a third Kind cluster named `k8s-cluster-3`. Deploy a simple Nginx application to `k8s-cluster-1`, an Apache application to `k8s-cluster-2`, and a custom "Welcome" application (simple HTTP server) to `k8s-cluster-3`. Use `kubectl config use-context` and `kubectl get pods` to confirm deployments on each cluster.

