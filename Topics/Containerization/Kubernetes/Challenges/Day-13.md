# **Day 13: Horizontal Pod Autoscaler (HPA)**

## **Concepts:**
* Why HPA? Automatically scaling applications (Deployments, StatefulSets, ReplicaSets) based on observed metrics (CPU, Memory, Custom Metrics).
* Metrics Server: A prerequisite for HPA, collects resource metrics.
* How HPA interacts with Deployments/StatefulSets by adjusting replica counts.
  
## **Examples (Code):**
```bash
# Ensure Metrics Server is running (essential for HPA)
# This is usually pre-installed in Kind, but if not:
# kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl apply -k kustomization.yaml
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

vi components.yaml

```

Add it the argument `- --kubelet-insecure-tls` to the metrics-server like this:
```yaml
...
  containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=10250
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-insecure-tls # This line
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        image: registry.k8s.io/metrics-server/metrics-server:v0.8.0
        imagePullPolicy: IfNotPresent
...
```

```bash
kubectl apply -f components.yaml
echo "Waiting for metrics-server to be ready..."
kubectl wait --namespace kube-system \
    --for=condition=ready pod \
    --selector=k8s-app=metrics-server \
    --timeout=90s

kubectl top nodes
kubectl top pods
```

```yaml
# php-apache-deployment.yaml (A common HPA example app)
apiVersion: apps/v1
kind: Deployment
metadata:
    name: php-apache
spec:
  selector:
    matchLabels:
      app: php-apache
  replicas: 1
  template:
    metadata:
      labels:
        app: php-apache
    spec:
      containers:
      - name: php-apache
        image: registry.k8s.io/hpa-example # A simple image that consumes CPU when accessed
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 500m
          requests:
            cpu: 200m # HPA targets requests
---
apiVersion: v1
kind: Service
metadata:
  name: php-apache
spec:
  selector:
    app: php-apache
  ports:
  - port: 80
  type: ClusterIP
```

```bash
kubectl apply -f php-apache-deployment.yaml
kubectl get pods -l app=php-apache -o wide
```

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50 # Target 50% CPU utilization
```

```bash
kubectl apply -f hpa.yaml
kubectl get hpa
kubectl describe hpa php-apache-hpa

# Generate load (from another terminal or pod)
# Using a busybox pod to generate load
kubectl run -it --rm load-generator --image=busybox /bin/sh
# Inside load-generator pod:
# while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
# Leave this running for a minute or two.
# Then exit the load-generator pod.

# Observe HPA scaling by repeatedly running:
kubectl get hpa php-apache-hpa -w
kubectl get pods -l app=php-apache -w # Watch pods scale up/down
```

## **Challenge:** 
Configure an HPA for your web application to scale between 1 and 5 replicas, targeting 70% CPU utilization. Generate sufficient load to trigger scaling up, then stop the load and observe the HPA scaling down.

## Further Readings
* https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
* https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/
* https://maggnus.com/install-metrics-server-on-the-kind-kubernetes-cluster-12b0a5faf94atell 