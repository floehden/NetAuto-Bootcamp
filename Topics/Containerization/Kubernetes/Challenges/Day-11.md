# **Day 11: StatefulSets for Stateful Applications**

## **Concepts:**
* When to use StatefulSets (stable network identities, ordered deployments/scaling/deletions, unique persistent storage).
* Differences between Deployments and StatefulSets.
* Headless Services with StatefulSets for stable network identities and DNS.

## **Examples (Code):**
```yaml
# headless-service.yaml (for StatefulSet DNS)
apiVersion: v1
kind: Service
metadata:
    name: web-headless
spec:
    selector:
        app: nginx-stateful # Matches StatefulSet Pods
    ports:
    - protocol: TCP
      port: 80
      targetPort: 80
    clusterIP: None # Makes it a headless service
```

```yaml
# statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
    name: web
spec:
    serviceName: "web-headless" # Must match headless service name
    replicas: 2
    selector:
        matchLabels:
            app: nginx-stateful
    template:
        metadata:
            labels:
                app: nginx-stateful
        spec:
            containers:
            - name: nginx
              image: nginx:latest
              ports:
              - containerPort: 80
                name: web
              volumeMounts:
              - name: www
                mountPath: /usr/share/nginx/html
    volumeClaimTemplates: # Automatically creates PVCs for each replica
    - metadata:
        name: www
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: standard # Kind's default StorageClass
        resources:
           requests:
                storage: 1Gi
```

```bash
kubectl apply -f headless-service.yaml -f statefulset.yaml
kubectl get statefulsets
kubectl get pods -l app=nginx-stateful -o wide
kubectl get pvc -l app=nginx-stateful

# Observe stable network identities and ordered startup
kubectl exec -it web-0 -- hostname
kubectl exec -it web-1 -- hostname

# Test DNS for individual pods
kubectl run -it --rm debug-sts-pod --image=busybox -- nslookup web-0.web-headless
kubectl run -it --rm debug-sts-pod-2 --image=busybox -- nslookup web-1.web-headless

# Scale the StatefulSet
kubectl scale statefulset web --replicas=3
kubectl get pods -l app=nginx-stateful -o wide # Observe ordered creation (web-2 comes up)
kubectl scale statefulset web --replicas=1 # Observe ordered deletion (web-2 then web-1 goes down)
```

## **Challenge:** 
Deploy a 3-replica StatefulSet of a simple "logger" application. Each replica should write a unique timestamped message to a file in its persistent volume (e.g., `/data/logs.txt`). Verify that each replica maintains its unique identity and data across restarts and scaling operations. Access the logs from each pod using `kubectl exec` to confirm persistence.

## Further Readings
* https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/
* https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/
