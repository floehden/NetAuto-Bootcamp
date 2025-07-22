# **Day 6: Provider Configuration, State Management & Best Practices**

## **Objective:** 
Understand provider-level configuration, advanced state management, and best practices.

## **Provider Configuration:**

  * Passing configuration to the provider (e.g., API keys, endpoints).
  * Using a `ConfigureContextFunc` in `plugin.Provider`.
  * Storing client/API connection details in the `meta` interface.

## **Advanced State Management:**

  * **Immutability:** Designing resources to be immutable where appropriate (use `ForceNew`).
  * **Import:** Enabling users to import existing resources into Terraform state.
  * **Diff Suppression:** When changes to an attribute shouldn't trigger an update plan.

#ä **Best Practices:**

  * **Error Handling:** Provide clear and actionable error messages.
  * **Logging:** Use Go's `log` package or a structured logger for debugging.
  * **Context:** Pass `context.Context` for cancellation and timeouts.
  * **Idempotency:** Ensure operations can be run multiple times without side effects.
  * **Testing:** Unit tests, integration tests.
  * **Documentation:** Clear examples and attribute descriptions.
  * **Version Control:** Use Git.
  * **Release Management:** How to tag and release providers.

## **Challenges:**

  * Properly securing sensitive information passed to the provider.
  * Handling complex real-world API interactions (retries, rate limiting, pagination).
  * Writing comprehensive tests.

## **Code Example (Day 6 - Update `main.go` and `resource_greeting_message.go`):**

**Update `main.go` for Provider Configuration:**

```go
package main

import (
	"context"
	"fmt"

	"github.com/hashicorp/terraform-plugin-sdk/v2/diag" // Added import
	"github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
	"github.com/hashicorp/terraform-plugin-sdk/v2/plugin"
)

// Define a client struct to hold API configuration
type apiClient struct {
	APIBaseURL string
	APIKey     string
	// ... potentially other client configurations
}

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
		Schema: map[string]*schema.Schema{
			"api_base_url": {
				Type:        schema.TypeString,
				Optional:    true,
				Default:     "http://localhost:8080/api", // Default for our simulated API
				Description: "The base URL for the greeting API.",
			},
			"api_key": {
				Type:        schema.TypeString,
				Optional:    true,
				Sensitive:   true, // Mark as sensitive
				Description: "The API key for authentication.",
			},
		},
		ResourcesMap: map[string]*schema.Resource{
			"greeting_message": resourceGreetingMessage(),
		},
		DataSourcesMap: map[string]*schema.DataSource{
			"greeting_message": dataSourceGreetingMessage(),
		},
		ConfigureContextFunc: providerConfigure,
	}
}

func providerConfigure(ctx context.Context, d *schema.ResourceData) (interface{}, diag.Diagnostics) {
	apiBaseURL := d.Get("api_base_url").(string)
	apiKey := d.Get("api_key").(string)

	// In a real provider, you would initialize your API client here
	// For our simple example, we'll just store the values
	client := &apiClient{
		APIBaseURL: apiBaseURL,
		APIKey:     apiKey,
	}

	return client, nil // Return the client object as the meta interface
}

// resourceGreetingMessage (from Day 2)
func resourceGreetingMessage() *schema.Resource {
    // ... (Your resource schema and CRUD functions)
}

// dataSourceGreetingMessage (from Day 5)
func dataSourceGreetingMessage() *schema.Resource {
    // ... (Your data source schema and Read function)
}

```

**Modify Resource/Data Source functions to use the `meta` interface:**

```go
package main

import (
	"context"
	"fmt"
	"strconv"
	"time"

	"github.com/hashicorp/terraform-plugin-sdk/v2/diag"
	"github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
)

// In-memory "database" to simulate our API (from Day 2)
var greetings = make(map[string]map[string]string)
var nextID = 1

// resourceGreetingMessage (from Day 2)
func resourceGreetingMessage() *schema.Resource {
	return &schema.Resource{
		CreateContext: resourceGreetingMessageCreate,
		ReadContext:   resourceGreetingMessageRead,
		UpdateContext: resourceGreetingMessageUpdate,
		DeleteContext: resourceGreetingMessageDelete,
		Schema: map[string]*schema.Schema{
			"message": {
				Type:        schema.TypeString,
				Required:    true,
				Description: "The greeting message.",
			},
			"author": {
				Type:        schema.TypeString,
				Optional:    true,
				Computed:    true,
				Description: "The author of the greeting.",
			},
			"id": {
				Type:        schema.TypeString,
				Computed:    true,
				Description: "Unique ID of the greeting.",
			},
		},
		Importer: &schema.ResourceImporter{ // Add importer functionality
			StateContext: schema.NoopContext, // For simple cases, NoopContext is fine
		},
	}
}

// Update CRUD functions to use the 'meta' interface (m interface{})
func resourceGreetingMessageCreate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	// client := m.(*apiClient) // Cast the meta interface to our client type
	// fmt.Printf("Using API Base URL: %s, API Key: %s\n", client.APIBaseURL, client.APIKey)

	message := d.Get("message").(string)
	author := d.Get("author").(string)

	if author == "" {
		author = "Anonymous"
	}

	id := strconv.Itoa(nextID)
	nextID++
	greetings[id] = map[string]string{
		"message": message,
		"author":  author,
	}

	d.SetId(id)
	d.Set("message", message)
	d.Set("author", author)

	return resourceGreetingMessageRead(ctx, d, m)
}

func resourceGreetingMessageRead(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	// client := m.(*apiClient)
	// fmt.Printf("Using API Base URL: %s, API Key: %s\n", client.APIBaseURL, client.APIKey)
	// ... (rest of the read function)
	var diags diag.Diagnostics

	greetingID := d.Id()
	greeting, ok := greetings[greetingID]
	if !ok {
		d.SetId("")
		return diags
	}

	d.Set("message", greeting["message"])
	d.Set("author", greeting["author"])

	return diags
}

func resourceGreetingMessageUpdate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	// client := m.(*apiClient)
	// fmt.Printf("Using API Base URL: %s, API Key: %s\n", client.APIBaseURL, client.APIKey)
	// ... (rest of the update function)
	var diags diag.Diagnostics

	greetingID := d.Id()
	greeting, ok := greetings[greetingID]
	if !ok {
		diags = append(diags, diag.Errorf("Greeting with ID %s not found for update", greetingID)...)
		return diags
	}

	if d.HasChange("message") {
		message := d.Get("message").(string)
		greeting["message"] = message
		d.Set("message", message)
	}

	if d.HasChange("author") {
		author := d.Get("author").(string)
		greeting["author"] = author
		d.Set("author", author)
	}

	greetings[greetingID] = greeting

	return resourceGreetingMessageRead(ctx, d, m)
}

func resourceGreetingMessageDelete(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	// client := m.(*apiClient)
	// fmt.Printf("Using API Base URL: %s, API Key: %s\n", client.APIBaseURL, client.APIKey)
	// ... (rest of the delete function)
	var diags diag.Diagnostics

	greetingID := d.Id()
	_, ok := greetings[greetingID]
	if !ok {
		return diags
	}

	delete(greetings, greetingID)
	d.SetId("")

	return diags
}

// dataSourceGreetingMessage (from Day 5)
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
	// client := m.(*apiClient)
	// fmt.Printf("Using API Base URL: %s, API Key: %s\n", client.APIBaseURL, client.APIKey)
	// ... (rest of the read function)
	var diags diag.Diagnostics

	id := d.Get("id").(string)
	messageFilter := d.Get("message").(string)

	var foundGreetingID string
	var foundGreeting map[string]string

	if id != "" {
		greeting, ok := greetings[id]
		if ok {
			foundGreetingID = id
			foundGreeting = greeting
		}
	} else if messageFilter != "" {
		for greetingID, greeting := range greetings {
			if greeting["message"] == messageFilter {
				if foundGreetingID != "" {
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

**Terraform Configuration Example (`main.tf` with provider config):**

```terraform
terraform {
  required_providers {
    greeting = {
      source  = "local/greeting"
      version = "1.0.0"
    }
  }
}

provider "greeting" {
  api_base_url = "http://my-greeting-api.com"
  api_key      = "supersecretkey" # In a real scenario, use TF_VAR_api_key or similar for sensitive values
}

resource "greeting_message" "hello" {
  message = "Hello, Terraform!"
  author  = "The Provider Builder"
}

output "hello_id" {
  value = greeting_message.hello.id
}
```
