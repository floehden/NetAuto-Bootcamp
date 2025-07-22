# **Day 4: Implementing CRUD Operations (Update & Delete) & Testing**

## **Objective:** 
Complete the CRUD operations and learn how to test the provider locally.

## **Update Operation (`UpdateContext`):**
1.  Check for changes using `d.HasChange("attribute_name")`.
2.  Retrieve old and new values.
3.  Perform API call to update the resource.
4.  Call `ReadContext` to refresh the state after update.

## **Delete Operation (`DeleteContext`):**
1.  Retrieve the resource `ID`.
2.  Perform API call to delete the resource.
3.  Call `d.SetId("")` to clear the ID, indicating the resource is gone.

## **Local Testing:**

* **Building the provider:** `go build -o terraform-provider-greeting`
* **Placing the provider:** Put the executable in Terraform's plugin directory (e.g., `~/.terraform.d/plugins/local/greeting/1.0.0/linux_amd64/`).
* **`terraform init`:** To discover the local provider.
* **`terraform plan` / `terraform apply` / `terraform destroy`:** To test the full lifecycle.

## **Challenges:**
* Idempotency of update and delete operations.
* Handling partial updates or complex attribute changes.
* Setting up local testing environment correctly.

## **Code Example (Day 4 - Update `resource_greeting_message.go`):**

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

// In-memory "database" to simulate our API (from Day 2/3)
var greetings = make(map[string]map[string]string)
var nextID = 1

func resourceGreetingMessage() *schema.Resource {
	// ... (schema definition from Day 2 remains the same)
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
	}
}

// resourceGreetingMessageCreate (from Day 3)
func resourceGreetingMessageCreate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
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

// resourceGreetingMessageRead (from Day 3)
func resourceGreetingMessageRead(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
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

	greetings[greetingID] = greeting // Update in our "database"

	return resourceGreetingMessageRead(ctx, d, m) // Read to ensure state is consistent
}

func resourceGreetingMessageDelete(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	var diags diag.Diagnostics

	greetingID := d.Id()
	_, ok := greetings[greetingID]
	if !ok {
		// Resource already deleted or never existed, nothing to do
		return diags
	}

	delete(greetings, greetingID) // Simulate API deletion
	d.SetId("")                   // Mark as deleted in Terraform state

	return diags
}

```

## **Terraform Configuration Example (`main.tf`):**

```terraform
terraform {
  required_providers {
    greeting = {
      source  = "local/greeting" # Or your actual source if published
      version = "1.0.0"          # Or your desired version
    }
  }
}

provider "greeting" {}

resource "greeting_message" "hello" {
  message = "Hello, Terraform!"
  author  = "The Provider Builder"
}

resource "greeting_message" "welcome" {
  message = "Welcome to the custom provider tutorial."
}

output "hello_id" {
  value = greeting_message.hello.id
}

output "welcome_message" {
  value = greeting_message.welcome.message
}
```

