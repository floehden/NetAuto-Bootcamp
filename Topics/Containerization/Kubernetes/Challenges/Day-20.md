## **Day 20: Kubernetes Security, Monitoring & Next Steps**

## **Concepts:**
* **Security Best Practices:** Review of RBAC, Pod Security Standards (PSS), Network Policies, Image Scanning, Secrets Management.
* **Monitoring in Kubernetes:** Importance of metrics, logs, and traces. Overview of common tools (Prometheus, Grafana).
* **Logging in Kubernetes:** Centralized logging. Overview of tools (Fluentd/Fluent Bit, Elasticsearch, Kibana, Loki, Grafana).
* **Observability:** The combination of monitoring, logging, and tracing.
* **What's Next?** Service Meshes in practice, advanced GitOps patterns, serverless (Knative), WebAssembly (Wasm), professional certifications.
  
## **Examples (Code - conceptual/installation links):**
* **Pod Security Standards:**
    * Discuss `restricted`, `baseline` policies.
    * Link to official docs for applying Pod Security Admission.
* **Prometheus & Grafana (Installation Overview):**
```bash
# Install kube-prometheus-stack via Helm (conceptual, requires Helm CLI)
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
kubectl wait --namespace monitoring \
    --for=condition=ready pod \
    --selector=app.kubernetes.io/instance=prometheus \
    --timeout=90s

# get Password
kubectl --namespace monitoring get secrets prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

kubectl port-forward svc/prometheus-grafana -n monitoring 3000:80 
# Access Grafana UI user admin, password from the command before at localhost:3000
```
* **Logging (Conceptual):** Explain `kubectl logs`, sidecar containers for logging, sending logs to a central store.
  * **Challenge:**
    1.  **Security Review:** Pick one of your Deployments from a previous day. Analyze its YAML: Does it follow the principle of least privilege for its Service Account? Does it have resource limits? Could Network Policies enhance its security? Write a short summary of potential improvements.
    2.  **Future Exploration:** Choose one "What's Next" topic (e.g., Istio, Knative, KubeVirt) that interests you the most. Research it briefly and write a short paragraph explaining its purpose and why it's a valuable extension to core Kubernetes.


## Further Reading
* https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-usage-monitoring/
* https://kubernetes.io/docs/concepts/cluster-administration/logging/