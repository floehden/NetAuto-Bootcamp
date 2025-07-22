# **Day 1: Understanding Terraform Providers & Setup**

## **Objective:** 
Grasp the role of a Terraform provider and set up the development environment.

## **Introduction:**
* What is Terraform? Infrastructure as Code (IaC).
* What is a Terraform Provider? Bridge between Terraform configuration and cloud/service APIs.
* Why build a custom provider? Managing niche services, internal APIs, or extending existing functionality.
* Key components of a provider: Resources, Data Sources, Schemas.

## **Core Concepts:**
* **Resources:** Represent managed infrastructure objects (e.g., a virtual machine, a database). They have a lifecycle (create, read, update, delete).
* **Data Sources:** Allow fetching information from an external service without managing its lifecycle. Useful for lookup, filtering, or referencing existing resources.
* **Schema:** Defines the structure and types of arguments for resources and data sources.

## **Development Environment Setup:**
1.  **Go Workspace:** Create a new directory for your provider project.
2.  **`go mod init`:** Initialize a Go module.
3.  **Terraform Plugin SDK:** Explain its importance. It provides the necessary interfaces and helper functions for provider development.
      * `go get github.com/hashicorp/terraform-plugin-sdk/v2`

## **Simple Use Case Introduction: A "Hello World" Service Provider**

We'll build a very basic provider that manages a "Greeting" resource. This "Greeting" will simply store a message and its author. For simplicity, our "API" will be an in-memory map within the provider itself, simulating an external service.
## **Challenges:**

  * Understanding the abstraction layers of Terraform and how providers fit in.
  * Initial Go environment setup can be tricky for newcomers.

## **Code Example (Day 1 - `main.go`):**

```go
package main

import (
	"context"
	"fmt"

	"github.com/hashicorp/terraform-plugin-sdk/v2/plugin"
)

func main() {
	fmt.Print("Starting Terraform provider...")
	plugin.Serve(&plugin.ServeOpts{
		ProviderFunc: Provider,
	})
	fmt.Print("Terraform provider stopped.")
}

// Provider returns a terraform.Provider.
func Provider() *plugin.Provider {
	return &plugin.Provider{
		Schema: map[string]*plugin.Schema{}, // Empty schema for now
		ResourcesMap: map[string]*plugin.Resource{
			"greeting_message": resourceGreetingMessage(), // Placeholder for our resource
		},
		DataSourcesMap: map[string]*plugin.DataSource{},
	}
}

// Placeholder for our resource function (will be implemented on Day 2)
func resourceGreetingMessage() *plugin.Resource {
	return &plugin.Resource{} // Will be filled in later
}
```
