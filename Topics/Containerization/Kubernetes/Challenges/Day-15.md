**Day 15: Helm: The Kubernetes Package Manager**

  * **Concepts:**
      * The need for application packaging and templating in Kubernetes (managing complex YAMLs).
      * Helm: Charts (packages), Repositories (chart storage), Releases (deployed instances of charts).
      * Chart structure (`Chart.yaml`, `values.yaml`, `templates/`, `charts/`, `_helpers.tpl`).
      * Templating with Go templates (variables, functions, conditionals).
  * **Examples (Code):**
    ```bash
    # Install Helm CLI (if not already installed)
    # brew install helm # macOS
    # snap install helm --classic # Ubuntu
    # choco install kubernetes-helm # Windows

    # Add a public Helm repository
    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo update

    # Search for charts
    helm search repo nginx
    helm search repo wordpress

    # Install a pre-existing Helm chart (Nginx example)
    helm install my-nginx bitnami/nginx --set service.type=NodePort --set service.nodePorts.http=30088
    kubectl get pods,svc -l app.kubernetes.io/instance=my-nginx

    # Access Nginx via NodePort on Kind (remember Kind's node IP from Day 4)
    NODE_IP=$(docker inspect -f '{{.NetworkSettings.Networks.kind.IPAddress}}' k8s-cluster-1-control-plane)
    echo "Access Nginx at http://${NODE_IP}:30088"
    curl http://${NODE_IP}:30088

    # Inspect a Helm release
    helm list
    helm status my-nginx
    helm get values my-nginx
    helm get manifest my-nginx

    # Upgrade a Helm release with new values
    helm upgrade my-nginx bitnami/nginx --set service.type=NodePort --set service.nodePorts.http=30088 --set replicaCount=2
    helm list
    kubectl get pods -l app.kubernetes.io/instance=my-nginx
    ```
  * **Challenge:** Install a complex application like WordPress and MySQL using a single Helm chart from a public repository (e.g., Bitnami WordPress), customizing the database password and WordPress username using `values.yaml` passed via `helm install -f values.yaml` or `--set`. Ensure you can access the WordPress instance via NodePort on your Kind cluster.

**Day 16: Creating Custom Helm Charts**

  * **Concepts:**
      * Best practices for creating your own charts.
      * Parameterizing charts with `values.yaml` for configurable deployments.
      * Using `_helpers.tpl` for reusable template fragments (e.g., common labels, full name).
      * Conditional rendering of resources based on `values.yaml` (e.g., enable/disable Ingress).
  * **Examples (Code):**
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
      enabled: false # Set to true to enable Ingress
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
  * **Challenge:** Create a Helm chart for a simple "Hello World" application. The chart should allow configuration of the image name, tag, replica count, and expose the application via a configurable Service type (ClusterIP or NodePort) using `values.yaml`. Also, include an optional Ingress that can be enabled/disabled via `values.yaml`, ensuring it works with your Kind cluster's Nginx Ingress Controller setup.

**Day 17: Helm Advanced Features and Best Practices**

  * **Concepts:**
      * Chart dependencies: Managing complex applications composed of multiple charts (e.g., application + database).
      * Hooks (pre-install, post-upgrade, pre-delete): Executing tasks at specific points in the release lifecycle.
      * Linting and testing charts (`helm lint`, `helm test`).
      * Managing releases: `helm rollback`, `helm uninstall`, `helm history`.
  * **Examples (Code):**
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
  * **Challenge:** Extend your "Hello World" chart to include a sub-chart dependency (e.g., a simple `busybox` Pod as a "utility" that runs once on install using a Helm hook and then completes). Ensure you add a Helm test to your chart that verifies the main application's service is reachable. Use `helm lint` and `helm test` on your chart.

**Day 18: Introduction to GitOps**

  * **Concepts:**
      * What is GitOps? Principles (Git as single source of truth, declarative configuration, automated delivery, pull-based reconciliation).
      * Benefits of GitOps (auditability, traceability, faster deployments, disaster recovery, security).
      * Difference from traditional CI/CD (push vs. pull model).
      * Key components: Git repository, Kubernetes cluster, GitOps operator (e.g., ArgoCD, FluxCD).
  * **Examples (Code):**
      * Conceptual explanation of a GitOps workflow:
        1.  Developer commits changes to application configuration (e.g., a YAML file in Git).
        2.  GitOps operator continuously monitors the Git repository.
        3.  Operator detects changes, pulls them.
        4.  Operator applies changes to the Kubernetes cluster.
        5.  Cluster state converges with Git state.
      * Diagrams comparing push-based CI/CD vs. pull-based GitOps.
  * **Challenge:** Research and list three advantages and three potential disadvantages of adopting a GitOps approach compared to traditional CI/CD.

**Day 19: ArgoCD: Declarative GitOps CD**

  * **Concepts:**
      * Introduction to ArgoCD: A declarative GitOps continuous delivery tool.
      * Core components: API Server, Application Controller, Repo Server, Dex, Redis.
      * Applications as custom resources (`Application` CRD).
      * Synchronization strategies (manual, auto-sync), health checks, pruning, self-healing.
  * **Examples (Code):**
    ```bash
    # Install ArgoCD on Kind
    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    echo "Waiting for ArgoCD pods to be ready..."
    kubectl wait --namespace argocd \
      --for=condition=ready pod \
      --selector=app.kubernetes.io/name=argocd-server \
      --timeout=90s

    # Get the ArgoCD UI password
    ARGOCD_SERVER_POD=$(kubectl get pods -n argocd -l app.kubernetes.io/name=argocd-server -o jsonpath='{.items[0].metadata.name}')
    kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d

    # Expose ArgoCD UI via port-forwarding (for Kind local access)
    echo "Port-forwarding ArgoCD server to localhost:8081. Open http://localhost:8081"
    kubectl port-forward svc/argocd-server -n argocd 8081:443 --address 0.0.0.0 & # Run in background
    # Access ArgoCD UI at https://localhost:8081 (username: admin, password: obtained above). Accept self-signed cert.

    # Create a Git repository (e.g., on GitHub) with a simple Nginx Deployment and Service
    # Example repo: https://github.com/<your-username>/argocd-example-app
    # Create the following files in the root of your repo:
    ```
    ```yaml
    # gitops-nginx-app/deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: gitops-nginx-deployment
      labels:
        app: gitops-nginx
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: gitops-nginx
      template:
        metadata:
          labels:
            app: gitops-nginx
        spec:
          containers:
          - name: nginx
            image: nginx:latest
            ports:
            - containerPort: 80
    ```
    ```yaml
    # gitops-nginx-app/service.yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: gitops-nginx-service
    spec:
      selector:
        app: gitops-nginx
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
      type: ClusterIP
    EOF
    ```
    ```bash
    # Commit and push these files to your Git repository

    # Create your first ArgoCD Application (argocd-app-nginx.yaml)
    cat <<EOF > argocd-app-nginx.yaml
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: my-first-gitops-app
      namespace: argocd
    spec:
      project: default
      source:
        repoURL: https://github.com/<your-username>/argocd-example-app.git # Replace with your repo
        targetRevision: HEAD
        path: . # Path within the repo where K8s manifests are
      destination:
        server: https://kubernetes.default.svc # Refers to the in-cluster API server
        namespace: default # Namespace to deploy into
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
    EOF
    ```
    ```bash
    kubectl apply -f argocd-app-nginx.yaml -n argocd
    # Observe in ArgoCD UI: Application will appear, synchronize, and resources will be created.
    kubectl get deployment gitops-nginx-deployment
    kubectl get service gitops-nginx-service

    # Make a change in Git (e.g., change replicas to 3 in deployment.yaml, commit, push)
    # Observe ArgoCD detecting the OutOfSync state and automatically syncing
    ```
  * **Challenge:** Create a Git repository with a simple `httpd` Deployment and Service YAML. Use ArgoCD to deploy this application to your Kind cluster. Make a change to the image tag in Git and observe ArgoCD automatically syncing the change. Access the ArgoCD UI via port-forwarding and explore the application's details.

**Day 20: ArgoCD with Helm Charts**

  * **Concepts:**
      * Deploying Helm charts with ArgoCD.
      * Managing Helm values via Git (in `values.yaml` or directly in `Application` spec).
      * Overriding chart values from ArgoCD `Application` spec (`helm.values` or `helm.valueFiles`).
      * **Loading Custom Images into Kind:** Essential for testing Helm charts that use custom images.
  * **Examples (Code):**
    ```bash
    # Use your custom Helm chart from Day 16 (my-custom-app/)
    # Create a simple custom image for testing
    cat <<EOF > my-custom-image/Dockerfile
    FROM nginx:alpine
    COPY index.html /usr/share/nginx/html/index.html
    EOF
    echo "<h1>Hello from my Custom Helm App in Kind!</h1>" > my-custom-image/index.html

    # Build and load custom image into Kind
    docker build -t my-custom-helm-image:v1 my-custom-image/
    kind load docker-image my-custom-helm-image:v1 --name k8s-cluster-1

    # Modify your my-custom-app/values.yaml to use this image
    # image:
    #   repository: my-custom-helm-image
    #   tag: v1
    #   pullPolicy: Never # Crucial for local Kind images

    # Create a Git repository for your Helm chart
    # Example repo: https://github.com/<your-username>/argocd-helm-app
    # Place your 'my-custom-app' chart folder inside this repo (e.g., at the root)

    # Create an ArgoCD Application for the Helm chart (argocd-app-helm.yaml)
    cat <<EOF > argocd-app-helm.yaml
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: my-helm-gitops-app
      namespace: argocd
    spec:
      project: default
      source:
        repoURL: https://github.com/<your-username>/argocd-helm-app.git # Your repo with Helm chart
        targetRevision: HEAD
        path: my-custom-app # Path to your Helm chart within the repo
        helm:
          valueFiles:
            - values.yaml # Path to values.yaml within the chart's directory
          values: | # Override specific values directly in Application CR
            replicaCount: 1
            appMessage: "Hello from ArgoCD via Helm! (Override)"
            ingress:
              enabled: true
              hosts:
                - host: helm-app.kind.local
                  paths:
                    - path: /
                      pathType: Prefix
              className: "nginx" # Specify your IngressClass
      destination:
        server: https://kubernetes.default.svc
        namespace: default
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
    EOF
    ```
    ```bash
    kubectl apply -f argocd-app-helm.yaml -n argocd
    # Observe in ArgoCD UI, it will detect the Helm chart and deploy.
    kubectl get deployment,svc,ingress -l app.kubernetes.io/instance=my-custom-app-release
    kubectl logs <my-helm-gitops-app-release-pod> # Check appMessage from override

    # Make a change to values.yaml in Git (e.g., replicaCount: 2) and push
    # Or, modify `appMessage` in the `helm.values` section of `argocd-app-helm.yaml` and re-apply
    # Observe ArgoCD syncing the change.
    ```
  * **Challenge:** Take your custom "Hello World" Helm chart from Day 16. Build a custom Docker image for it and load it into your Kind cluster. Configure ArgoCD to deploy this Helm chart, ensuring it uses your custom image. Modify a parameter (e.g., `appMessage` or `replicaCount`) directly within the ArgoCD `Application` manifest and verify ArgoCD updates the application.

**Day 21: ArgoCD Advanced Features & Multi-Cluster Preparation**

  * **Concepts:**
      * Synchronization options: `automated` (auto-sync), `manual` sync, `sync waves` (ordered deployment), `pre/post sync hooks`.
      * Rollback and drift detection with ArgoCD.
      * ApplicationSets for managing multiple applications (e.g., deploying to multiple clusters/namespaces, or multiple instances of an app).
      * RBAC in ArgoCD (users, groups, projects, roles).
      * **Preparation for Multi-Cluster:** Understanding `kubeconfig` files and contexts.
  * **Examples (Code):**
    ```bash
    # Test manual sync and auto-sync in ArgoCD UI (toggle sync policy)

    # Demonstrate drift detection: manually modify a deployed resource
    kubectl scale deployment my-helm-gitops-app-release -n default --replicas=5
    # Observe in ArgoCD UI: Application becomes 'OutOfSync'. Click 'Sync' to revert or 'Diff' to see changes.

    # Rollback in ArgoCD UI: Go to application history, select a previous revision, and click 'Rollback'.

    # Reviewing Kubeconfig for Multi-Cluster
    ls ~/.kube/config
    kubectl config get-contexts
    kubectl config current-context

    # Example of a `kind-config-cluster2.yaml`
    cat <<EOF > kind-config-cluster2.yaml
    kind: Cluster
    apiVersion: kind.x-k8s.io/v1alpha4
    name: k8s-cluster-2
    nodes:
    - role: control-plane
    EOF
    ```
    ```bash
    # Create a second Kind cluster
    kind create cluster --name k8s-cluster-2 --config kind-config-cluster2.yaml
    echo "Waiting for cluster k8s-cluster-2 to be ready..."
    kubectl cluster-info --context kind-k8s-cluster-2

    # List contexts again to see both clusters
    kubectl config get-contexts

    # Add k8s-cluster-2 to ArgoCD (from ArgoCD UI: Settings -> Clusters -> + Register Cluster)
    # It will provide a `kubectl config export` command, copy and run it.
    # Alternatively, you can use ArgoCD CLI (requires argocd cli installed)
    # argocd cluster add kind-k8s-cluster-2
    # Ensure you are targeting the ArgoCD server's port-forwarded address, or its service IP.
    ```
  * **Challenge:** Intentionally manually modify a deployed resource (e.g., change the image tag directly with `kubectl edit deployment` for an ArgoCD-managed app). Observe how ArgoCD detects the drift and then use ArgoCD to either revert (sync) or view the difference. Create a second Kind cluster (`k8s-cluster-2`) and add it to your ArgoCD setup. Verify both clusters appear in the ArgoCD UI.

