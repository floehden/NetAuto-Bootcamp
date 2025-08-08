# **Day 14: Resource Management Review & Troubleshooting Basics**

## **Concepts:**
* Review of resource requests/limits, quotas, and limit ranges.
* Common Pod states and how to interpret them (CrashLoopBackOff, ErrImagePull, Pending, OOMKilled).
* Essential troubleshooting commands (`kubectl describe`, `kubectl logs`, `kubectl events`, `kubectl get events`).
* Using `kubectl top` for resource usage.

## **Examples (Code):**
```bash
# Demonstrate a Pod with ErrImagePull
cat <<EOF > bad-image-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: bad-image-pod
spec:
  containers:
  - name: my-container
    image: non-existent-image:latest # This image does not exist
EOF
kubectl apply -f bad-image-pod.yaml
kubectl get pods bad-image-pod
kubectl describe pod bad-image-pod # Look at Events section for ErrImagePull
kubectl logs bad-image-pod # Likely no logs if container never starts
kubectl delete -f bad-image-pod.yaml

# Demonstrate a Pod with CrashLoopBackOff (e.g., from Day 10 challenge)
# Apply failing-app-deployment.yaml from Day 10 and observe.
# kubectl get pods -l app=failing-app -w
# kubectl describe pod <failing-app-pod> # Look for crash details
# kubectl logs <failing-app-pod>

# Using kubectl top
kubectl top pod
kubectl top node
```

## **Challenge:** 
Intentionally create a Deployment with a missing or incorrect image tag to trigger an `ErrImagePull`. Use `kubectl describe` and `kubectl events` to diagnose the problem. Then, create a pod that immediately exits (e.g., `image: busybox`, `command: ["exit", "1"]`) to trigger `CrashLoopBackOff`, and diagnose it using logs and describe.

## Further Readings
* https://kubernetes.io/docs/tasks/debug/debug-application/
* https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/
* https://kubernetes.io/docs/tasks/debug/debug-cluster/