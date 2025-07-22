**Day 8: DaemonSets and Jobs/CronJobs**

  * **Concepts:**
      * DaemonSets: Ensuring a Pod runs on every (or selected) node, used for cluster-level agents (e.g., logging, monitoring, network proxies).
      * Jobs: Running a task to completion, useful for batch processing, one-time scripts.
      * CronJobs: Scheduling Jobs to run periodically based on a cron schedule.
  * **Examples (Code):**
    ```yaml
    # daemonset.yaml (simple node logger)
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: node-logger
    spec:
      selector:
        matchLabels:
          app: node-logger
      template:
        metadata:
          labels:
            app: node-logger
        spec:
          containers:
          - name: logger
            image: busybox
            command: ["sh", "-c", "while true; do echo \$(date): Running on node \$(NODE_NAME) with IP \$(POD_IP); sleep 10; done"]
            env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: POD_IP
              valueFrom:
                fieldRef:
                  fieldPath: status.podIP
    ```
    ```bash
    kubectl apply -f daemonset.yaml
    kubectl get daemonsets
    kubectl get pods -l app=node-logger -o wide
    kubectl logs <any-node-logger-pod>
    ```
    ```yaml
    # job.yaml (one-time task)
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: pi-calculator
    spec:
      template:
        spec:
          containers:
          - name: pi
            image: perl
            command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
          restartPolicy: OnFailure
      backoffLimit: 4
    ```
    ```bash
    kubectl apply -f job.yaml
    kubectl get jobs
    kubectl get pods -l job-name=pi-calculator
    kubectl logs <pi-calculator-pod>
    ```
    ```yaml
    # cronjob.yaml (scheduled task)
    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: hello-cronjob
    spec:
      schedule: "*/1 * * * *" # Every minute
      jobTemplate:
        spec:
          template:
            spec:
              containers:
              - name: hello
                image: busybox
                command: ["sh", "-c", "echo 'Hello from the cron job!' && date"]
              restartPolicy: OnFailure
    ```
    ```bash
    kubectl apply -f cronjob.yaml
    kubectl get cronjobs
    kubectl get jobs -l cronjob=hello-cronjob
    kubectl get pods -l cronjob=hello-cronjob # Check logs of latest pod
    ```
  * **Challenge:** Create a DaemonSet that simulates a node-level agent (e.g., just logs its hostname and node IP every 5 seconds). Create a CronJob that runs every 2 minutes and performs a simulated cleanup task (e.g., creating and then deleting a temporary file in `/tmp` within the container).

**Day 9: Ingress: External Access with Routing (Kind-specific)**

  * **Concepts:**
      * The problem with multiple NodePorts for exposing many applications.
      * Ingress: HTTP/HTTPS routing rules for external access to Services.
      * Ingress Controllers (Nginx Ingress Controller, Traefik, Istio, etc.) – required to implement Ingress rules.
      * Ingress rules: Host-based routing, path-based routing, TLS termination.
      * **Kind Ingress Setup:** Using `extraPortMappings` in Kind config for direct host access.
  * **Examples (Code):**
    ```yaml
    # kind-config-ingress.yaml (Used to create the Kind cluster)
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    name: k8s-cluster-1 # Ensure you delete and recreate if existing
    nodes:
    - role: control-plane
      kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "ingress-ready=true" # Label the control-plane node
      extraPortMappings:
      - containerPort: 80
        hostPort: 8080 # Map host port 8080 to container port 80 (HTTP)
        listenAddress: "127.0.0.1" # Listen on localhost
      - containerPort: 443
        hostPort: 8443 # Map host port 8443 to container port 443 (HTTPS)
        listenAddress: "127.0.0.1"
    ```
    ```bash
    # Delete existing cluster and create with ingress port mappings
    kind delete cluster --name k8s-cluster-1
    kind create cluster --name k8s-cluster-1 --config kind-config-ingress.yaml
    kubectl config use-context kind-k8s-cluster-1

    # Install Nginx Ingress Controller
    # (Using bare metal manifest for Kind, as Cloud-provider LB won't work)
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/cloud/deploy.yaml
    # Wait for the Ingress Controller Pods to be Running and LoadBalancer IP to be assigned (it will be pending on Kind, but still accessible via NodePort or direct pod IP)
    echo "Waiting for ingress-nginx controller to be ready..."
    kubectl wait --namespace ingress-nginx \
      --for=condition=ready pod \
      --selector=app.kubernetes.io/component=controller \
      --timeout=90s
    ```
    ```yaml
    # app-a-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: app-a-deployment
    spec:
      replicas: 1
      selector: { matchLabels: { app: app-a } }
      template:
        metadata: { labels: { app: app-a } }
        spec: { containers: [ { name: app-a, image: nginxdemos/hello:plain-text, ports: [ { containerPort: 80 } ] } ] }
    ---
    # app-a-service.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: app-a-service
    spec:
      selector: { app: app-a }
      ports: [ { protocol: TCP, port: 80, targetPort: 80 } ]
      type: ClusterIP
    ```
    ```yaml
    # app-b-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: app-b-deployment
    spec:
      replicas: 1
      selector: { matchLabels: { app: app-b } }
      template:
        metadata: { labels: { app: app-b } }
        spec: { containers: [ { name: app-b, image: hashicorp/http-echo --text="Hello from App B!", args: ["--listen=:5678"], ports: [ { containerPort: 5678 } ] } ] }
    ---
    # app-b-service.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: app-b-service
    spec:
      selector: { app: app-b }
      ports: [ { protocol: TCP, port: 80, targetPort: 5678 } ] # Map external 80 to internal 5678
      type: ClusterIP
    ```
    ```bash
    kubectl apply -f app-a-deployment.yaml -f app-a-service.yaml
    kubectl apply -f app-b-deployment.yaml -f app-b-service.yaml
    ```
    ```yaml
    # ingress-routing.yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: example-ingress
      annotations:
        nginx.ingress.kubernetes.io/rewrite-target: /$1
        nginx.ingress.kubernetes.io/ssl-redirect: "false" # Disable redirect for http test
    spec:
      rules:
      - host: app.local
        http:
          paths:
          - path: /a(/|$)(.*) # Path for app-a
            pathType: Prefix
            backend:
              service:
                name: app-a-service
                port:
                  number: 80
          - path: /b(/|$)(.*) # Path for app-b
            pathType: Prefix
            backend:
              service:
                name: app-b-service
                port:
                  number: 80
      - host: otherapp.local
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: app-b-service
                port:
                  number: 80
    ```
    ```bash
    kubectl apply -f ingress-routing.yaml
    kubectl get ingress example-ingress

    # Test access on host machine (update /etc/hosts or equivalent)
    echo "127.0.0.1 app.local otherapp.local" | sudo tee -a /etc/hosts # Add to your hosts file

    echo "Testing Ingress on http://localhost:8080"
    curl http://localhost:8080/a
    curl http://localhost:8080/b
    curl http://localhost:8080/otherapp.local # This will hit the default rule for otherapp.local
    ```
  * **Challenge:** Deploy three different web applications. Configure an Ingress to route requests:
      * `app1.kind.local/alpha` to app 1
      * `app2.kind.local/beta` to app 2
      * All other requests to `app3.kind.local`
        Ensure you map the Kind cluster's exposed ports (e.g., 8080) to your host's `/etc/hosts` file.

**Day 10: Liveness and Readiness Probes**

  * **Concepts:**
      * Liveness Probes: Detect when a container is unhealthy (deadlocked, unresponsive) and restart it.
      * Readiness Probes: Detect when a container is ready to serve traffic (e.g., after initialization, database connection). Prevents traffic from reaching an unready Pod.
      * HTTP, TCP, and Exec probes.
  * **Examples (Code):**
    ```yaml
    # probe-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: my-app-probes
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: my-app-probes
      template:
        metadata:
          labels:
            app: my-app-probes
        spec:
          containers:
          - name: my-container
            image: nginx
            ports:
            - containerPort: 80
            livenessProbe:
              httpGet:
                path: /index.html
                port: 80
              initialDelaySeconds: 5
              periodSeconds: 5
              failureThreshold: 3
            readinessProbe:
              httpGet:
                path: /index.html
                port: 80
              initialDelaySeconds: 10 # Simulate longer startup
              periodSeconds: 5
              failureThreshold: 3
    ```
    ```bash
    kubectl apply -f probe-deployment.yaml
    kubectl get pods -l app=my-app-probes -o wide
    kubectl describe pod <pod-name> # Check Events section for probe status
    ```
      * **Simulating Liveness Probe failure with a custom image:**
        ```dockerfile
        # Dockerfile for a failing app (save as failing-app/Dockerfile)
        FROM alpine/git
        CMD ["sh", "-c", "echo 'Starting healthy...'; sleep 15; echo 'Becoming unhealthy and exiting...'; exit 1"]
        ```
        ```bash
        # Build and load image into Kind
        docker build -t failing-app:v1 failing-app/
        kind load docker-image failing-app:v1 --name k8s-cluster-1

        # failing-app-deployment.yaml
        cat <<EOF > failing-app-deployment.yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: failing-app-deployment
        spec:
          replicas: 1
          selector: { matchLabels: { app: failing-app } }
          template:
            metadata: { labels: { app: failing-app } }
            spec:
              containers:
              - name: failing-container
                image: failing-app:v1
                livenessProbe:
                  exec:
                    command: ["sh", "-c", "if [ -f /tmp/healthy ]; then exit 0; else exit 1; fi"]
                  initialDelaySeconds: 5
                  periodSeconds: 5
                  failureThreshold: 1 # Fail quickly
        EOF
        ```
        ```bash
        kubectl apply -f failing-app-deployment.yaml
        kubectl get pods -l app=failing-app -w # Watch for restarts
        kubectl logs <failing-app-pod>
        ```
  * **Challenge:** Modify the `failing-app-deployment.yaml`. Create a Liveness Probe that checks for a file `/tmp/healthy`. Your container should create this file after 10 seconds, and then delete it after another 10 seconds. Observe the Pod's lifecycle: `Running`, then `CrashLoopBackOff`.

**Day 11: StatefulSets for Stateful Applications**

  * **Concepts:**
      * When to use StatefulSets (stable network identities, ordered deployments/scaling/deletions, unique persistent storage).
      * Differences between Deployments and StatefulSets.
      * Headless Services with StatefulSets for stable network identities and DNS.
  * **Examples (Code):**
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
  * **Challenge:** Deploy a 3-replica StatefulSet of a simple "logger" application. Each replica should write a unique timestamped message to a file in its persistent volume (e.g., `/data/logs.txt`). Verify that each replica maintains its unique identity and data across restarts and scaling operations. Access the logs from each pod using `kubectl exec` to confirm persistence.

**Day 12: Role-Based Access Control (RBAC)**

  * **Concepts:**
      * The importance of RBAC for securing your cluster, principle of least privilege.
      * Service Accounts: Identity for processes running in Pods.
      * Roles: Permissions within a namespace.
      * ClusterRoles: Permissions across the entire cluster.
      * RoleBindings: Granting Roles to Service Accounts/Users/Groups within a namespace.
      * ClusterRoleBindings: Granting ClusterRoles to Service Accounts/Users/Groups cluster-wide.
  * **Examples (Code):**
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
  * **Challenge:** Create a dedicated Service Account for a "monitoring" application. Create a ClusterRole that grants read access to all Deployments and Services across *all* namespaces. Bind the ClusterRole to the Service Account. Deploy a simple Pod that uses this Service Account and attempts to list Deployments and Services in a different namespace (e.g., `kube-system`) to verify the ClusterRoleBinding.

**Day 13: Horizontal Pod Autoscaler (HPA)**

  * **Concepts:**
      * Why HPA? Automatically scaling applications (Deployments, StatefulSets, ReplicaSets) based on observed metrics (CPU, Memory, Custom Metrics).
      * Metrics Server: A prerequisite for HPA, collects resource metrics.
      * How HPA interacts with Deployments/StatefulSets by adjusting replica counts.
  * **Examples (Code):**
    ```bash
    # Ensure Metrics Server is running (essential for HPA)
    # This is usually pre-installed in Kind, but if not:
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
    echo "Waiting for metrics-server to be ready..."
    kubectl wait --namespace kube-system \
      --for=condition=ready pod \
      --selector=app.kubernetes.io/name=metrics-server \
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
  * **Challenge:** Configure an HPA for your web application to scale between 1 and 5 replicas, targeting 70% CPU utilization. Generate sufficient load to trigger scaling up, then stop the load and observe the HPA scaling down.

**Day 14: Resource Management Review & Troubleshooting Basics**

  * **Concepts:**
      * Review of resource requests/limits, quotas, and limit ranges.
      * Common Pod states and how to interpret them (CrashLoopBackOff, ErrImagePull, Pending, OOMKilled).
      * Essential troubleshooting commands (`kubectl describe`, `kubectl logs`, `kubectl events`, `kubectl get events`).
      * Using `kubectl top` for resource usage.
  * **Examples (Code):**
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
  * **Challenge:** Intentionally create a Deployment with a missing or incorrect image tag to trigger an `ErrImagePull`. Use `kubectl describe` and `kubectl events` to diagnose the problem. Then, create a pod that immediately exits (e.g., `image: busybox`, `command: ["exit", "1"]`) to trigger `CrashLoopBackOff`, and diagnose it using logs and describe.
