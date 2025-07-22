# Day 5: Testing the Operator Locally

## **Learning Objectives:**

* Run the Operator locally against a Kubernetes cluster.
* Create a sample Custom Resource.
* Observe the Operator's behavior and the created resources.

## **Challenges:**
* Debugging Go applications.
* Understanding `make run` and `KUBECONFIG`.

## **Code Examples:**

1.  **Build and run locally:**
Ensure your `KUBECONFIG` is set to point to your development cluster (e.g., minikube).

```bash
make install # Install CRDs to the cluster
make run
```
The `make run` command will start your controller locally, connecting to the Kubernetes API server specified by your `KUBECONFIG`.

2.  **Create a sample `HelloWorld` CR:**
Create `config/samples/webapp_v1_helloworld.yaml` with the following content:

```yaml
apiVersion: webapp.example.com/v1
kind: HelloWorld
metadata:
    name: my-first-helloworld
    namespace: default # Or your preferred namespace
spec:
    message: "Hello from my Kubebuilder Operator!"
    replicas: 2
```

Then apply it:

```bash
kubectl apply -f config/samples/webapp_v1_helloworld.yaml
```

3.  **Verify resources:**

```bash
kubectl get helloworlds
kubectl get deployments -l helloworld=my-first-helloworld
kubectl get services -l helloworld=my-first-helloworld
kubectl get configmaps -l helloworld=my-first-helloworld
```

You should see the `Deployment`, `Service`, and `ConfigMap` created by your Operator. Access the Nginx server via the Service's NodePort (e.g., `minikube ip`:30000).

