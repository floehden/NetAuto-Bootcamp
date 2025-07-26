# **Day 12: Role-Based Access Control (RBAC)**

## **Concepts:**
* The importance of RBAC for securing your cluster, principle of least privilege.
* Service Accounts: Identity for processes running in Pods.
* Roles: Permissions within a namespace.
* ClusterRoles: Permissions across the entire cluster.
* RoleBindings: Granting Roles to Service Accounts/Users/Groups within a namespace.
* ClusterRoleBindings: Granting ClusterRoles to Service Accounts/Users/Groups cluster-wide.

##  **Examples (Code):**
```yaml
# serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
    name: my-app-sa
```

```yaml
# role.yaml (read-only access to pods in default namespace)
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
    name: pod-reader
    namespace: default
rules:
- apiGroups: [""] # "" indicates the core API group
    resources: ["pods", "pods/log"]
    verbs: ["get", "watch", "list"]
```

```yaml
# rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
    name: read-pods-in-default
    namespace: default
subjects:
- kind: ServiceAccount
    name: my-app-sa
    namespace: default # ServiceAccount's namespace
roleRef:
    kind: Role
    name: pod-reader
    apiGroup: rbac.authorization.k8s.io
```

```yaml
# pod-with-sa.yaml
apiVersion: v1
kind: Pod
metadata:
    name: restricted-pod
spec:
    serviceAccountName: my-app-sa
    containers:
    - name: busybox
    image: busybox
    command: ["sh", "-c", "echo 'Attempting to list pods...'; kubectl get pods; echo 'Attempting to create a pod...'; kubectl run unauthorized-pod --image=nginx --restart=Never; sleep 3600"]
```

```bash
kubectl apply -f serviceaccount.yaml -f role.yaml -f rolebinding.yaml -f pod-with-sa.yaml
kubectl logs restricted-pod # Observe success for get pods, failure for run pod (unauthorized)
```

```yaml
# clusterrole.yaml (conceptual, grants read-only access to all pods cluster-wide)
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
    name: cluster-pod-reader
rules:
- apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "watch", "list"]
```

```yaml
# clusterrolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
    name: my-app-sa-cluster-read
subjects:
- kind: ServiceAccount
    name: my-app-sa
    namespace: default # The SA must exist in this namespace
roleRef:
    kind: ClusterRole
    name: cluster-pod-reader
    apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f clusterrole.yaml -f clusterrolebinding.yaml
# Now, if you run a pod with 'my-app-sa', it can list pods in all namespaces.
# To test this, replace the previous pod-with-sa.yaml's content
# with a command to list pods in 'kube-system' namespace:
# command: ["sh", "-c", "echo 'Attempting to list pods in kube-system...'; kubectl get pods -n kube-system; sleep 3600"]
```

## **Challenge:** 
Create a dedicated Service Account for a "monitoring" application. Create a ClusterRole that grants read access to all Deployments and Services across *all* namespaces. Bind the ClusterRole to the Service Account. Deploy a simple Pod that uses this Service Account and attempts to list Deployments and Services in a different namespace (e.g., `kube-system`) to verify the ClusterRoleBinding.
