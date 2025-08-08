# **Day 3: Deployments and ReplicaSets**

## **Concepts:**
* Why Deployments? Managing Pods for desired state, rolling updates, rollbacks, self-healing.
* Understanding ReplicaSets as the underlying controller for Deployments.
* Declarative approach with YAML.

 ## **Examples (Code):**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2 # Initial image
          ports:
          - containerPort: 80
```
```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get replicasets
kubectl get pods -l app=nginx

# Scale a Deployment
kubectl scale deployment nginx-deployment --replicas=5
kubectl get pods -l app=nginx

# Perform a rolling update
# Modify deployment.yaml to image: nginx:1.16.1
# Save the updated deployment.yaml
kubectl apply -f deployment.yaml
kubectl rollout status deployment/nginx-deployment
kubectl get pods -l app=nginx -o wide

# Rollback a Deployment
kubectl rollout history deployment/nginx-deployment
kubectl rollout undo deployment/nginx-deployment
kubectl get pods -l app=nginx -o wide
```

## **Challenge:** 
Create a Deployment with 3 replicas of a `httpd` (Apache) application. Update the image to a non-existent version (e.g., `httpd:non-existent`) to cause a failed rollout, then rollback to the previous working version.

## Further readings
* https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
* https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
