# **Day 15: Helm: The Kubernetes Package Manager**

## **Concepts:**
* The need for application packaging and templating in Kubernetes (managing complex YAMLs).
* Helm: Charts (packages), Repositories (chart storage), Releases (deployed instances of charts).
* Chart structure (`Chart.yaml`, `values.yaml`, `templates/`, `charts/`, `_helpers.tpl`).
* Templating with Go templates (variables, functions, conditionals).

## **Examples (Code):**
```bash
# Install Helm CLI (if not already installed)
# brew install helm # macOS
# snap install helm --classic # Ubuntu
# choco install kubernetes-helm # Windows

# Add a public Helm repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo nginx
helm search repo wordpress

# Install a pre-existing Helm chart (Nginx example)
helm install my-nginx bitnami/nginx --set service.type=NodePort --set service.nodePorts.http=30088
kubectl get pods,svc -l app.kubernetes.io/instance=my-nginx

# Access Nginx via NodePort on Kind (remember Kind's node IP from Day 4)
NODE_IP=$(docker inspect -f '{{.NetworkSettings.Networks.kind.IPAddress}}' k8s-cluster-1-control-plane)
echo "Access Nginx at http://${NODE_IP}:30088"
curl http://${NODE_IP}:30088

# Inspect a Helm release
helm list
helm status my-nginx
helm get values my-nginx
helm get manifest my-nginx

# Upgrade a Helm release with new values
helm upgrade my-nginx bitnami/nginx --set service.type=NodePort --set service.nodePorts.http=30088 --set replicaCount=2
helm list
kubectl get pods -l app.kubernetes.io/instance=my-nginx
```
  
## **Challenge:** 
Install a complex application like WordPress and MySQL using a single Helm chart from a public repository (e.g., Bitnami WordPress), customizing the database password and WordPress username using `values.yaml` passed via `helm install -f values.yaml` or `--set`. Ensure you can access the WordPress instance via NodePort on your Kind cluster.

