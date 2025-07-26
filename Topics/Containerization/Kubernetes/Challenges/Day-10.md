# **Day 10: Liveness and Readiness Probes**

## **Concepts:**
* Liveness Probes: Detect when a container is unhealthy (deadlocked, unresponsive) and restart it.
* Readiness Probes: Detect when a container is ready to serve traffic (e.g., after initialization, database connection). Prevents traffic from reaching an unready Pod.
* HTTP, TCP, and Exec probes.

## **Examples (Code):**
```yaml
# probe-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: my-app-probes
spec:
    replicas: 1
    selector:
    matchLabels:
        app: my-app-probes
    template:
    metadata:
        labels:
        app: my-app-probes
    spec:
        containers:
        - name: my-container
        image: nginx
        ports:
        - containerPort: 80
        livenessProbe:
            httpGet:
            path: /index.html
            port: 80
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 3
        readinessProbe:
            httpGet:
            path: /index.html
            port: 80
            initialDelaySeconds: 10 # Simulate longer startup
            periodSeconds: 5
            failureThreshold: 3
```

```bash
kubectl apply -f probe-deployment.yaml
kubectl get pods -l app=my-app-probes -o wide
kubectl describe pod <pod-name> # Check Events section for probe status
```

* **Simulating Liveness Probe failure with a custom image:**
```dockerfile
# Dockerfile for a failing app (save as failing-app/Dockerfile)
FROM alpine/git
CMD ["sh", "-c", "echo 'Starting healthy...'; sleep 15; echo 'Becoming unhealthy and exiting...'; exit 1"]
```

```bash
# Build and load image into Kind
docker build -t failing-app:v1 failing-app/
kind load docker-image failing-app:v1 --name k8s-cluster-1

# failing-app-deployment.yaml
cat <<EOF > failing-app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: failing-app-deployment
spec:
    replicas: 1
    selector: { matchLabels: { app: failing-app } }
    template:
    metadata: { labels: { app: failing-app } }
    spec:
        containers:
        - name: failing-container
        image: failing-app:v1
        livenessProbe:
            exec:
            command: ["sh", "-c", "if [ -f /tmp/healthy ]; then exit 0; else exit 1; fi"]
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 1 # Fail quickly
EOF
```

```bash
kubectl apply -f failing-app-deployment.yaml
kubectl get pods -l app=failing-app -w # Watch for restarts
kubectl logs <failing-app-pod>
```

## **Challenge:** 
Modify the `failing-app-deployment.yaml`. Create a Liveness Probe that checks for a file `/tmp/healthy`. Your container should create this file after 10 seconds, and then delete it after another 10 seconds. Observe the Pod's lifecycle: `Running`, then `CrashLoopBackOff`.

