# Day 3: Implementing the Controller Logic (Part 1 - Reconciliation)

## **Learning Objectives:**

* Understand the `Reconcile` function.
* Fetch the Custom Resource.
* Create a Kubernetes Deployment based on the CR's `Spec`.

## **Challenges:**

* Idempotency of the `Reconcile` loop.
* Error handling and requeueing.

## **Code Examples:**
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

