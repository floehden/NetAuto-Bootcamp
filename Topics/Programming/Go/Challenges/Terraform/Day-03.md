# **Day 3: Implementing CRUD Operations (Create & Read)**

## **Objective:** 
Implement the `Create` and `Read` operations for our resource.

## **Create Operation (`CreateContext`):**
1.  Retrieve input values from `d *schema.ResourceData`.
2.  Perform API call (simulated by adding to our `greetings` map).
3.  Set the resource `ID` (`d.SetId()`).
4.  Set computed values back to `d`.

## **Read Operation (`ReadContext`):**
1.  Retrieve the resource `ID` (`d.Id()`).
2.  Perform API call to fetch the resource (simulated by looking up in `greetings`).
3.  If the resource doesn't exist (e.g., deleted externally), call `d.SetId("")` to mark it for recreation.
4.  Set fetched values back to `d`.

## **Error Handling and Diagnostics (`diag.Diagnostics`):**
* How to return errors effectively for Terraform.
* `diag.Errorf`, `diag.FromErr`.

## **Challenges:**
* Understanding the flow of data between Terraform state and your provider.
* Correctly handling cases where resources might not exist (e.g., API errors, external deletion).
* Managing state in the `ResourceData` object.

## **Code Example (Day 3 - Update `resource_greeting_message.go`):**

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

func resourceGreetingMessageCreate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	message := d.Get("message").(string)
	author := d.Get("author").(string) // Will be "" if not provided

	if author == "" {
		author = "Anonymous" // Simulate API computing a default author
	}

	// Simulate API call to create greeting
	id := strconv.Itoa(nextID)
	nextID++
	greetings[id] = map[string]string{
		"message": message,
		"author":  author,
	}

	d.SetId(id)
	d.Set("message", message)
	d.Set("author", author) // Set the potentially computed author

	return resourceGreetingMessageRead(ctx, d, m) // Read to ensure state is consistent
}

func resourceGreetingMessageRead(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	var diags diag.Diagnostics

	greetingID := d.Id()
	greeting, ok := greetings[greetingID]
	if !ok {
		// Resource not found, mark for recreation
		d.SetId("")
		return diags
	}

	d.Set("message", greeting["message"])
	d.Set("author", greeting["author"])

	return diags
}

// Placeholder for Update and Delete (from Day 2, will be updated on Day 4)
func resourceGreetingMessageUpdate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	return diag.Errorf("Update not implemented yet")
}

func resourceGreetingMessageDelete(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	return diag.Errorf("Delete not implemented yet")
}

```
