# **Day 9: Ingress: External Access with Routing (Kind-specific)**

# **Concepts:**
* The problem with multiple NodePorts for exposing many applications.
* Ingress: HTTP/HTTPS routing rules for external access to Services.
* Ingress Controllers (Nginx Ingress Controller, Traefik, Istio, etc.) – required to implement Ingress rules.
* Ingress rules: Host-based routing, path-based routing, TLS termination.
* **Kind Ingress Setup:** Using `extraPortMappings` in Kind config for direct host access.

# **Examples (Code):**
```yaml
# kind-config-ingress.yaml (Used to create the Kind cluster)
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: k8s-cluster-1 # Ensure you delete and recreate if existing
nodes:
- role: control-plane
    kubeadmConfigPatches:
    - |
    kind: InitConfiguration
    nodeRegistration:
        kubeletExtraArgs:
        node-labels: "ingress-ready=true" # Label the control-plane node
    extraPortMappings:
    - containerPort: 80
    hostPort: 8080 # Map host port 8080 to container port 80 (HTTP)
    listenAddress: "127.0.0.1" # Listen on localhost
    - containerPort: 443
    hostPort: 8443 # Map host port 8443 to container port 443 (HTTPS)
    listenAddress: "127.0.0.1"
```

```bash
# Delete existing cluster and create with ingress port mappings
kind delete cluster --name k8s-cluster-1
kind create cluster --name k8s-cluster-1 --config kind-config-ingress.yaml
kubectl config use-context kind-k8s-cluster-1

# Install Nginx Ingress Controller
# (Using bare metal manifest for Kind, as Cloud-provider LB won't work)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/cloud/deploy.yaml
# Wait for the Ingress Controller Pods to be Running and LoadBalancer IP to be assigned (it will be pending on Kind, but still accessible via NodePort or direct pod IP)
echo "Waiting for ingress-nginx controller to be ready..."
kubectl wait --namespace ingress-nginx \
    --for=condition=ready pod \
    --selector=app.kubernetes.io/component=controller \
    --timeout=90s
```
```yaml
# app-a-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: app-a-deployment
spec:
    replicas: 1
    selector: { matchLabels: { app: app-a } }
    template:
    metadata: { labels: { app: app-a } }
    spec: { containers: [ { name: app-a, image: nginxdemos/hello:plain-text, ports: [ { containerPort: 80 } ] } ] }
---
# app-a-service.yaml
apiVersion: v1
kind: Service
metadata:
    name: app-a-service
spec:
    selector: { app: app-a }
    ports: [ { protocol: TCP, port: 80, targetPort: 80 } ]
    type: ClusterIP
```

```yaml
# app-b-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: app-b-deployment
spec:
    replicas: 1
    selector: { matchLabels: { app: app-b } }
    template:
    metadata: { labels: { app: app-b } }
    spec: { containers: [ { name: app-b, image: hashicorp/http-echo --text="Hello from App B!", args: ["--listen=:5678"], ports: [ { containerPort: 5678 } ] } ] }
---
# app-b-service.yaml
apiVersion: v1
kind: Service
metadata:
    name: app-b-service
spec:
    selector: { app: app-b }
    ports: [ { protocol: TCP, port: 80, targetPort: 5678 } ] # Map external 80 to internal 5678
    type: ClusterIP
```

```bash
kubectl apply -f app-a-deployment.yaml -f app-a-service.yaml
kubectl apply -f app-b-deployment.yaml -f app-b-service.yaml
```

```yaml
# ingress-routing.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: example-ingress
    annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
    nginx.ingress.kubernetes.io/ssl-redirect: "false" # Disable redirect for http test
spec:
    rules:
    - host: app.local
    http:
        paths:
        - path: /a(/|$)(.*) # Path for app-a
        pathType: Prefix
        backend:
            service:
            name: app-a-service
            port:
                number: 80
        - path: /b(/|$)(.*) # Path for app-b
        pathType: Prefix
        backend:
            service:
            name: app-b-service
            port:
                number: 80
    - host: otherapp.local
    http:
        paths:
        - path: /
        pathType: Prefix
        backend:
            service:
            name: app-b-service
            port:
                number: 80
```

```bash
kubectl apply -f ingress-routing.yaml
kubectl get ingress example-ingress

# Test access on host machine (update /etc/hosts or equivalent)
echo "127.0.0.1 app.local otherapp.local" | sudo tee -a /etc/hosts # Add to your hosts file

echo "Testing Ingress on http://localhost:8080"
curl http://localhost:8080/a
curl http://localhost:8080/b
curl http://localhost:8080/otherapp.local # This will hit the default rule for otherapp.local
```

# **Challenge:** Deploy three different web applications. Configure an Ingress to route requests:
* `app1.kind.local/alpha` to app 1
* `app2.kind.local/beta` to app 2
* All other requests to `app3.kind.local`
Ensure you map the Kind cluster's exposed ports (e.g., 8080) to your host's `/etc/hosts` file.

