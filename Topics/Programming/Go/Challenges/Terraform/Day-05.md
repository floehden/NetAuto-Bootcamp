# **Day 5: Data Sources & Advanced Schema Concepts**

## **Objective:** 
Implement a data source and explore more advanced schema features.

## **Data Sources:**

* Similar to resources, but only have a `ReadContext` function.
* Used to fetch existing data without managing its lifecycle.
* Useful for looking up resources by name, filtering, etc.

## **Implementing a "Greeting" Data Source:**

* Allow users to look up a greeting by its ID or message.

## **Advanced Schema Concepts:**

* **`ConflictsWith`:** Prevent specific attributes from being used together.
* **`Sensitive`:** Mark attributes that should not be displayed in logs (e.g., passwords).
* **`Computed: true` with `Optional: true`:** When an attribute can be provided by the user or computed by the API.
* **`ForceNew: true`:** If changing an attribute requires recreating the resource.
* **`TypeSet` / `TypeList` / `TypeMap`:** Handling collections and nested blocks.

## **Challenges:**

* Designing efficient lookup mechanisms for data sources.
* Understanding the nuances of advanced schema flags and their impact on resource behavior.

## **Code Example (Day 5 - Add `data_source_greeting_message.go`):**

```go
package main

import (
	"context"
	"fmt"

	"github.com/hashicorp/terraform-plugin-sdk/v2/diag"
	"github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
)

func dataSourceGreetingMessage() *schema.Resource {
	return &schema.Resource{
		ReadContext: dataSourceGreetingMessageRead,
		Schema: map[string]*schema.Schema{
			"id": {
				Type:        schema.TypeString,
				Optional:    true,
				Computed:    true,
				Description: "The ID of the greeting.",
			},
			"message": {
				Type:        schema.TypeString,
				Optional:    true,
				Computed:    true,
				Description: "The greeting message.",
			},
			"author": {
				Type:        schema.TypeString,
				Computed:    true,
				Description: "The author of the greeting.",
			},
		},
	}
}

func dataSourceGreetingMessageRead(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	var diags diag.Diagnostics

	id := d.Get("id").(string)
	messageFilter := d.Get("message").(string) // Optional filter by message

	var foundGreetingID string
	var foundGreeting map[string]string

	if id != "" {
		// Prioritize lookup by ID
		greeting, ok := greetings[id] // `greetings` from resource file
		if ok {
			foundGreetingID = id
			foundGreeting = greeting
		}
	} else if messageFilter != "" {
		// If no ID, try to find by message
		for greetingID, greeting := range greetings {
			if greeting["message"] == messageFilter {
				if foundGreetingID != "" {
					// Found more than one, error
					return diag.Errorf("Multiple greetings found with message '%s'. Please specify ID.", messageFilter)
				}
				foundGreetingID = greetingID
				foundGreeting = greeting
			}
		}
	}

	if foundGreetingID == "" {
		return diag.Errorf("No greeting found with the given criteria.")
	}

	d.SetId(foundGreetingID)
	d.Set("message", foundGreeting["message"])
	d.Set("author", foundGreeting["author"])

	return diags
}

```

**Update `main.go` to include the data source:**

```go
package main

import (
	"context"
	"fmt"

	"github.com/hashicorp/terraform-plugin-sdk/v2/plugin"
	"github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema" // Add this import
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
		Schema: map[string]*plugin.Schema{},
		ResourcesMap: map[string]*plugin.Resource{
			"greeting_message": resourceGreetingMessage(),
		},
		DataSourcesMap: map[string]*plugin.DataSource{
			"greeting_message": dataSourceGreetingMessage(), // Register the data source
		},
	}
}

// resourceGreetingMessage (from Day 2)
func resourceGreetingMessage() *schema.Resource {
    // ... (Your resource schema and CRUD functions)
}
```

**Terraform Configuration Example (`main.tf` with data source):**

```terraform
# ... (existing provider and resource blocks) ...

data "greeting_message" "specific_hello" {
  id = greeting_message.hello.id
}

data "greeting_message" "welcome_lookup" {
  message = "Welcome to the custom provider tutorial."
}

output "data_hello_message" {
  value = data.greeting_message.specific_hello.message
}

output "data_welcome_author" {
  value = data.greeting_message.welcome_lookup.author
}
```

