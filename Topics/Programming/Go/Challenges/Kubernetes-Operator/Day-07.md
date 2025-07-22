# Day 7: Deploying the Operator to a Cluster

## **Learning Objectives:**

* Build a Docker image for your Operator.
* Deploy the Operator to a Kubernetes cluster using `make deploy`.
* Understand the necessary RBAC and deployment configurations.

## **Challenges:**

* Container image registry access and authentication.
* Troubleshooting deployment issues in a cluster.

## **Code Examples:**

1.  **Build and push Docker image:**
Replace `your-registry/your-repo` with your Docker Hub or private registry path.

```bash
export IMG=your-registry/your-repo/helloworld-operator:v0.1.0
make docker-build
make docker-push
```

2.  **Deploy to cluster:**
Ensure `KUBECONFIG` points to your target cluster.

```bash
make deploy IMG=${IMG}
```

3.  **Verify deployment:**

```bash
kubectl get deployments -n helloworld-operator-system # Check operator deployment
kubectl get pods -n helloworld-operator-system      # Check operator pod
kubectl apply -f config/samples/webapp_v1_helloworld.yaml # Apply sample CR
kubectl get helloworlds
```
