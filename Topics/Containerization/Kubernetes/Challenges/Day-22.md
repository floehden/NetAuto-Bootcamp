**Day 22: FluxCD: A Kubernetes-Native GitOps Tool**

  * **Concepts:**
      * Introduction to FluxCD: A CNCF graduated project, Kubernetes-native GitOps.
      * Flux Toolkit components: Source Controller (fetches from Git), Kustomize Controller (applies Kustomize), Helm Controller (applies Helm releases), Notification Controller.
      * Flux's pull-based reconciliation (cluster-side agent pulls changes).
      * Git source management (`GitRepository` CRD).
  * **Examples (Code):**
    ```bash
    # Switch back to your first Kind cluster for Flux setup
    kubectl config use-context kind-k8s-cluster-1

    # Install FluxCD CLI (if not already installed)
    # brew install fluxcd/flux/flux

    # Bootstrap Flux onto your cluster (replace with your Git repo)
    # This will install Flux components and configure them to watch your repo.
    # Create an *empty* Git repository first (e.g., flux-gitops-repo) on GitHub.
    flux bootstrap github \
      --owner=<your-github-username> \
      --repository=<your-flux-repo-name> \
      --branch=main \
      --path=clusters/k8s-cluster-1 # This path will store cluster-specific configs
      --personal # Use if authenticating with a personal access token for GitHub
    # Follow prompts to generate a deploy key and add to GitHub.

    # Verify Flux installation
    flux get sources git
    flux get kustomizations
    kubectl get pods -n flux-system

    # Create a simple Nginx manifest in your Git repo under `clusters/k8s-cluster-1/`
    # Commit and push this file to your Flux Git repository
    # clusters/k8s-cluster-1/nginx-app.yaml
    cat <<EOF > clusters/k8s-cluster-1/nginx-app.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: flux-nginx-deployment
      labels:
        app: flux-nginx
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: flux-nginx
      template:
        metadata:
          labels:
            app: flux-nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.21.6
            ports:
            - containerPort: 80
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: flux-nginx-service
    spec:
      selector:
        app: flux-nginx
      ports:
        - protocol: TCP
          port: 80
          targetPort: 80
      type: ClusterIP
    EOF
    ```
    ```bash
    # Commit and push this file to your Flux Git repository (to the 'clusters/k8s-cluster-1' path)

    # Observe Flux deploying the application
    flux get kustomizations # Check status of the Kustomization created by bootstrap
    kubectl get deployment flux-nginx-deployment
    kubectl get service flux-nginx-service

    # Make a change in Git (e.g., replicas: 3), commit and push
    # Observe Flux detecting and applying the change
    ```
  * **Challenge:** Bootstrap FluxCD to your `k8s-cluster-1` Kind cluster, linking it to a new Git repository (`flux-app-repo`). Create a simple Nginx Deployment and Service in this repository under the path specified during bootstrap. Push the changes and verify Flux deploys the application. Make a change in Git and confirm Flux updates the cluster.

**Day 23: FluxCD with Helm and Kustomize**

  * **Concepts:**
      * Deploying Helm Releases with Flux's Helm Controller (`HelmRelease` CRD).
      * Using Kustomize with Flux for environmental overlays (`Kustomization` CRD).
      * Image automation with Flux (optional, more advanced topic, concept only).
  * **Examples (Code):**
    ```bash
    # Add your custom Helm chart from Day 16 (my-custom-app/) to your Flux Git repository
    # e.g., in a directory: `charts/my-custom-app/`

    # Define a HelmRepository in Git (clusters/k8s-cluster-1/sources/helm-repo.yaml)
    # This refers to the Git repository containing your charts
    cat <<EOF > clusters/k8s-cluster-1/sources/helm-repo.yaml
    apiVersion: source.toolkit.fluxcd.io/v1
    kind: GitRepository
    metadata:
      name: my-charts-source
      namespace: flux-system
    spec:
      interval: 1m
      url: https://github.com/<your-github-username>/<your-flux-repo-name>.git # Your Flux repo
      ref:
        branch: main
    EOF

    # Define a HelmRelease in Git (clusters/k8s-cluster-1/releases/my-app.yaml)
    cat <<EOF > clusters/k8s-cluster-1/releases/my-app.yaml
    apiVersion: helm.toolkit.fluxcd.io/v2beta1
    kind: HelmRelease
    metadata:
      name: my-custom-app-release
      namespace: default # Namespace where the Helm release will be installed
    spec:
      interval: 5m
      chart:
        spec:
          chart: ./charts/my-custom-app # Path to your chart within the GitRepository
          sourceRef:
            kind: GitRepository
            name: my-charts-source
            namespace: flux-system
          version: "0.1.0" # Version of your chart (from my-custom-app/Chart.yaml)
      values:
        replicaCount: 1
        appMessage: "Hello from Flux via Helm!"
        ingress:
          enabled: true
          hosts:
            - host: flux-helm-app.kind.local
              paths:
                - path: /
                  pathType: Prefix
          className: "nginx"
    EOF
    ```
    ```bash
    # Commit and push both helm-repo.yaml and my-app.yaml to your Flux Git repo
    flux get helmreleases
    kubectl get deployment,svc,ingress -l app.kubernetes.io/instance=my-custom-app-release

    # Kustomize example:
    # In your Git repo:
    # app/base/kustomization.yaml
    # app/base/deployment.yaml
    # app/overlays/dev/kustomization.yaml (patches to base for dev env)
    # app/overlays/prod/kustomization.yaml (patches to base for prod env)

    # Define a Flux Kustomization CRD (clusters/k8s-cluster-1/dev-kustomization.yaml)
    cat <<EOF > clusters/k8s-cluster-1/dev-kustomization.yaml
    apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
    kind: Kustomization
    metadata:
      name: dev-app-kustomize
      namespace: flux-system
    spec:
      interval: 1m
      path: ./app/overlays/dev # Path to the Kustomize overlay in your Git repo
      prune: true
      sourceRef:
        kind: GitRepository
        name: <your-flux-repo-name> # Name of your GitRepository source
      targetNamespace: dev-namespace # Namespace where this Kustomization will be applied
      # Make sure dev-namespace exists
    EOF
    ```
    ```bash
    kubectl create namespace dev-namespace # Create target namespace for Kustomization
    # Commit and push dev-kustomization.yaml and your Kustomize structure (app/base, app/overlays/dev)
    flux get kustomizations
    kubectl get all -n dev-namespace
    ```
  * **Challenge:** Use FluxCD to deploy your custom Helm chart from Day 16. Then, create a Kustomize base and a `dev` overlay in your Git repository. Configure Flux to deploy the `dev` Kustomize overlay to a new namespace `my-kustomize-dev-ns`, demonstrating how to apply environmental-specific configurations using Kustomize.

**Day 24: Introduction to Multi-Cluster Kubernetes with Kind**

  * **Concepts:**
      * Why Multi-Cluster? Use cases (High Availability/Disaster Recovery, Geographic Distribution, Regulatory Compliance, Workload Isolation, Scalability).
      * Challenges in Multi-Cluster (Networking, Identity, Observability, Deployment).
      * **Managing Multiple Kind Clusters:** Creating, deleting, and switching contexts.
      * Deploying independent applications to different Kind clusters.
  * **Examples (Code):**
    ```bash
    # Ensure you have k8s-cluster-1 (from Day 1) and k8s-cluster-2 (from Day 21)

    # List all available Kind clusters
    kind get clusters

    # Switch contexts
    kubectl config use-context kind-k8s-cluster-1
    echo "Current context: $(kubectl config current-context)"
    kubectl get nodes

    kubectl config use-context kind-k8s-cluster-2
    echo "Current context: $(kubectl config current-context)"
    kubectl get nodes

    # Deploy App A to Cluster 1
    # app-a-cluster1-deployment.yaml
    cat <<EOF > app-a-cluster1-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: app-a-cluster1
    spec:
      replicas: 1
      selector: { matchLabels: { app: app-a-cluster1 } }
      template:
        metadata: { labels: { app: app-a-cluster1 } }
        spec: { containers: [ { name: app-a, image: nginxdemos/hello:plain-text, ports: [ { containerPort: 80 } ] } ] }
    EOF
    kubectl apply -f app-a-cluster1-deployment.yaml --context kind-k8s-cluster-1
    kubectl get pods -l app=app-a-cluster1 --context kind-k8s-cluster-1

    # Deploy App B to Cluster 2
    # app-b-cluster2-deployment.yaml
    cat <<EOF > app-b-cluster2-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: app-b-cluster2
    spec:
      replicas: 1
      selector: { matchLabels: { app: app-b-cluster2 } }
      template:
        metadata: { labels: { app: app-b-cluster2 } }
        spec: { containers: [ { name: app-b, image: hashicorp/http-echo --text="Hello from Cluster 2!", args: ["--listen=:80"], ports: [ { containerPort: 80 } ] } ] }
    EOF
    kubectl apply -f app-b-cluster2-deployment.yaml --context kind-k8s-cluster-2
    kubectl get pods -l app=app-b-cluster2 --context kind-k8s-cluster-2
    ```
  * **Challenge:** Create a third Kind cluster named `k8s-cluster-3`. Deploy a simple Nginx application to `k8s-cluster-1`, an Apache application to `k8s-cluster-2`, and a custom "Welcome" application (simple HTTP server) to `k8s-cluster-3`. Use `kubectl config use-context` and `kubectl get pods` to confirm deployments on each cluster.

**Day 25: GitOps for Multi-Cluster with ArgoCD ApplicationSets**

  * **Concepts:**
      * **ArgoCD ApplicationSets:** A powerful ArgoCD controller for automating the creation and management of Applications across multiple clusters or namespaces.
      * Generators: List, Cluster, Git, Matrix, SCM Provider (GitHub/GitLab), Pull Request.
      * Templating Applications for multiple targets.
      * **Multi-Cluster GitOps Workflow:** Single Git repo defines apps for multiple clusters.
  * **Examples (Code):**
    ```bash
    # Ensure ArgoCD is running on k8s-cluster-1 (from Day 19)
    kubectl config use-context kind-k8s-cluster-1

    # Ensure k8s-cluster-2 is registered with ArgoCD (from Day 21)
    # Check `argocd cluster list` via argocd CLI or UI

    # Create a new Git repository for multi-cluster apps
    # Example repo: https://github.com/<your-username>/argocd-multicluster-apps
    # Add a simple Nginx Deployment and Service YAML to this repo (e.g., in a `nginx-app/` directory)
    # nginx-app/deployment.yaml (use a generic image like nginx:alpine)
    # nginx-app/service.yaml
    ```
    ```yaml
    # argocd-applicationset-example.yaml (in your ArgoCD Git repo, or apply directly)
    apiVersion: argoproj.io/v1alpha1
    kind: ApplicationSet
    metadata:
      name: common-nginx-apps
      namespace: argocd
    spec:
      generators:
      - clusters: {} # Generates an application for each registered cluster
      template:
        metadata:
          name: '{{ .name }}-nginx-app' # Name of the generated ArgoCD Application
          labels:
            argocd.argoproj.io/managed-by: common-nginx-apps
        spec:
          project: default
          source:
            repoURL: https://github.com/<your-username>/argocd-multicluster-apps.git # Your new Git repo
            targetRevision: HEAD
            path: nginx-app # Path to the Nginx app in the repo
          destination:
            server: '{{ .server }}' # Target cluster server from generator
            namespace: default # Namespace to deploy into on each cluster
          syncPolicy:
            automated:
              prune: true
              selfHeal: true
    EOF
    ```
    ```bash
    kubectl apply -f argocd-applicationset-example.yaml -n argocd
    # Observe in ArgoCD UI: New Applications will be created for each registered cluster (k8s-cluster-1-nginx-app, k8s-cluster-2-nginx-app).
    # Check if pods are running on both Kind clusters:
    kubectl get pods -n default --context kind-k8s-cluster-1 -l app=nginx
    kubectl get pods -n default --context kind-k8s-cluster-2 -l app=nginx

    # Modify `nginx-app/deployment.yaml` in Git (e.g., change replicas from 1 to 2), commit and push.
    # Observe both cluster's applications syncing in ArgoCD UI and scaling up.
    ```
  * **Challenge:** Using ArgoCD ApplicationSets, deploy your "Hello World" Helm chart (from Day 20) to both `k8s-cluster-2` and `k8s-cluster-3`. Ensure each deployment is customized slightly (e.g., `appMessage` includes the cluster name). Verify deployments on all clusters via `kubectl`.

**Day 26: Advanced Kubernetes Networking: CNI Deep Dive & Multi-Cluster Considerations**

  * **Concepts:**
      * **CNI (Container Network Interface):** What it is, how it enables pod networking. Brief overview of common CNIs (Calico, Cilium, Flannel, Kindnetd).
      * **Network Policies:** Detailed rules for controlling Pod-to-Pod communication (ingress/egress rules).
      * **Multi-Cluster Networking Challenges:** How do Pods across different clusters communicate? (Not directly without external solutions).
      * **Service Mesh (Conceptual Introduction):** How a service mesh (Istio, Linkerd) can abstract multi-cluster communication, traffic routing, observability, and security.
  * **Examples (Code):**
    ```yaml
    # Network Policy example (only allow ingress from 'frontend' to 'backend' in 'default' namespace)
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: allow-frontend-to-backend
      namespace: default
    spec:
      podSelector:
        matchLabels:
          app: backend-app
      policyTypes:
        - Ingress
      ingress:
        - from:
            - podSelector: # Pods with label app: frontend-app
                matchLabels:
                  app: frontend-app
          ports:
            - protocol: TCP
              port: 80 # Allow traffic to port 80 on backend
    ```
    ```bash
    kubectl config use-context kind-k8s-cluster-1
    # Deploy backend and frontend apps
    cat <<EOF > netpol-test-apps.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata: { name: frontend-app }
    spec:
      selector: { matchLabels: { app: frontend-app } }
      template:
        metadata: { labels: { app: frontend-app } }
        spec:
          containers:
          - name: frontend
            image: busybox
            command: ["sh", "-c", "sleep 3600"]
    ---
    apiVersion: v1
    kind: Service
    metadata: { name: frontend-service }
    spec:
      selector: { app: frontend-app }
      ports: [ { protocol: TCP, port: 80, targetPort: 80 } ]
      type: ClusterIP
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata: { name: backend-app }
    spec:
      selector: { matchLabels: { app: backend-app } }
      template:
        metadata: { labels: { app: backend-app } }
        spec:
          containers:
          - name: backend
            image: hashicorp/http-echo --text="Hello from Backend!" --listen=:80 # Simple HTTP server
            ports: [ { containerPort: 80 } ]
    ---
    apiVersion: v1
    kind: Service
    metadata: { name: backend-service }
    spec:
      selector: { app: backend-app }
      ports: [ { protocol: TCP, port: 80, targetPort: 80 } ]
      type: ClusterIP
    EOF
    ```
    ```bash
    kubectl apply -f netpol-test-apps.yaml
    kubectl apply -f allow-frontend-to-backend.yaml

    # Test connectivity from frontend to backend (should work)
    kubectl exec -it $(kubectl get pod -l app=frontend-app -o jsonpath='{.items[0].metadata.name}') -- curl backend-service:80

    # Test connectivity from another pod (e.g., an unlabelled busybox) to backend (should NOT work)
    kubectl run -it --rm test-unauthorized --image=busybox -- curl backend-service:80
    ```
      * **Service Mesh Concept:**
          * Diagram of sidecar proxy injection.
          * Discuss features: Traffic management (routing, retries, circuit breakers), Observability (metrics, tracing, logging), Security (mTLS).
          * Mention tools like Istio, Linkerd, Consul Connect.
  * **Challenge:** Create two namespaces: `internal-apps` and `external-clients`. Deploy a backend application in `internal-apps`. Deploy a frontend application in `external-clients`. Create a Network Policy that *only* allows the frontend application to access the backend application, denying all other ingress traffic to the backend. Verify with `kubectl exec` from both authorized and unauthorized pods.

**Day 27: Custom Resource Definitions (CRDs) & Operators**

  * **Concepts:**
      * **Extending Kubernetes API:** The need to define custom resources beyond built-in types (Pod, Deployment, Service).
      * **Custom Resource Definitions (CRDs):** How to define your own API objects (e.g., `MyApp`, `DatabaseCluster`, `Workflow`).
      * **Custom Resources (CRs):** Instances of CRDs.
      * **Operators:** Applications that encapsulate operational knowledge for a specific domain (e.g., managing a database, message queue, or CI/CD system).
      * **Controller Pattern:** Operators implement this by continually reconciling the desired state (defined in a CR) with the actual state of the cluster.
  * **Examples (Code):**
    ```yaml
    # Simple CRD definition (mycrd.yaml) - This defines the *schema* for your new resource
    apiVersion: apiextensions.k8s.io/v1
    kind: CustomResourceDefinition
    metadata:
      name: myapps.stable.example.com
    spec:
      group: stable.example.com
      versions:
        - name: v1
          served: true
          storage: true
          schema:
            openAPIV3Schema:
              type: object
              properties:
                spec:
                  type: object
                  properties:
                    message:
                      type: string
                      description: "The message to display."
                    replicas:
                      type: integer
                      description: "Number of replicas for the app."
      scope: Namespaced # Or Cluster
      names:
        plural: myapps
        singular: myapp
        kind: MyApp
        shortNames:
        - ma
    ```
    ```bash
    kubectl apply -f mycrd.yaml
    kubectl get crd # Verify your CRD exists

    # Create a Custom Resource (my-custom-resource.yaml) - This is an *instance* of your CRD
    apiVersion: stable.example.com/v1
    kind: MyApp
    metadata:
      name: my-first-custom-app
    spec:
      message: "Hello from my Custom Resource!"
      replicas: 2
    ```
    ```bash
    kubectl apply -f my-custom-resource.yaml
    kubectl get myapps
    kubectl describe myapp my-first-custom-app

    # Operator Concept Discussion:
    # An Operator would be a Deployment running in your cluster.
    # It would watch for MyApp CRs. When it sees 'my-first-custom-app',
    # it would create an Nginx Deployment with 2 replicas and the "Hello" message.
    # If replicas changed to 3 in the MyApp CR, the Operator would update the Deployment.
    # This involves writing Go code (or using Operator SDK/KubeBuilder).
    ```
  * **Challenge:** Define a CRD for a "BlogPost" resource with fields like `title`, `author`, `content`, and `publishedDate`. Apply the CRD to your Kind cluster. Then, create two instances (Custom Resources) of your `BlogPost` resource. Verify they exist using `kubectl get blogposts`.

**Day 28: Kubernetes Security, Monitoring & Next Steps**

  * **Concepts:**
      * **Security Best Practices:** Review of RBAC, Pod Security Standards (PSS), Network Policies, Image Scanning, Secrets Management.
      * **Monitoring in Kubernetes:** Importance of metrics, logs, and traces. Overview of common tools (Prometheus, Grafana).
      * **Logging in Kubernetes:** Centralized logging. Overview of tools (Fluentd/Fluent Bit, Elasticsearch, Kibana, Loki, Grafana).
      * **Observability:** The combination of monitoring, logging, and tracing.
      * **What's Next?** Service Meshes in practice, advanced GitOps patterns, serverless (Knative), WebAssembly (Wasm), professional certifications.
  * **Examples (Code - conceptual/installation links):**
      * **Pod Security Standards:**
          * Discuss `restricted`, `baseline` policies.
          * Link to official docs for applying Pod Security Admission.
      * **Prometheus & Grafana (Installation Overview):**
        ```bash
        # Install kube-prometheus-stack via Helm (conceptual, requires Helm CLI)
        # helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
        # helm repo update
        # helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
        # kubectl port-forward svc/prometheus-grafana -n monitoring 3000:80 # Access Grafana UI
        ```
      * **Logging (Conceptual):** Explain `kubectl logs`, sidecar containers for logging, sending logs to a central store.
  * **Challenge:**
    1.  **Security Review:** Pick one of your Deployments from a previous day. Analyze its YAML: Does it follow the principle of least privilege for its Service Account? Does it have resource limits? Could Network Policies enhance its security? Write a short summary of potential improvements.
    2.  **Future Exploration:** Choose one "What's Next" topic (e.g., Istio, Knative, KubeVirt) that interests you the most. Research it briefly and write a short paragraph explaining its purpose and why it's a valuable extension to core Kubernetes.
