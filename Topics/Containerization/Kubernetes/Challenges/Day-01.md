# Day 1: Introduction to Containerization & Kind Cluster Setup**

## **Concepts:**
  * What are containers? (Docker overview: isolation, portability, lightweight).
  * Why Kubernetes? (Orchestration, scalability, high availability, self-healing).
  * Kubernetes Architecture: Control Plane (API Server, Scheduler, Controller Manager, etcd), Worker Nodes (Kubelet, Kube-proxy, Container Runtime).
  * **Introducing Kind:** A tool for running local Kubernetes clusters using Docker containers as "nodes." Ideal for development and CI.

## **Examples (Code):**
```bash
# Verify Docker is running
docker info

# Install Kind (if not already installed)
# follow the instructions here: https://kind.sigs.k8s.io/docs/user/quick-start/

# Create a simple Kind cluster
kind create cluster --name k8s-cluster-1
echo "Waiting for cluster k8s-cluster-1 to be ready..."
kubectl cluster-info --context kind-k8s-cluster-1

# Verify Kind cluster nodes
kubectl get nodes --context kind-k8s-cluster-1

# Run a simple Docker container (review from previous tutorial)
docker run -d -p 80:80 --name my-nginx-kind-test nginx:latest
curl localhost
docker stop my-nginx-kind-test && docker rm my-nginx-kind-test
```
## **Challenge:** 
Create a Kind cluster named `my-first-cluster`. List its nodes and confirm `kubectl` is configured to interact with it. Delete the cluster.

## Further readings
* https://kind.sigs.k8s.io/docs/user/quick-start/