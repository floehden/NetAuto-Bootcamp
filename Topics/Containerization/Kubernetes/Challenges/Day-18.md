# **Day 18: Advanced Kubernetes Networking: CNI Deep Dive & Multi-Cluster Considerations**

## **Concepts:**
* **CNI (Container Network Interface):** What it is, how it enables pod networking. Brief overview of common CNIs (Calico, Cilium, Flannel, Kindnetd).
* **Network Policies:** Detailed rules for controlling Pod-to-Pod communication (ingress/egress rules).
* **Multi-Cluster Networking Challenges:** How do Pods across different clusters communicate? (Not directly without external solutions).
* **Service Mesh (Conceptual Introduction):** How a service mesh (Istio, Linkerd) can abstract multi-cluster communication, traffic routing, observability, and security.

## **Examples (Code):**
```yaml
# Network Policy example (only allow ingress from 'frontend' to 'backend' in 'default' namespace)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
    name: allow-frontend-to-backend
    namespace: default
spec:
    podSelector:
        matchLabels:
            app: backend-app
    policyTypes:
    - Ingress
    ingress:
    - from:
      - podSelector: # Pods with label app: frontend-app
          matchLabels:
            app: frontend-app
      ports:
      - protocol: TCP
        port: 80 # Allow traffic to port 80 on backend
```

```bash
kubectl config use-context kind-k8s-cluster-1
# Deploy backend and frontend apps
cat <<EOF > netpol-test-apps.yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: frontend-app }
spec:
    selector: { matchLabels: { app: frontend-app } }
    template:
        metadata: { labels: { app: frontend-app } }
        spec:
            containers:
            - name: frontend
              image: busybox
              command: ["sh", "-c", "sleep 3600"]
---
apiVersion: v1
kind: Service
metadata: { name: frontend-service }
spec:
    selector: { app: frontend-app }
    ports: [ { protocol: TCP, port: 80, targetPort: 80 } ]
    type: ClusterIP
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-app
spec:
  selector:
    matchLabels:
      app: backend-app
  template:
    metadata:
      labels:
        app: backend-app
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo:latest # Correct Docker image name
        ports:
        - containerPort: 80
        command: ["/http-echo"] # The executable inside the container
        args: # The command-line arguments
        - "--text=Hello from Backend!"
        - "--listen=:80"
---
apiVersion: v1
kind: Service
metadata: { name: backend-service }
spec:
    selector: { app: backend-app }
    ports: [ { protocol: TCP, port: 80, targetPort: 80 } ]
    type: ClusterIP
EOF

kubectl apply -f netpol-test-apps.yaml
kubectl apply -f allow-frontend-to-backend.yaml

# Test connectivity from frontend to backend (should work)
kubectl exec -it $(kubectl get pod -l app=frontend-app -o jsonpath='{.items[0].metadata.name}') -- curl backend-service:80

# Test connectivity from another pod (e.g., an unlabelled busybox) to backend (should NOT work)
kubectl run -it --rm test-unauthorized --image=busybox -- curl backend-service:80
```
* **Service Mesh Concept:**
    * Diagram of sidecar proxy injection.
    * Discuss features: Traffic management (routing, retries, circuit breakers), Observability (metrics, tracing, logging), Security (mTLS).
    * Mention tools like Istio, Linkerd, Consul Connect.

##  * **Challenge:** 
Create two namespaces: `internal-apps` and `external-clients`. Deploy a backend application in `internal-apps`. Deploy a frontend application in `external-clients`. Create a Network Policy that *only* allows the frontend application to access the backend application, denying all other ingress traffic to the backend. Verify with `kubectl exec` from both authorized and unauthorized pods.

