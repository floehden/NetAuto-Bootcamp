# **Day 23: FluxCD with Helm and Kustomize**

## **Concepts:**
* Deploying Helm Releases with Flux's Helm Controller (`HelmRelease` CRD).
* Using Kustomize with Flux for environmental overlays (`Kustomization` CRD).
* Image automation with Flux (optional, more advanced topic, concept only).

## **Examples (Code):**
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

## **Challenge:** 
Use FluxCD to deploy your custom Helm chart from Day 16. Then, create a Kustomize base and a `dev` overlay in your Git repository. Configure Flux to deploy the `dev` Kustomize overlay to a new namespace `my-kustomize-dev-ns`, demonstrating how to apply environmental-specific configurations using Kustomize.

