# **Day 16: Creating Custom Helm Charts**

## * **Concepts:**
* Best practices for creating your own charts.
* Parameterizing charts with `values.yaml` for configurable deployments.
* Using `_helpers.tpl` for reusable template fragments (e.g., common labels, full name).
* Conditional rendering of resources based on `values.yaml` (e.g., enable/disable Ingress).

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

# Modify my-custom-app/templates/service.yaml to conditionally create Ingress
# Remove the default Ingress from my-custom-app/templates/ingress.yaml
# Add the Ingress definition to values.yaml and use an if block in templates
```

```yaml
# my-custom-app/templates/ingress.yaml (ensure this file exists and contains the logic)
{{- if .Values.ingress.enabled -}}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
    name: {{ include "my-custom-app.fullname" . }}
    labels:
    {{- include "my-custom-app.labels" . | nindent 4 }}
    {{- with .Values.ingress.annotations }}
    annotations:
    {{- toYaml . | nindent 4 }}
    {{- end }}
spec:
    rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
        http:
        paths:
            {{- range .paths }}
            - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
                service:
                name: {{ include "my-custom-app.fullname" $ }}
                port:
                    number: {{ $.Values.service.port }} # Use the service port
            {{- end }}
    {{- end }}
{{- end }}
```

```yaml
# my-custom-app/values.yaml (add/modify)
replicaCount: 1

appMessage: "Hello from my custom Helm chart!"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true # Set to true to enable Ingress
  className: "" # Use "nginx" if you installed nginx ingress controller
  annotations: {}
  hosts:
  - host: myapp.kind.local
    paths:
    - path: /
      pathType: Prefix
```

```bash
# Install your custom chart
helm install my-custom-release ./my-custom-app
kubectl get pods -l app.kubernetes.io/instance=my-custom-release
kubectl logs <my-custom-app-pod> # Check for appMessage

# Upgrade with Ingress enabled
helm upgrade my-custom-release ./my-custom-app --set ingress.enabled=true --set ingress.hosts[0].host=mycustomapp.kind.local --set ingress.className="nginx"
kubectl get ingress
# Remember to update your /etc/hosts for mycustomapp.kind.local to Kind's control-plane IP and use port 8080 (or your mapped port)
curl http://localhost:8080/ -H "Host: mycustomapp.kind.local" # If ingress is working
```

## **Challenge:** 
Create a Helm chart for a simple "Hello World" application. The chart should allow configuration of the image name, tag, replica count, and expose the application via a configurable Service type (ClusterIP or NodePort) using `values.yaml`. Also, include an optional Ingress that can be enabled/disabled via `values.yaml`, ensuring it works with your Kind cluster's Nginx Ingress Controller setup.

