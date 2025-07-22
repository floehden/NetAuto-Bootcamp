# **Day 2: Defining Resources & Schema**

## **Objective:** 
Learn to define a resource, its schema, and basic CRUD operations.

## **Resource Definition:**
* A resource is represented by a `plugin.Resource` struct.
* It contains methods for `Create`, `Read`, `Update`, and `Delete` operations.
* Each operation receives a `*schema.ResourceData` object, which holds the current state of the resource in Terraform.

## **Schema Definition:**
* The `Schema` map within a resource defines the arguments (attributes) that users can configure.
* Each argument has a `Type`, `Required`/`Optional`, `Computed`, `Description`, etc.
* **Types:** `schema.TypeString`, `schema.TypeInt`, `schema.TypeBool`, `schema.TypeList`, `schema.TypeSet`, `schema.TypeMap`.

## **Implementing the "Greeting" Resource Schema:**

  * `message` (TypeString, Required, Computed): The greeting message itself.
  * `author` (TypeString, Optional, Computed): Who created the greeting.
  * `id` (TypeString, Computed): A unique identifier for the greeting (simulating an API ID).

## **Challenges:**

  * Understanding the different schema types and their implications.
  * Distinguishing between `Required`, `Optional`, and `Computed` attributes.
  * Handling computed values that might come from the API after creation.

## **Code Example (Day 2 - `resource_greeting_message.go`):**

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

// In-memory "database" to simulate our API
var greetings = make(map[string]map[string]string)
var nextID = 1

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
				Computed:    true, // If not provided, it will be computed (e.g., by the API)
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

// Placeholder CRUD functions (will be implemented on Day 3)
func resourceGreetingMessageCreate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	return diag.Errorf("Create not implemented yet")
}

func resourceGreetingMessageRead(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	return diag.Errorf("Read not implemented yet")
}

func resourceGreetingMessageUpdate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	return diag.Errorf("Update not implemented yet")
}

func resourceGreetingMessageDelete(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	return diag.Errorf("Delete not implemented yet")
}

```

