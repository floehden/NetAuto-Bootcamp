
**Day 1: Introduction to Containerization & Kind Cluster Setup**

  * **Concepts:**
      * What are containers? (Docker overview: isolation, portability, lightweight).
      * Why Kubernetes? (Orchestration, scalability, high availability, self-healing).
      * Kubernetes Architecture: Control Plane (API Server, Scheduler, Controller Manager, etcd), Worker Nodes (Kubelet, Kube-proxy, Container Runtime).
      * **Introducing Kind:** A tool for running local Kubernetes clusters using Docker containers as "nodes." Ideal for development and CI.
  * **Examples (Code):**
    ```bash
    # Verify Docker is running
    docker info

    # Install Kind (if not already installed)
    # go install sigs.k8s.io/kind@v0.22.0 # Or use Homebrew/Chocolatey

    # Create a simple Kind cluster
    kind create cluster --name k8s-cluster-1
    echo "Waiting for cluster k8s-cluster-1 to be ready..."
    kubectl cluster-info --context kind-k8s-cluster-1

    # Verify Kind cluster nodes
    kubectl get nodes --context kind-k8s-cluster-1

    # Run a simple Docker container (review from previous tutorial)
    docker run -d -p 80:80 --name my-nginx-kind-test nginx:latest
    curl localhost
    docker stop my-nginx-kind-test && docker rm my-nginx-kind-test
    ```
  * **Challenge:** Create a Kind cluster named `my-first-cluster`. List its nodes and confirm `kubectl` is configured to interact with it. Delete the cluster.

**Day 2: `kubectl` Basics and Pods**

  * **Concepts:**
      * Introduction to `kubectl` (the Kubernetes command-line tool).
      * Imperative vs. Declarative commands.
      * What is a Pod? The smallest deployable unit in Kubernetes, co-located containers, shared network and storage.
      * Pod lifecycle, status (`Pending`, `Running`, `Succeeded`, `Failed`, `Unknown`).
  * **Examples (Code):**
    ```bash
    # Recreate default Kind cluster for continuity
    kind create cluster --name k8s-cluster-1

    # Basic kubectl commands with context
    kubectl config get-contexts
    kubectl config use-context kind-k8s-cluster-1
    kubectl get nodes

    # Create a simple Nginx Pod Imperatively
    kubectl run my-nginx-pod --image=nginx:latest --port=80

    # Inspect Pod details
    kubectl get pod my-nginx-pod
    kubectl describe pod my-nginx-pod
    kubectl logs my-nginx-pod

    # Delete a Pod
    kubectl delete pod my-nginx-pod

    # Create a simple Nginx Pod Declaratively (pod.yaml)
    cat <<EOF > pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: declarative-nginx-pod
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
    EOF
    kubectl apply -f pod.yaml
    kubectl get pod declarative-nginx-pod
    kubectl delete -f pod.yaml
    ```
  * **Challenge:** Create a Pod with two containers (e.g., Nginx and an `alpine` container that continuously `ping`s Nginx's IP within the Pod). Get the logs from both containers.

**Day 3: Deployments and ReplicaSets**

  * **Concepts:**
      * Why Deployments? Managing Pods for desired state, rolling updates, rollbacks, self-healing.
      * Understanding ReplicaSets as the underlying controller for Deployments.
      * Declarative approach with YAML.
  * **Examples (Code):**
    ```yaml
    # deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.14.2 # Initial image
            ports:
            - containerPort: 80
    ```
    ```bash
    kubectl apply -f deployment.yaml
    kubectl get deployments
    kubectl get replicasets
    kubectl get pods -l app=nginx

    # Scale a Deployment
    kubectl scale deployment nginx-deployment --replicas=5
    kubectl get pods -l app=nginx

    # Perform a rolling update
    # Modify deployment.yaml to image: nginx:1.16.1
    # Save the updated deployment.yaml
    kubectl apply -f deployment.yaml
    kubectl rollout status deployment/nginx-deployment
    kubectl get pods -l app=nginx -o wide

    # Rollback a Deployment
    kubectl rollout history deployment/nginx-deployment
    kubectl rollout undo deployment/nginx-deployment
    kubectl get pods -l app=nginx -o wide
    ```
  * **Challenge:** Create a Deployment with 3 replicas of a `httpd` (Apache) application. Update the image to a non-existent version (e.g., `httpd:non-existent`) to cause a failed rollout, then rollback to the previous working version.

**Day 4: Kubernetes Networking Fundamentals & Services**

  * **Concepts:**
      * The **Kubernetes Network Model**: Flat network, all Pods can communicate without NAT.
      * **Pod-to-Pod Communication**: How CNI plugins (Kind uses `kindnetd` by default) enable this.
      * **Service DNS**: How Services provide stable names for Pods.
      * **Cluster DNS (CoreDNS)**: Resolving Service names to IP addresses.
      * Service types: ClusterIP, NodePort, LoadBalancer (Kind simulation), ExternalName.
  * **Examples (Code):**
    ```bash
    # Deploy two simple Nginx pods for testing pod-to-pod and DNS
    kubectl run nginx-app-1 --image=nginx --labels="app=nginx-test"
    kubectl run nginx-app-2 --image=nginx --labels="app=nginx-test"
    kubectl get pods -l app=nginx-test -o wide

    # Get IPs and test direct connectivity (Kind's CNI allows this)
    POD1_IP=$(kubectl get pod nginx-app-1 -o jsonpath='{.status.podIP}')
    kubectl exec -it nginx-app-2 -- curl $POD1_IP

    # Deploy a ClusterIP Service for nginx-test
    cat <<EOF > nginx-service-dns.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-test-service
    spec:
      selector:
        app: nginx-test
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
      type: ClusterIP
    EOF
    ```
    ```bash
    kubectl apply -f nginx-service-dns.yaml
    kubectl get svc nginx-test-service

    # Test DNS resolution from another pod
    kubectl run -it --rm test-dns-pod --image=busybox -- nslookup nginx-test-service
    kubectl run -it --rm test-dns-pod-fqdn --image=busybox -- nslookup nginx-test-service.default.svc.cluster.local
    kubectl run -it --rm test-dns-curl --image=busybox -- wget -O- nginx-test-service:80

    # Change Service type to NodePort
    # (Re-using deployment.yaml from Day 3)
    cat <<EOF > nginx-nodeport-service.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-nodeport-service
    spec:
      selector:
        app: nginx
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
          nodePort: 30080 # Optional: You can specify a port or let K8s assign one
      type: NodePort
    EOF
    ```
    ```bash
    kubectl apply -f nginx-nodeport-service.yaml
    kubectl get svc nginx-nodeport-service

    # Access from outside Kind cluster (using Kind's node IP and NodePort)
    NODE_IP=$(docker inspect -f '{{.NetworkSettings.Networks.kind.IPAddress}}' k8s-cluster-1-control-plane)
    echo "Access your Nginx app at http://${NODE_IP}:30080"
    curl http://${NODE_IP}:30080 # This should work if Nginx deployment is running
    ```
  * **Challenge:** Deploy two different applications, `app-green` and `app-blue`, in the `default` namespace. Each should have a Deployment and a ClusterIP Service. From a third diagnostic Pod, verify that you can reach both `app-green-service` and `app-blue-service` using their short DNS names. Then, change `app-green-service` to NodePort and access it from your host machine via the Kind control plane node's IP and NodePort.

**Day 5: Namespaces and Resource Management**

  * **Concepts:**
      * What are Namespaces? Logical isolation for resources, environments, teams.
      * `kubectl` context and switching namespaces.
      * Resource Quotas: Limiting total resource consumption per namespace.
      * Limit Ranges: Setting default/maximum resource requests/limits for Pods within a namespace.
  * **Examples (Code):**
    ```bash
    # Create new namespaces
    kubectl create namespace dev
    kubectl create namespace prod
    kubectl get namespaces

    # Deploy a simple app into dev namespace (e.g., re-use nginx-deployment.yaml but add -n dev)
    kubectl apply -f deployment.yaml -n dev
    kubectl get pods -n dev

    # Switch context to a namespace (temporary for current commands)
    kubectl config set-context --current --namespace=dev
    kubectl get pods # Now shows pods in 'dev' namespace
    kubectl config set-context --current --namespace=default # Switch back

    # Define Resource Quota (quota.yaml)
    cat <<EOF > quota.yaml
    apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: dev-quota
      namespace: dev
    spec:
      hard:
        pods: "5"
        requests.cpu: "1"
        requests.memory: "1Gi"
        limits.cpu: "2"
        limits.memory: "2Gi"
    EOF
    ```
    ```bash
    kubectl apply -f quota.yaml -n dev
    kubectl describe resourcequota dev-quota -n dev

    # Define Limit Range (limits.yaml)
    cat <<EOF > limits.yaml
    apiVersion: v1
    kind: LimitRange
    metadata:
      name: pod-limits
      namespace: dev
    spec:
      limits:
      - default:
          cpu: 500m
          memory: 512Mi
        defaultRequest:
          cpu: 100m
          memory: 128Mi
        type: Container
    EOF
    ```
    ```bash
    kubectl apply -f limits.yaml -n dev
    kubectl describe limitrange pod-limits -n dev

    # Try to deploy a Pod exceeding limits or quota in 'dev' namespace
    # Example Pod to test limits (pod-high-cpu.yaml)
    cat <<EOF > pod-high-cpu.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: high-cpu-pod
      namespace: dev
    spec:
      containers:
      - name: high-cpu-container
        image: busybox
        command: ["sh", "-c", "sleep 3600"]
        resources:
          requests:
            cpu: 1.5 # Exceeds quota.requests.cpu
          limits:
            cpu: 3 # Exceeds quota.limits.cpu
    EOF
    kubectl apply -f pod-high-cpu.yaml -n dev # This should fail
    kubectl delete -f pod-high-cpu.yaml -n dev # Clean up attempt
    ```
  * **Challenge:** Create a `testing` and `staging` namespace. Deploy a simple application into each. Set a `ResourceQuota` in the `testing` namespace to limit Pods to 2, CPU to 500m, and memory to 256Mi. Attempt to deploy a third Pod in `testing` to see the quota in action.

**Day 6: ConfigMaps and Secrets**

  * **Concepts:**
      * ConfigMaps: Storing non-sensitive configuration data (key-value pairs, entire files).
      * Secrets: Storing sensitive data (passowords, API keys) in base64 encoded format (not encrypted at rest without external solutions).
      * Mounting ConfigMaps/Secrets as environment variables or files (volumes).
  * **Examples (Code):**
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
    ```
    ```bash
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
    ```
    ```bash
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
    ```
    ```bash
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
        command: ["sh", "-c", "echo DB Password is \$DB_PASSWORD && sleep 3600"]
        env:
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: my-app-secret
              key: db_password
    EOF
    ```
    ```bash
    kubectl apply -f pod-with-secret-env.yaml
    kubectl logs my-app-secret-env-pod
    ```
  * **Challenge:** Deploy a simple application (e.g., a custom `busybox` script) that reads a database connection string from a Secret and a welcome message from a ConfigMap. Verify the application uses these values by getting its logs.

**Day 7: Persistent Storage (Kind-Specific Considerations)**

  * **Concepts:**
      * The need for persistent storage in containers (data survives Pod restarts/deletions).
      * PersistentVolumes (PVs): Cluster-wide storage resources.
      * PersistentVolumeClaims (PVCs): Requests for PVs by applications.
      * StorageClasses: Dynamic provisioning of PVs.
      * Access Modes (ReadWriteOnce, ReadOnlyMany, ReadWriteMany).
      * **Kind's default storage:** Kind uses a `hostPath` provisioner by default, which maps to a directory on the Docker daemon's host. This is fine for development but not for production.
  * **Examples (Code):**
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
        command: ["sh", "-c", "while true; do echo \$(date) >> /data/test.txt; sleep 5; done"]
        volumeMounts:
        - name: persistent-storage
          mountPath: /data
      volumes:
      - name: persistent-storage
        persistentVolumeClaim:
          claimName: my-pvc
    EOF
    ```
    ```bash
    kubectl apply -f pod-with-pvc.yaml
    kubectl logs data-writer-pod
    kubectl delete pod data-writer-pod
    # Wait for pod to terminate, then re-create
    kubectl apply -f pod-with-pvc.yaml
    kubectl logs data-writer-pod # Observe data persists
    ```
      * **Optional: Using a `local-path-provisioner` in Kind for more controlled local storage** (more realistic for multi-node Kind setup or testing specific storage classes).
        ```bash
        # Install local-path-provisioner in Kind
        # (Refer to https://github.com/rancher/local-path-provisioner for latest instructions)
        # Example: kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
        kubectl get storageclass # Verify 'local-path' or similar is created

        # Then use it in PVC:
        # pvc-with-local-path-sc.yaml
        # apiVersion: v1
        # kind: PersistentVolumeClaim
        # metadata:
        #   name: my-pvc-local-path
        # spec:
        #   accessModes:
        #     - ReadWriteOnce
        #   storageClassName: local-path # Use the name of the installed storage class
        #   resources:
        #     requests:
        #       storage: 500Mi
        ```
  * **Challenge:** Deploy a simple application (e.g., a counter that stores its value in a file, or a basic `redis` instance) using a Deployment and a PVC. Delete the Pod and verify that the data is retained when a new Pod starts. If you have time, try installing `local-path-provisioner` in Kind and use its StorageClass.

