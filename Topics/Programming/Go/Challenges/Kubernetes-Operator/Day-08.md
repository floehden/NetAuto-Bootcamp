# Day 8: Cleanup and Further Exploration

## **Learning Objectives:**
* Clean up deployed resources.
* Review best practices for Operator development.
* Identify next steps for more complex Operators.

## **Challenges:**

* Understanding when and how to use Finalizers for graceful deletion.
* Scalability and performance considerations for Operators.

## **Code Examples:**

1.  **Delete the sample CR:**

```bash
kubectl delete -f config/samples/webapp_v1_helloworld.yaml
```

2.  **Undeploy the Operator:**

```bash
make undeploy
```

## **Challenges and Best Practices:**

* **Idempotency:** Your `Reconcile` function must be idempotent. Running it multiple times with the same desired state should always result in the same actual state without side effects.
* **Error Handling and Requeues:** Differentiate between transient errors (requeue with exponential backoff) and permanent errors (do not requeue, log error).
* **Owner References:** Properly set owner references for resources created by your Operator. This ensures Kubernetes garbage collection cleans up dependent resources when the Custom Resource is deleted.
* **Status Updates:** Use the `/status` subresource for status updates. This allows permission separation between changing the `spec` and updating the `status`.
* **Finalizers:** Implement finalizers for complex cleanup logic that needs to happen *before* a CR is truly deleted (e.g., deleting external cloud resources).
* **RBAC:** Adhere to the principle of least privilege. Only grant your Operator the necessary permissions.
* **Testing:** Write unit, integration, and end-to-end tests for your Operator.
* **Logging and Metrics:** Implement robust logging and expose Prometheus metrics for observability.
* **Scalability:** Consider how your Operator will perform with a large number of CRs or under heavy load. Use watches sparingly if not strictly necessary.
* **Community Operators:** Leverage existing Operators for common applications if available, rather than building from scratch.
* **Operator SDK vs. Kubebuilder:** While this tutorial focuses on Kubebuilder, Operator SDK is another excellent tool. Kubebuilder focuses more on the Go framework and `controller-runtime`, while Operator SDK provides additional tooling for Helm and Ansible-based operators and lifecycle management. Often, they can be used together.
