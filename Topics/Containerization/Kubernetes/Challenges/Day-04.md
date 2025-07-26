# **Day 4: Kubernetes Networking Fundamentals & Services**

## **Concepts:**
* The **Kubernetes Network Model**: Flat network, all Pods can communicate without NAT.
* **Pod-to-Pod Communication**: How CNI plugins (Kind uses `kindnetd` by default) enable this.
* **Service DNS**: How Services provide stable names for Pods.
* **Cluster DNS (CoreDNS)**: Resolving Service names to IP addresses.
* Service types: ClusterIP, NodePort, LoadBalancer (Kind simulation), ExternalName.

## **Examples (Code):**
```bash
# Deploy two simple Nginx pods for testing pod-to-pod and DNS
kubectl run nginx-app-1 --image=nginx --labels="app=nginx-test"
kubectl run nginx-app-2 --image=nginx --labels="app=nginx-test"
kubectl get pods -l app=nginx-test -o wide

# Get IPs and test direct connectivity (Kind's CNI allows this)
POD1_IP=$(kubectl get pod nginx-app-1 -o jsonpath='{.status.podIP}')
kubectl exec -it nginx-app-2 -- curl $POD1_IP

# Deploy a ClusterIP Service for nginx-test
cat <<EOF > nginx-service-dns.yaml
apiVersion: v1
kind: Service
metadata:
    name: nginx-test-service
spec:
    selector:
    app: nginx-test
    ports:
    - protocol: TCP
        port: 80
        targetPort: 80
    type: ClusterIP
EOF
```
```bash
kubectl apply -f nginx-service-dns.yaml
kubectl get svc nginx-test-service

# Test DNS resolution from another pod
kubectl run -it --rm test-dns-pod --image=busybox -- nslookup nginx-test-service
kubectl run -it --rm test-dns-pod-fqdn --image=busybox -- nslookup nginx-test-service.default.svc.cluster.local
kubectl run -it --rm test-dns-curl --image=busybox -- wget -O- nginx-test-service:80

# Change Service type to NodePort
# (Re-using deployment.yaml from Day 3)
cat <<EOF > nginx-nodeport-service.yaml
apiVersion: v1
kind: Service
metadata:
    name: nginx-nodeport-service
spec:
    selector:
    app: nginx
    ports:
    - protocol: TCP
        port: 80
        targetPort: 80
        nodePort: 30080 # Optional: You can specify a port or let K8s assign one
    type: NodePort
EOF
```
```bash
kubectl apply -f nginx-nodeport-service.yaml
kubectl get svc nginx-nodeport-service

# Access from outside Kind cluster (using Kind's node IP and NodePort)
NODE_IP=$(docker inspect -f '{{.NetworkSettings.Networks.kind.IPAddress}}' k8s-cluster-1-control-plane)
echo "Access your Nginx app at http://${NODE_IP}:30080"
curl http://${NODE_IP}:30080 # This should work if Nginx deployment is running
```

## **Challenge:**
 Deploy two different applications, `app-green` and `app-blue`, in the `default` namespace. Each should have a Deployment and a ClusterIP Service. From a third diagnostic Pod, verify that you can reach both `app-green-service` and `app-blue-service` using their short DNS names. Then, change `app-green-service` to NodePort and access it from your host machine via the Kind control plane node's IP and NodePort.

