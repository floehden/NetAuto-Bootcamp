# **Day 6: ConfigMaps and Secrets**

## **Concepts:**
* ConfigMaps: Storing non-sensitive configuration data (key-value pairs, entire files).
* Secrets: Storing sensitive data (passowords, API keys) in base64 encoded format (not encrypted at rest without external solutions).
* Mounting ConfigMaps/Secrets as environment variables or files (volumes).

## **Examples (Code):**
```bash
# Create ConfigMap from literal values
kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=development
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml

# Create ConfigMap from a file
echo "greeting=Hello from ConfigMap File!" > config.properties
kubectl create configmap app-props --from-file=config.properties
kubectl get configmap app-props -o yaml

# Inject ConfigMap data into a Pod as environment variables (pod-with-configmap-env.yaml)
cat <<EOF > pod-with-configmap-env.yaml
apiVersion: v1
kind: Pod
metadata:
    name: my-app-pod-env
spec:
    containers:
    - name: my-app-container
      image: busybox
      command: ["sh", "-c", "echo My app color is \$APP_COLOR and mode is \$APP_MODE && sleep 3600"]
      envFrom:
      - configMapRef:
          name: app-config
EOF

kubectl apply -f pod-with-configmap-env.yaml
kubectl logs my-app-pod-env

# Mount ConfigMap data as a volume (pod-with-configmap-volume.yaml)
cat <<EOF > pod-with-configmap-volume.yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod-volume
spec:
  containers:
  - name: my-app-container
    image: busybox
    command: ["sh", "-c", "cat /etc/config/greeting.properties && sleep 3600"]
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: app-props
        items:
        - key: config.properties # The key from the ConfigMap
          path: greeting.properties # The file name in the volume
EOF

kubectl apply -f pod-with-configmap-volume.yaml
kubectl logs my-app-pod-volume

# Create a Secret (base64 encoded manually for example)
echo -n "my_super_secret_password" | base64
# Example output: bXlfc3VwZXJfc2VjcmV0X3Bhc3N3b3JkCg==

# secret.yaml
cat <<EOF > secret.yaml
apiVersion: v1
kind: Secret
metadata:
    name: my-app-secret
type: Opaque
data:
    db_password: $(echo -n "my_super_secret_password" | base64) # Base64 encoded password
    api_key: $(echo -n "https_key" | base64) # Base64 encoded 'https_key'
EOF

kubectl apply -f secret.yaml
kubectl get secret my-app-secret -o yaml # Shows base64 encoded
kubectl describe secret my-app-secret # Shows decoded

# Inject Secret into a Pod as environment variable (pod-with-secret-env.yaml)
cat <<EOF > pod-with-secret-env.yaml
apiVersion: v1
kind: Pod
metadata:
    name: my-app-secret-env-pod
spec:
    containers:
    - name: my-app-secret-container
      image: busybox
      command: ["sh", "-c", "echo DB Password is $DB_PASSWORD && sleep 3600"]
      env:
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: my-app-secret
            key: db_password
EOF

kubectl apply -f pod-with-secret-env.yaml
kubectl logs my-app-secret-env-pod
```
  
## **Challenge:** 
Deploy a simple application (e.g., a custom `busybox` script) that reads a database connection string from a Secret and a welcome message from a ConfigMap. Verify the application uses these values by getting its logs.

## Further Readings
* https://kubernetes.io/docs/concepts/configuration/configmap/
* https://kubernetes.io/docs/concepts/configuration/secret/
* https://kubernetes.io/docs/concepts/configuration/overview/