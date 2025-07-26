# **Day 27: Custom Resource Definitions (CRDs) & Operators**

## **Concepts:**
* **Extending Kubernetes API:** The need to define custom resources beyond built-in types (Pod, Deployment, Service).
* **Custom Resource Definitions (CRDs):** How to define your own API objects (e.g., `MyApp`, `DatabaseCluster`, `Workflow`).
* **Custom Resources (CRs):** Instances of CRDs.
* **Operators:** Applications that encapsulate operational knowledge for a specific domain (e.g., managing a database, message queue, or CI/CD system).
* **Controller Pattern:** Operators implement this by continually reconciling the desired state (defined in a CR) with the actual state of the cluster.

## **Examples (Code):**
```yaml
# Simple CRD definition (mycrd.yaml) - This defines the *schema* for your new resource
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
    name: myapps.stable.example.com
spec:
    group: stable.example.com
    versions:
    - name: v1
        served: true
        storage: true
        schema:
        openAPIV3Schema:
            type: object
            properties:
            spec:
                type: object
                properties:
                message:
                    type: string
                    description: "The message to display."
                replicas:
                    type: integer
                    description: "Number of replicas for the app."
    scope: Namespaced # Or Cluster
    names:
    plural: myapps
    singular: myapp
    kind: MyApp
    shortNames:
    - ma
```

```bash
kubectl apply -f mycrd.yaml
kubectl get crd # Verify your CRD exists

# Create a Custom Resource (my-custom-resource.yaml) - This is an *instance* of your CRD
apiVersion: stable.example.com/v1
kind: MyApp
metadata:
    name: my-first-custom-app
spec:
    message: "Hello from my Custom Resource!"
    replicas: 2
```

```bash
kubectl apply -f my-custom-resource.yaml
kubectl get myapps
kubectl describe myapp my-first-custom-app

# Operator Concept Discussion:
# An Operator would be a Deployment running in your cluster.
# It would watch for MyApp CRs. When it sees 'my-first-custom-app',
# it would create an Nginx Deployment with 2 replicas and the "Hello" message.
# If replicas changed to 3 in the MyApp CR, the Operator would update the Deployment.
# This involves writing Go code (or using Operator SDK/KubeBuilder).
```

## **Challenge:** 
Define a CRD for a "BlogPost" resource with fields like `title`, `author`, `content`, and `publishedDate`. Apply the CRD to your Kind cluster. Then, create two instances (Custom Resources) of your `BlogPost` resource. Verify they exist using `kubectl get blogposts`.

