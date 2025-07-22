
# Day 4: Implementing the Controller Logic (Part 2 - Services and ConfigMaps)

## **Learning Objectives:**
* Create a Kubernetes Service to expose the Nginx Deployment.
* Create a ConfigMap to serve the custom message.
* Understand owner references for garbage collection.

## **Challenges:**
* Managing multiple dependent resources.
* Ensuring correct ConfigMap content.

## **Code Examples:**

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
