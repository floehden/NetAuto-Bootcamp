# **Day 7: Persistent Storage (Kind-Specific Considerations)**

 ## **Concepts:**
* The need for persistent storage in containers (data survives Pod restarts/deletions).
* PersistentVolumes (PVs): Cluster-wide storage resources.
* PersistentVolumeClaims (PVCs): Requests for PVs by applications.
* StorageClasses: Dynamic provisioning of PVs.
* Access Modes (ReadWriteOnce, ReadOnlyMany, ReadWriteMany).
* **Kind's default storage:** Kind uses a `hostPath` provisioner by default, which maps to a directory on the Docker daemon's host. This is fine for development but not for production.

## **Examples (Code):**
```yaml
# pvc.yaml (Kind's default storage class will auto-provision a hostPath PV)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
    accessModes:
    - ReadWriteOnce
    resources:
      requests:
        storage: 500Mi
```

```bash
kubectl apply -f pvc.yaml
kubectl get pv,pvc

# Pod using PVC (pod-with-pvc.yaml)
cat <<EOF > pod-with-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
    name: data-writer-pod
spec:
    containers:
    - name: writer-container
      image: busybox
      command: ["sh", "-c", "while true; do echo $(date) >> /data/test.txt; sleep 5; done"]
      volumeMounts:
        - name: persistent-storage
          mountPath: /data
    volumes:
    - name: persistent-storage
      persistentVolumeClaim:
        claimName: my-pvc
EOF

kubectl apply -f pod-with-pvc.yaml
kubectl logs data-writer-pod
kubectl delete pod data-writer-pod
# Wait for pod to terminate, then re-create
kubectl apply -f pod-with-pvc.yaml
kubectl logs data-writer-pod # Observe data persists
```

### **Optional: 
Using a `local-path-provisioner` in Kind for more controlled local storage** (more realistic for multi-node Kind setup or testing specific storage classes).
```bash
# Install local-path-provisioner in Kind
# (Refer to https://github.com/rancher/local-path-provisioner for latest instructions)
# Example: kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
kubectl get storageclass # Verify 'local-path' or similar is created

# Then use it in PVC:
cat <<EOF > pvc-with-local-path-sc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc-local-path
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-path # Use the name of the installed storage class
  resources:
    requests:
      storage: 500Mi
EOF
```

## **Challenge:** 
Deploy a simple application (e.g., a counter that stores its value in a file, or a basic `redis` instance) xusing a Deployment and a PVC. Delete the Pod and verify that the data is retained when a new Pod starts. If you have time, try installing `local-path-provisioner` in Kind and use its StorageClass.

## Further Readings
* https://kubernetes.io/docs/concepts/storage/persistent-volumes/