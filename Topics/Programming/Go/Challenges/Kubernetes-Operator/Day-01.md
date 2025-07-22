### Day 1: Setting up the Environment and Project Scaffolding

## **Learning Objectives:**

  * Understand Kubebuilder installation.
  * Initialize a new Kubebuilder project.
  * Explore the generated project structure.

## **Challenges:**
  * Ensuring all prerequisites are met.
  * Understanding the purpose of each generated file.

## **Code Examples:**

1.  **Install Kubebuilder:**

```bash
os=$(go env GOOS)
arch=$(go env GOARCH)
# Download the latest version (adjust as needed)
curl -L -o kubebuilder https://go.kubebuilder.io/dl/latest/${os}/${arch}
chmod +x kubebuilder
sudo mv kubebuilder /usr/local/bin/
kubebuilder version
```

2.  **Initialize a new project:**

```bash
mkdir helloworld-operator
cd helloworld-operator
kubebuilder init --domain example.com --repo github.com/yourusername/helloworld-operator
```

* `--domain`: Used for your API group (e.g., `helloworld.example.com`).
* `--repo`: Go module path for your project.

## **Expected Output/Structure:**
After `kubebuilder init`, you'll see a directory structure similar to this:

```
helloworld-operator/
├── Dockerfile
├── Makefile
├── PROJECT
├── api/
├── bin/
├── config/
│   ├── crd/
│   ├── default/
│   ├── manager/
│   ├── rbac/
│   └── samples/
├── controllers/
├── go.mod
├── go.sum
└── main.go
```

## **Explanation:**

  * `Makefile`: Contains common targets for building, deploying, and testing.
  * `main.go`: Entry point for your Operator.
  * `api/`: Contains the API definitions (Go structs for your Custom Resources).
  * `controllers/`: Contains the controller logic (the `Reconcile` function).
  * `config/`: Kubernetes manifests for deploying your Operator, CRDs, RBAC, etc.
