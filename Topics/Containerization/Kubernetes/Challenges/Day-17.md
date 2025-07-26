## **Day 17: Helm Advanced Features and Best Practices**

## **Concepts:**
* Chart dependencies: Managing complex applications composed of multiple charts (e.g., application + database).
* Hooks (pre-install, post-upgrade, pre-delete): Executing tasks at specific points in the release lifecycle.
* Linting and testing charts (`helm lint`, `helm test`).
* Managing releases: `helm rollback`, `helm uninstall`, `helm history`.

## * **Examples (Code):**
```bash
# Chart dependencies: Create a new chart and add a dependency to `my-custom-app`
helm create parent-chart
# Edit parent-chart/Chart.yaml
# dependencies:
# - name: my-custom-app
#   version: "0.1.0" # Version of my-custom-app (from my-custom-app/Chart.yaml)
#   repository: "file://../my-custom-app" # Path to your local chart
# (Ensure my-custom-app is in a sibling directory to parent-chart)

# Example of a Helm Hook (add to my-custom-app/templates/pre-install-hook.yaml)
cat <<EOF > my-custom-app/templates/pre-install-hook.yaml
apiVersion: v1
kind: Pod
metadata:
    name: {{ include "my-custom-app.fullname" . }}-pre-install-hook
    annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
    containers:
    - name: pre-install-container
    image: busybox
    command: ["sh", "-c", "echo 'Running pre-install hook for {{ include \"my-custom-app.fullname\" . }}!'; sleep 5"]
    restartPolicy: Never
EOF
```

```bash
# Install chart with hook
helm install hook-test ./my-custom-app
kubectl get pods -l helm.sh/hook=pre-install,pre-upgrade # Observe the hook pod

# Linting and testing
helm lint ./my-custom-app
# Add a test (my-custom-app/templates/tests/test-connection.yaml)
cat <<EOF > my-custom-app/templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
    name: "{{ include "my-custom-app.fullname" . }}-test-connection"
    labels:
    {{- include "my-custom-app.labels" . | nindent 4 }}
    annotations:
    "helm.sh/hook": test
spec:
    containers:
    - name: wget
        image: busybox
        command: ['wget']
        args: ['{{ include "my-custom-app.fullname" . }}:{{ .Values.service.port }}']
    restartPolicy: Never
EOF
```

```bash
helm test hook-test

# Rollback
helm upgrade hook-test ./my-custom-app --set replicaCount=2 # Upgrade
helm history hook-test
helm rollback hook-test 1 # Rollback to previous revision
```

## **Challenge:**
Extend your "Hello World" chart to include a sub-chart dependency (e.g., a simple `busybox` Pod as a "utility" that runs once on install using a Helm hook and then completes). Ensure you add a Helm test to your chart that verifies the main application's service is reachable. Use `helm lint` and `helm test` on your chart.

