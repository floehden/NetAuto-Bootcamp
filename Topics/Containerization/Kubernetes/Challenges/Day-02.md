# **Day 2: `kubectl` Basics and Pods**

## **Concepts:**
* Introduction to `kubectl` (the Kubernetes command-line tool).
* Imperative vs. Declarative commands.
* What is a Pod? The smallest deployable unit in Kubernetes, co-located containers, shared network and storage.
* Pod lifecycle, status (`Pending`, `Running`, `Succeeded`, `Failed`, `Unknown`).

## **Examples (Code):**
```bash
# Recreate default Kind cluster for continuity
kind create cluster --name k8s-cluster-1

# Basic kubectl commands with context
kubectl config get-contexts
kubectl config use-context kind-k8s-cluster-1
kubectl get nodes

# Create a simple Nginx Pod Imperatively
kubectl run my-nginx-pod --image=nginx:latest --port=80

# Inspect Pod details
kubectl get pod my-nginx-pod
kubectl describe pod my-nginx-pod
kubectl logs my-nginx-pod

# Delete a Pod
kubectl delete pod my-nginx-pod

# Create a simple Nginx Pod Declaratively (pod.yaml)
cat <<EOF > pod.yaml
apiVersion: v1
kind: Pod
metadata:
    name: declarative-nginx-pod
    labels:
        app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
EOF
kubectl apply -f pod.yaml
kubectl get pod declarative-nginx-pod
kubectl delete -f pod.yaml
```

## **Challenge:** 
Create a Pod with two containers (e.g., Nginx and an `alpine` container that continuously `ping`s Nginx's IP within the Pod). Get the logs from both containers.

## Further readings
* https://kubernetes.io/docs/concepts/workloads/pods/