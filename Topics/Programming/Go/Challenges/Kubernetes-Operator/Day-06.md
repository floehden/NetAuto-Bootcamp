# Day 6: Advanced Topics: Webhooks (Optional but Recommended)

## **Learning Objectives:**

* Understand Admission Webhooks (Validating and Mutating).
* Generate a webhook using Kubebuilder.
* Implement basic validation for your Custom Resource.

## **Challenges:**

* Understanding Kubernetes admission control.
* Debugging webhook interactions.

## **Code Examples:**

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

