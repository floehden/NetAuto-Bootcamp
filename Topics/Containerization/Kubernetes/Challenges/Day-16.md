# **Day 16: Creating Custom Helm Charts**

## * **Concepts:**
* Best practices for creating your own charts.
* Parameterizing charts with `values.yaml` for configurable deployments.
* Using `_helpers.tpl` for reusable template fragments (e.g., common labels, full name).
<!-- * Conditional rendering of resources based on `values.yaml` (e.g., enable/disable Ingress). -->

## **Examples (Code):**
```bash
# Create a new Helm chart
helm create my-custom-app

# Explore the generated chart structure
ls my-custom-app
ls my-custom-app/templates

# Modify my-custom-app/templates/deployment.yaml to add an env var from values
# Find the 'containers' section and add:
#           env:
#           - name: APP_MESSAGE
#             value: {{ .Values.appMessage | quote }}

# Modify my-custom-app/values.yaml
# Add: appMessage: "Hello from my custom Helm chart!"
# Ensure replicaCount is 1

```


```yaml
# my-custom-app/values.yaml (add/modify)
replicaCount: 1

appMessage: "Hello from my custom Helm chart!"

service:
  type: ClusterIP
  port: 80
```

```bash
# Install your custom chart
helm install my-custom-release ./my-custom-app
kubectl get pods -l app.kubernetes.io/instance=my-custom-release
kubectl describe pod <my-custom-app-pod> # Check for appMessage


```

## **Challenge:** 
Create a Helm chart for a simple "Hello World" application. The chart should allow configuration of the image name, tag, replica count, and expose the application via a configurable Service type (ClusterIP or NodePort) using `values.yaml`. Also, include an optional Ingress that can be enabled/disabled via `values.yaml`, ensuring it works with your Kind cluster's Nginx Ingress Controller setup.

