## What is a Kubernetes Operator?

A Kubernetes Operator is a method of packaging, deploying, and managing a Kubernetes-native application. Kubernetes applications are complex and stateful, and operators are designed to automate the operational knowledge required to manage these applications, extending the Kubernetes API to handle domain-specific tasks.

Think of it this way: Kubernetes provides powerful primitives like Deployments and StatefulSets for stateless and somewhat stateful applications. However, managing a complex database like PostgreSQL or a message queue like Kafka involves specific operational tasks (e.g., backups, upgrades, scaling, failovers) that go beyond these primitives. An Operator encapsulates this domain-specific operational knowledge into software, allowing you to manage these complex applications in a Kubernetes-native way, as if they were built-in Kubernetes resources.

**Key Components of an Operator:**

  * **Custom Resource Definition (CRD):** Extends the Kubernetes API by defining a new resource type. This allows users to declare the desired state of their application using familiar Kubernetes YAML.
  * **Controller:** A control loop that continuously watches for changes to instances of your Custom Resource (CR). When a change is detected (creation, update, deletion), the controller takes actions to reconcile the actual state with the desired state defined in the CR.

## Why Kubebuilder?

Kubebuilder is a framework for building Kubernetes APIs using the `controller-runtime` library. It provides:

  * **Scaffolding:** Generates boilerplate code for CRDs, controllers, and project structure.
  * **Code Generation:** Automates the creation of client-go code and manifests based on Go structs and marker comments.
  * **Testing Utilities:** Provides tools for testing your controllers.
  * **Best Practices:** Encourages good practices for Operator development.

## Simple Use Case: A "HelloWorld" Operator

We'll build a "HelloWorld" Operator. This operator will manage a custom resource called `HelloWorld`, which, when created, will deploy a simple Nginx web server and expose it via a Kubernetes Service. The `HelloWorld` custom resource will have a `message` field, which the Nginx server will display.

-----

## 8-Day Tutorial Breakdown

This tutorial assumes you have basic familiarity with Go programming, Kubernetes concepts (Pods, Deployments, Services), and `kubectl`.

**Prerequisites:**

  * Go (v1.20 or later recommended)
  * kubectl
  * Docker or Podman
  * A Kubernetes cluster (minikube, kind, or a cloud-based cluster)
  * Kubebuilder (install instructions will be provided on Day 1)

-----

### Day 1: Setting up the Environment and Project Scaffolding

**Learning Objectives:**

  * Understand Kubebuilder installation.
  * Initialize a new Kubebuilder project.
  * Explore the generated project structure.

**Challenges:**

  * Ensuring all prerequisites are met.
  * Understanding the purpose of each generated file.

**Code Examples:**

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

**Expected Output/Structure:**
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

**Explanation:**

  * `Makefile`: Contains common targets for building, deploying, and testing.
  * `main.go`: Entry point for your Operator.
  * `api/`: Contains the API definitions (Go structs for your Custom Resources).
  * `controllers/`: Contains the controller logic (the `Reconcile` function).
  * `config/`: Kubernetes manifests for deploying your Operator, CRDs, RBAC, etc.

-----

### Day 2: Defining the Custom Resource (CRD)

**Learning Objectives:**

  * Generate a new API (CRD and its Go types).
  * Understand the `Spec` and `Status` of a custom resource.
  * Add fields to the Custom Resource `Spec`.

**Challenges:**

  * Designing an effective and stable API for your custom resource.
  * Understanding Go struct tags (`json`, `omitempty`).

**Code Examples:**

1.  **Generate the API:**

    ```bash
    kubebuilder create api --group webapp --version v1 --kind HelloWorld
    ```

    When prompted:

      * `Create Resource [y/n]`: y
      * `Create Controller [y/n]`: y (we'll implement this later)

2.  **Define `HelloWorldSpec` and `HelloWorldStatus`:**
    Open `api/v1/helloworld_types.go` and modify the `HelloWorldSpec` and add `HelloWorldStatus`:

    ```go
    package v1

    import (
    	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    )

    // HelloWorldSpec defines the desired state of HelloWorld
    type HelloWorldSpec struct {
    	// +kubebuilder:validation:MinLength=1
    	// Message to display on the Nginx welcome page.
    	Message string `json:"message,omitempty"`
    	// Number of Nginx replicas.
    	// +kubebuilder:validation:Minimum=1
    	Replicas *int32 `json:"replicas,omitempty"`
    }

    // HelloWorldStatus defines the observed state of HelloWorld
    type HelloWorldStatus struct {
    	// INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
    	// Important: Run "make generate" to regenerate code after modifying this file

    	// Current state of the Nginx deployment managed by the Operator
    	DeploymentReady bool `json:"deploymentReady,omitempty"`
    	// Current number of ready replicas
    	ReadyReplicas int32 `json:"readyReplicas,omitempty"`
    }

    //+kubebuilder:object:root=true
    //+kubebuilder:subresource:status

    // HelloWorld is the Schema for the helloworlds API
    type HelloWorld struct {
    	metav1.TypeMeta   `json:",inline"`
    	metav1.ObjectMeta `json:"metadata,omitempty"`

    	Spec   HelloWorldSpec   `json:"spec,omitempty"`
    	Status HelloWorldStatus `json:"status,omitempty"`
    }

    //+kubebuilder:object:root=true

    // HelloWorldList contains a list of HelloWorld
    type HelloWorldList struct {
    	metav1.TypeMeta `json:",inline"`
    	metav1.ListMeta `json:"metadata,omitempty"`
    	Items           []HelloWorld `json:"items"`
    }

    func init() {
    	SchemeBuilder.Register(&HelloWorld{}, &HelloWorldList{})
    }
    ```

      * `+kubebuilder:validation:MinLength=1`: Example of a validation marker.
      * `+kubebuilder:subresource:status`: Enables the `/status` subresource for efficient status updates.

3.  **Generate manifests and deepcopy methods:**

    ```bash
    make generate
    make manifests
    ```

    This updates `config/crd/bases/webapp.example.com_helloworlds.yaml` and adds `zz_generated.deepcopy.go` for the new types.

-----

### Day 3: Implementing the Controller Logic (Part 1 - Reconciliation)

**Learning Objectives:**

  * Understand the `Reconcile` function.
  * Fetch the Custom Resource.
  * Create a Kubernetes Deployment based on the CR's `Spec`.

**Challenges:**

  * Idempotency of the `Reconcile` loop.
  * Error handling and requeueing.

**Code Examples:**

Open `controllers/helloworld_controller.go`. The core logic resides in the `Reconcile` method.

```go
package controllers

import (
	"context"
	"fmt"
	"time"

	appsv1 "k8s.io/api/apps/v1"
	corev1 "k8s.io/api/core/v1"
	"k8s.io/apimachinery/pkg/api/errors"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/runtime"
	"k8s.io/apimachinery/pkg/types"
	ctrl "sigs.k8s.io/controller-runtime"
	"sigs.k8s.io/controller-runtime/pkg/client"
	"sigs.k8s.io/controller-runtime/pkg/log"
	"sigs.k8s.io/controller-runtime/pkg/controller/controllerutil" // Import for controllerutil

	webappv1 "github.com/yourusername/helloworld-operator/api/v1" // Replace with your repo path
)

// HelloWorldReconciler reconciles a HelloWorld object
type HelloWorldReconciler struct {
	client.Client
	Scheme *runtime.Scheme
}

//+kubebuilder:rbac:groups=webapp.example.com,resources=helloworlds,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups=webapp.example.com,resources=helloworlds/status,verbs=get;update;patch
//+kubebuilder:rbac:groups=webapp.example.com,resources=helloworlds/finalizers,verbs=update
//+kubebuilder:rbac:groups=apps,resources=deployments,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups="",resources=services,verbs=get;list;watch;create;update;patch;delete
//+kubebuilder:rbac:groups="",resources=pods,verbs=get;list;watch

// Reconcile is part of the main kubernetes reconciliation loop which aims to
// move the current state of the cluster closer to the desired state.
// TODO(user): Modify the Reconcile function to compare the state specified by
// the HelloWorld object against the actual cluster state, and then
// perform operations to make the cluster state reflect the state specified by
// the user.
//
// For more details, check Reconcile and its Result here:
// - https://pkg.go.dev/sigs.k8s.io/controller-runtime@v0.18.2/pkg/reconcile
func (r *HelloWorldReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
	_ = log.FromContext(ctx)

	// Fetch the HelloWorld instance
	helloworld := &webappv1.HelloWorld{}
	err := r.Get(ctx, req.NamespacedName, helloworld)
	if err != nil {
		if errors.IsNotFound(err) {
			// Request object not found, could have been deleted after reconcile request.
			// Owned objects are automatically garbage collected. For additional cleanup logic,
			// use finalizers.
			log.Log.Info("HelloWorld resource not found. Ignoring since object must be deleted")
			return ctrl.Result{}, nil
		}
		// Error reading the object - requeue the request.
		log.Log.Error(err, "Failed to get HelloWorld")
		return ctrl.Result{}, err
	}

	// Define the desired Deployment
	desiredDeployment := r.desiredDeployment(helloworld)

	// Check if the Deployment already exists
	foundDeployment := &appsv1.Deployment{}
	err = r.Get(ctx, types.NamespacedName{Name: desiredDeployment.Name, Namespace: desiredDeployment.Namespace}, foundDeployment)
	if err != nil && errors.IsNotFound(err) {
		log.Log.Info("Creating a new Deployment", "Deployment.Namespace", desiredDeployment.Namespace, "Deployment.Name", desiredDeployment.Name)
		// Set the HelloWorld instance as the owner and controller
		if err = controllerutil.SetControllerReference(helloworld, desiredDeployment, r.Scheme); err != nil {
			log.Log.Error(err, "Failed to set controller reference for Deployment")
			return ctrl.Result{}, err
		}
		err = r.Create(ctx, desiredDeployment)
		if err != nil {
			log.Log.Error(err, "Failed to create new Deployment", "Deployment.Namespace", desiredDeployment.Namespace, "Deployment.Name", desiredDeployment.Name)
			return ctrl.Result{}, err
		}
		// Deployment created successfully - return and requeue
		return ctrl.Result{Requeue: true}, nil // Requeue to check status
	} else if err != nil {
		log.Log.Error(err, "Failed to get Deployment")
		return ctrl.Result{}, err
	}

	// Update the Deployment if the spec differs
	if !r.deploymentEquals(desiredDeployment, foundDeployment) {
		log.Log.Info("Updating Deployment", "Deployment.Namespace", foundDeployment.Namespace, "Deployment.Name", foundDeployment.Name)
		foundDeployment.Spec = desiredDeployment.Spec
		err = r.Update(ctx, foundDeployment)
		if err != nil {
			log.Log.Error(err, "Failed to update Deployment", "Deployment.Namespace", foundDeployment.Namespace, "Deployment.Name", foundDeployment.Name)
			return ctrl.Result{}, err
		}
		return ctrl.Result{Requeue: true}, nil // Requeue to check status
	}

	// Update HelloWorld status if it's not consistent with the Deployment
	deploymentReady := foundDeployment.Status.ReadyReplicas == *foundDeployment.Spec.Replicas
	if helloworld.Status.DeploymentReady != deploymentReady || helloworld.Status.ReadyReplicas != foundDeployment.Status.ReadyReplicas {
		helloworld.Status.DeploymentReady = deploymentReady
		helloworld.Status.ReadyReplicas = foundDeployment.Status.ReadyReplicas
		log.Log.Info("Updating HelloWorld status", "Name", helloworld.Name, "DeploymentReady", deploymentReady, "ReadyReplicas", foundDeployment.Status.ReadyReplicas)
		err := r.Status().Update(ctx, helloworld)
		if err != nil {
			log.Log.Error(err, "Failed to update HelloWorld status")
			return ctrl.Result{}, err
		}
	}

	return ctrl.Result{}, nil
}

// desiredDeployment returns a Deployment object for the HelloWorld CR
func (r *HelloWorldReconciler) desiredDeployment(helloworld *webappv1.HelloWorld) *appsv1.Deployment {
	labels := map[string]string{
		"app":        "nginx",
		"helloworld": helloworld.Name,
	}
	messageContent := fmt.Sprintf("Hello from Operator! Message: %s", helloworld.Spec.Message)
	replicas := int32(1) // Default to 1 replica
	if helloworld.Spec.Replicas != nil {
		replicas = *helloworld.Spec.Replicas
	}

	return &appsv1.Deployment{
		ObjectMeta: metav1.ObjectMeta{
			Name:      helloworld.Name + "-deployment",
			Namespace: helloworld.Namespace,
			Labels:    labels,
		},
		Spec: appsv1.DeploymentSpec{
			Replicas: &replicas,
			Selector: &metav1.LabelSelector{
				MatchLabels: labels,
			},
			Template: corev1.PodTemplateSpec{
				ObjectMeta: metav1.ObjectMeta{
					Labels: labels,
				},
				Spec: corev1.PodSpec{
					Containers: []corev1.Container{{
						Name:  "nginx",
						Image: "nginx:latest", // Or a custom image with the message
						Ports: []corev1.ContainerPort{{
							ContainerPort: 80,
							Name:          "http",
						}},
						VolumeMounts: []corev1.VolumeMount{
							{
								Name:      "nginx-config",
								MountPath: "/etc/nginx/conf.d",
								ReadOnly:  true,
							},
						},
					}},
					Volumes: []corev1.Volume{
						{
							Name: "nginx-config",
							ConfigMap: &corev1.ConfigMapVolumeSource{
								LocalObjectReference: corev1.LocalObjectReference{
									Name: helloworld.Name + "-config",
								},
							},
						},
					},
				},
			},
		},
	}
}

// deploymentEquals checks if two deployments are semantically equal for reconciliation purposes
func (r *HelloWorldReconciler) deploymentEquals(desired, current *appsv1.Deployment) bool {
	// Simple check: compare replicas and image.
	// For a real operator, you'd need a more comprehensive deep comparison.
	if *desired.Spec.Replicas != *current.Spec.Replicas {
		return false
	}
	if len(desired.Spec.Template.Spec.Containers) > 0 && len(current.Spec.Template.Spec.Containers) > 0 {
		if desired.Spec.Template.Spec.Containers[0].Image != current.Spec.Template.Spec.Containers[0].Image {
			return false
		}
	}
	return true
}

// SetupWithManager sets up the controller with the Manager.
func (r *HelloWorldReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&webappv1.HelloWorld{}).
		Owns(&appsv1.Deployment{}). // Watch Deployments owned by HelloWorld
		Complete(r)
}
```

**Note:** The `deploymentEquals` function is a simplified example. In a production-grade operator, you would need a more robust comparison of the deployment's spec to truly detect differences and trigger updates. You'd likely use a diffing library or carefully compare relevant fields.

-----

### Day 4: Implementing the Controller Logic (Part 2 - Services and ConfigMaps)

**Learning Objectives:**

  * Create a Kubernetes Service to expose the Nginx Deployment.
  * Create a ConfigMap to serve the custom message.
  * Understand owner references for garbage collection.

**Challenges:**

  * Managing multiple dependent resources.
  * Ensuring correct ConfigMap content.

**Code Examples:**

Continue modifying `controllers/helloworld_controller.go`. Add these functions and integrate them into the `Reconcile` loop.

```go
// (Inside HelloWorldReconciler struct, after Reconcile)

// desiredService returns a Service object for the HelloWorld CR
func (r *HelloWorldReconciler) desiredService(helloworld *webappv1.HelloWorld) *corev1.Service {
	labels := map[string]string{
		"app":        "nginx",
		"helloworld": helloworld.Name,
	}

	return &corev1.Service{
		ObjectMeta: metav1.ObjectMeta{
			Name:      helloworld.Name + "-service",
			Namespace: helloworld.Namespace,
			Labels:    labels,
		},
		Spec: corev1.ServiceSpec{
			Selector: labels,
			Ports: []corev1.ServicePort{
				{
					Protocol:   corev1.ProtocolTCP,
					Port:       80,
					TargetPort: intstr.FromInt(80),
					NodePort:   30000, // Example NodePort, adjust as needed or use LoadBalancer/ClusterIP
				},
			},
			Type: corev1.ServiceTypeNodePort, // Or ClusterIP, LoadBalancer
		},
	}
}

// desiredConfigMap returns a ConfigMap object for the HelloWorld CR
func (r *HelloWorldReconciler) desiredConfigMap(helloworld *webappv1.HelloWorld) *corev1.ConfigMap {
	messageContent := fmt.Sprintf(`
server {
    listen 80;
    location / {
        return 200 "%s\n";
    }
}`, helloworld.Spec.Message)

	return &corev1.ConfigMap{
		ObjectMeta: metav1.ObjectMeta{
			Name:      helloworld.Name + "-config",
			Namespace: helloworld.Namespace,
			Labels: map[string]string{
				"app":        "nginx",
				"helloworld": helloworld.Name,
			},
		},
		Data: map[string]string{
			"default.conf": messageContent,
		},
	}
}
```

**Integrate into `Reconcile` (add after Deployment logic):**

```go
// ... (inside Reconcile function)

	// Define the desired ConfigMap
	desiredConfigMap := r.desiredConfigMap(helloworld)

	// Check if the ConfigMap already exists
	foundConfigMap := &corev1.ConfigMap{}
	err = r.Get(ctx, types.NamespacedName{Name: desiredConfigMap.Name, Namespace: desiredConfigMap.Namespace}, foundConfigMap)
	if err != nil && errors.IsNotFound(err) {
		log.Log.Info("Creating a new ConfigMap", "ConfigMap.Namespace", desiredConfigMap.Namespace, "ConfigMap.Name", desiredConfigMap.Name)
		if err = controllerutil.SetControllerReference(helloworld, desiredConfigMap, r.Scheme); err != nil {
			log.Log.Error(err, "Failed to set controller reference for ConfigMap")
			return ctrl.Result{}, err
		}
		err = r.Create(ctx, desiredConfigMap)
		if err != nil {
			log.Log.Error(err, "Failed to create new ConfigMap", "ConfigMap.Namespace", desiredConfigMap.Namespace, "ConfigMap.Name", desiredConfigMap.Name)
			return ctrl.Result{}, err
		}
		return ctrl.Result{Requeue: true}, nil
	} else if err != nil {
		log.Log.Error(err, "Failed to get ConfigMap")
		return ctrl.Result{}, err
	}

	// Update ConfigMap if content differs (simple check)
	if foundConfigMap.Data["default.conf"] != desiredConfigMap.Data["default.conf"] {
		log.Log.Info("Updating ConfigMap", "ConfigMap.Namespace", foundConfigMap.Namespace, "ConfigMap.Name", foundConfigMap.Name)
		foundConfigMap.Data = desiredConfigMap.Data
		err = r.Update(ctx, foundConfigMap)
		if err != nil {
			log.Log.Error(err, "Failed to update ConfigMap", "ConfigMap.Namespace", foundConfigMap.Namespace, "ConfigMap.Name", foundConfigMap.Name)
			return ctrl.Result{}, err
		}
		return ctrl.Result{Requeue: true}, nil
	}

	// Define the desired Service
	desiredService := r.desiredService(helloworld)

	// Check if the Service already exists
	foundService := &corev1.Service{}
	err = r.Get(ctx, types.NamespacedName{Name: desiredService.Name, Namespace: desiredService.Namespace}, foundService)
	if err != nil && errors.IsNotFound(err) {
		log.Log.Info("Creating a new Service", "Service.Namespace", desiredService.Namespace, "Service.Name", desiredService.Name)
		if err = controllerutil.SetControllerReference(helloworld, desiredService, r.Scheme); err != nil {
			log.Log.Error(err, "Failed to set controller reference for Service")
			return ctrl.Result{}, err
		}
		err = r.Create(ctx, desiredService)
		if err != nil {
			log.Log.Error(err, "Failed to create new Service", "Service.Namespace", desiredService.Namespace, "Service.Name", desiredService.Name)
			return ctrl.Result{}, err
		}
		return ctrl.Result{Requeue: true}, nil
	} else if err != nil {
		log.Log.Error(err, "Failed to get Service")
		return ctrl.Result{}, err
	}

	// For simplicity, we're not checking for Service updates here.
	// In a real scenario, you would compare `desiredService.Spec` with `foundService.Spec`
	// and update if necessary.

// ... (end of Reconcile function)
```

**Also, add `intstr "k8s.io/apimachinery/pkg/util/intstr"` import.**

**And update `SetupWithManager` to watch Services and ConfigMaps:**

```go
// SetupWithManager sets up the controller with the Manager.
func (r *HelloWorldReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&webappv1.HelloWorld{}).
		Owns(&appsv1.Deployment{}).
		Owns(&corev1.Service{}).    // Watch Services owned by HelloWorld
		Owns(&corev1.ConfigMap{}). // Watch ConfigMaps owned by HelloWorld
		Complete(r)
}
```

-----

### Day 5: Testing the Operator Locally

**Learning Objectives:**

  * Run the Operator locally against a Kubernetes cluster.
  * Create a sample Custom Resource.
  * Observe the Operator's behavior and the created resources.

**Challenges:**

  * Debugging Go applications.
  * Understanding `make run` and `KUBECONFIG`.

**Code Examples:**

1.  **Build and run locally:**
    Ensure your `KUBECONFIG` is set to point to your development cluster (e.g., minikube).

    ```bash
    make install # Install CRDs to the cluster
    make run
    ```

    The `make run` command will start your controller locally, connecting to the Kubernetes API server specified by your `KUBECONFIG`.

2.  **Create a sample `HelloWorld` CR:**
    Create `config/samples/webapp_v1_helloworld.yaml` with the following content:

    ```yaml
    apiVersion: webapp.example.com/v1
    kind: HelloWorld
    metadata:
      name: my-first-helloworld
      namespace: default # Or your preferred namespace
    spec:
      message: "Hello from my Kubebuilder Operator!"
      replicas: 2
    ```

    Then apply it:

    ```bash
    kubectl apply -f config/samples/webapp_v1_helloworld.yaml
    ```

3.  **Verify resources:**

    ```bash
    kubectl get helloworlds
    kubectl get deployments -l helloworld=my-first-helloworld
    kubectl get services -l helloworld=my-first-helloworld
    kubectl get configmaps -l helloworld=my-first-helloworld
    ```

    You should see the `Deployment`, `Service`, and `ConfigMap` created by your Operator. Access the Nginx server via the Service's NodePort (e.g., `minikube ip`:30000).

-----

### Day 6: Advanced Topics: Webhooks (Optional but Recommended)

**Learning Objectives:**

  * Understand Admission Webhooks (Validating and Mutating).
  * Generate a webhook using Kubebuilder.
  * Implement basic validation for your Custom Resource.

**Challenges:**

  * Understanding Kubernetes admission control.
  * Debugging webhook interactions.

**Code Examples:**

1.  **Create a Validating Webhook:**

    ```bash
    kubebuilder create webhook --group webapp --version v1 --kind HelloWorld --defaulting --programmatic-validation
    ```

      * `--defaulting`: For `Defaulter` interface.
      * `--programmatic-validation`: For `Validator` interface.

2.  **Implement Validation Logic (e.g., `api/v1/helloworld_webhook.go`):**

    ```go
    package v1

    import (
    	"k8s.io/apimachinery/pkg/runtime"
    	ctrl "sigs.k8s.io/controller-runtime"
    	logf "sigs.k8s.io/controller-runtime/pkg/log"
    	"sigs.k8s.io/controller-runtime/pkg/webhook"
    	"sigs.k8s.io/controller-runtime/pkg/webhook/admission"
    )

    // log is for logging in this package.
    var helloworldlog = logf.Log.WithName("helloworld-resource")

    func (r *HelloWorld) SetupWebhookWithManager(mgr ctrl.Manager) error {
    	return ctrl.NewWebhookManagedBy(mgr).
    		For(r).
    		Complete()
    }

    // TODO(user): change verbs to "verbs=create;update;delete" if you want to enable deletion validation.
    //+kubebuilder:webhook:path=/mutate-webapp-example-com-v1-helloworld,mutating=true,failurePolicy=fail,sideEffects=None,groups=webapp.example.com,resources=helloworlds,verbs=create;update,versions=v1,name=mhelloworld.kb.io,admissionReviewVersions=v1
    //+kubebuilder:webhook:path=/validate-webapp-example-com-v1-helloworld,mutating=false,failurePolicy=fail,sideEffects=None,groups=webapp.example.com,resources=helloworlds,verbs=create;update,versions=v1,name=vhelloworld.kb.io,admissionReviewVersions=v1

    var _ webhook.Defaulter = &HelloWorld{}
    var _ webhook.Validator = &HelloWorld{}

    // Default implements webhook.Defaulter so a webhook will be registered for the type
    func (r *HelloWorld) Default() {
    	helloworldlog.Info("default", "name", r.Name)
    	if r.Spec.Replicas == nil {
    		defaultReplicas := int32(1)
    		r.Spec.Replicas = &defaultReplicas
    	}
    	if r.Spec.Message == "" {
    		r.Spec.Message = "Default message from Kubebuilder Operator!"
    	}
    }

    // ValidateCreate implements webhook.Validator so a webhook will be registered for the type
    func (r *HelloWorld) ValidateCreate() (admission.Warnings, error) {
    	helloworldlog.Info("validate create", "name", r.Name)
    	if r.Spec.Replicas != nil && *r.Spec.Replicas < 1 {
    		return nil, fmt.Errorf("replicas must be a positive integer, got %d", *r.Spec.Replicas)
    	}
    	return nil, nil
    }

    // ValidateUpdate implements webhook.Validator so a webhook will be registered for the type
    func (r *HelloWorld) ValidateUpdate(old runtime.Object) (admission.Warnings, error) {
    	helloworldlog.Info("validate update", "name", r.Name)
    	oldHelloWorld := old.(*HelloWorld)
    	if r.Spec.Replicas != nil && *r.Spec.Replicas < 1 {
    		return nil, fmt.Errorf("replicas must be a positive integer, got %d", *r.Spec.Replicas)
    	}
    	// Example: prevent message change if it's already set (simple immutable check)
    	if oldHelloWorld.Spec.Message != "" && r.Spec.Message != oldHelloWorld.Spec.Message {
    		return nil, fmt.Errorf("message field is immutable after creation")
    	}
    	return nil, nil
    }

    // ValidateDelete implements webhook.Validator so a webhook will be registered for the type
    func (r *HelloWorld) ValidateDelete() (admission.Warnings, error) {
    	helloworldlog.Info("validate delete", "name", r.Name)
    	// TODO(user): fill in your validation logic upon object deletion.
    	return nil, nil
    }
    ```

3.  **Register webhook in `main.go`:**

    ```go
    // ... (inside main function, after client and scheme setup)
    if err = (&webappv1.HelloWorld{}).SetupWebhookWithManager(mgr); err != nil {
    	setupLog.Error(err, "unable to create webhook", "webhook", "HelloWorld")
    	os.Exit(1)
    }
    ```

4.  **Re-generate and run:**

    ```bash
    make generate
    make manifests
    make install
    make run
    ```

    Test by applying `HelloWorld` CRs with invalid replica counts or attempting to change the message.

-----

### Day 7: Deploying the Operator to a Cluster

**Learning Objectives:**

  * Build a Docker image for your Operator.
  * Deploy the Operator to a Kubernetes cluster using `make deploy`.
  * Understand the necessary RBAC and deployment configurations.

**Challenges:**

  * Container image registry access and authentication.
  * Troubleshooting deployment issues in a cluster.

**Code Examples:**

1.  **Build and push Docker image:**
    Replace `your-registry/your-repo` with your Docker Hub or private registry path.

    ```bash
    export IMG=your-registry/your-repo/helloworld-operator:v0.1.0
    make docker-build
    make docker-push
    ```

2.  **Deploy to cluster:**
    Ensure `KUBECONFIG` points to your target cluster.

    ```bash
    make deploy IMG=${IMG}
    ```

3.  **Verify deployment:**

    ```bash
    kubectl get deployments -n helloworld-operator-system # Check operator deployment
    kubectl get pods -n helloworld-operator-system      # Check operator pod
    kubectl apply -f config/samples/webapp_v1_helloworld.yaml # Apply sample CR
    kubectl get helloworlds
    ```

-----

### Day 8: Cleanup and Further Exploration

**Learning Objectives:**

  * Clean up deployed resources.
  * Review best practices for Operator development.
  * Identify next steps for more complex Operators.

**Challenges:**

  * Understanding when and how to use Finalizers for graceful deletion.
  * Scalability and performance considerations for Operators.

**Code Examples:**

1.  **Delete the sample CR:**

    ```bash
    kubectl delete -f config/samples/webapp_v1_helloworld.yaml
    ```

2.  **Undeploy the Operator:**

    ```bash
    make undeploy
    ```

**Challenges and Best Practices:**

  * **Idempotency:** Your `Reconcile` function must be idempotent. Running it multiple times with the same desired state should always result in the same actual state without side effects.
  * **Error Handling and Requeues:** Differentiate between transient errors (requeue with exponential backoff) and permanent errors (do not requeue, log error).
  * **Owner References:** Properly set owner references for resources created by your Operator. This ensures Kubernetes garbage collection cleans up dependent resources when the Custom Resource is deleted.
  * **Status Updates:** Use the `/status` subresource for status updates. This allows permission separation between changing the `spec` and updating the `status`.
  * **Finalizers:** Implement finalizers for complex cleanup logic that needs to happen *before* a CR is truly deleted (e.g., deleting external cloud resources).
  * **RBAC:** Adhere to the principle of least privilege. Only grant your Operator the necessary permissions.
  * **Testing:** Write unit, integration, and end-to-end tests for your Operator.
  * **Logging and Metrics:** Implement robust logging and expose Prometheus metrics for observability.
  * **Scalability:** Consider how your Operator will perform with a large number of CRs or under heavy load. Use watches sparingly if not strictly necessary.
  * **Community Operators:** Leverage existing Operators for common applications if available, rather than building from scratch.
  * **Operator SDK vs. Kubebuilder:** While this tutorial focuses on Kubebuilder, Operator SDK is another excellent tool. Kubebuilder focuses more on the Go framework and `controller-runtime`, while Operator SDK provides additional tooling for Helm and Ansible-based operators and lifecycle management. Often, they can be used together.

-----

This 8-day tutorial provides a solid foundation for building Kubernetes Operators with Kubebuilder. Remember that building robust Operators for production environments requires deep understanding of Kubernetes, careful error handling, and thorough testing. Happy Operating\!