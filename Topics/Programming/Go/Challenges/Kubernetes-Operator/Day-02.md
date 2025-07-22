# Day 2: Defining the Custom Resource (CRD)

## **Learning Objectives:**

* Generate a new API (CRD and its Go types).
* Understand the `Spec` and `Status` of a custom resource.
* Add fields to the Custom Resource `Spec`.

## **Challenges:**

* Designing an effective and stable API for your custom resource.
* Understanding Go struct tags (`json`, `omitempty`).

## **Code Examples:**

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

