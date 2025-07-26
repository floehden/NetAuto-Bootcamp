# **Day 5: Namespaces and Resource Management**

## **Concepts:**
* What are Namespaces? Logical isolation for resources, environments, teams.
* `kubectl` context and switching namespaces.
* Resource Quotas: Limiting total resource consumption per namespace.
* Limit Ranges: Setting default/maximum resource requests/limits for Pods within a namespace.

## **Examples (Code):**
```bash
# Create new namespaces
kubectl create namespace dev
kubectl create namespace prod
kubectl get namespaces

# Deploy a simple app into dev namespace (e.g., re-use nginx-deployment.yaml but add -n dev)
kubectl apply -f deployment.yaml -n dev
kubectl get pods -n dev

# Switch context to a namespace (temporary for current commands)
kubectl config set-context --current --namespace=dev
kubectl get pods # Now shows pods in 'dev' namespace
kubectl config set-context --current --namespace=default # Switch back

# Define Resource Quota (quota.yaml)
cat <<EOF > quota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
    name: dev-quota
    namespace: dev
spec:
    hard:
    pods: "5"
    requests.cpu: "1"
    requests.memory: "1Gi"
    limits.cpu: "2"
    limits.memory: "2Gi"
EOF
```
```bash
kubectl apply -f quota.yaml -n dev
kubectl describe resourcequota dev-quota -n dev

# Define Limit Range (limits.yaml)
cat <<EOF > limits.yaml
apiVersion: v1
kind: LimitRange
metadata:
    name: pod-limits
    namespace: dev
spec:
    limits:
    - default:
        cpu: 500m
        memory: 512Mi
    defaultRequest:
        cpu: 100m
        memory: 128Mi
    type: Container
EOF
```
```bash
kubectl apply -f limits.yaml -n dev
kubectl describe limitrange pod-limits -n dev

# Try to deploy a Pod exceeding limits or quota in 'dev' namespace
# Example Pod to test limits (pod-high-cpu.yaml)
cat <<EOF > pod-high-cpu.yaml
apiVersion: v1
kind: Pod
metadata:
    name: high-cpu-pod
    namespace: dev
spec:
    containers:
    - name: high-cpu-container
    image: busybox
    command: ["sh", "-c", "sleep 3600"]
    resources:
        requests:
        cpu: 1.5 # Exceeds quota.requests.cpu
        limits:
        cpu: 3 # Exceeds quota.limits.cpu
EOF
kubectl apply -f pod-high-cpu.yaml -n dev # This should fail
kubectl delete -f pod-high-cpu.yaml -n dev # Clean up attempt
```

## **Challenge:** 
Create a `testing` and `staging` namespace. Deploy a simple application into each. Set a `ResourceQuota` in the `testing` namespace to limit Pods to 2, CPU to 500m, and memory to 256Mi. Attempt to deploy a third Pod in `testing` to see the quota in action.

