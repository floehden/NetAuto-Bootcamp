# **Day 4: Implementing CRUD Operations (Update & Delete) & Testing**

## **Objective:** 
Complete the CRUD operations and learn how to test the provider locally with the framework.

## **Update Operation (`Update` method):**

1.  Retrieve the current state from `req.State.Get()` and the desired plan from `req.Plan.Get()`.
2.  Compare the state and plan to identify changes.
3.  Call your API client to update the resource.
4.  Set the updated state using `resp.State.Set()`.

## **Delete Operation (`Delete` method):**

1.  Retrieve the current state from `req.State.Get()`.
2.  Call your API client to delete the resource.
3.  Remove the resource from Terraform state using `resp.State.RemoveResource()`.

## **Local Testing with Plugin Framework:**

  * The `main.go` using `providerserver.Serve` automatically handles the gRPC server for the provider.
  * **Building the provider:** `go build -o terraform-provider-greeting`
  * **Placing the provider:** The framework uses a different discovery mechanism. You'll typically place the binary in `~/.terraform.d/plugins/local/greeting/<version>/<os_arch>/` (e.g., `~/.terraform.d/plugins/local/greeting/1.0.0/linux_amd64/`).
  * **`terraform init`:** To discover the local provider.
  * **`terraform plan` / `terraform apply` / `terraform destroy`:** To test the full lifecycle.

## **Challenges:**

  * Identifying changed attributes correctly for updates.
  * Ensuring idempotency of update and delete operations.
  * Setting up the local testing environment correctly.

## **Code Example (Day 4 - Update `resource_greeting_message.go`):**

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/hashicorp/terraform-plugin-framework/diag"
	"github.com/hashicorp/terraform-plugin-framework/tfsdk"
	"github.com/hashicorp/terraform-plugin-framework/types"
)

// Ensure the implementation satisfies the tfsdk.ResourceTypeWith
// and tfsdk.Resource interfaces.
var _ tfsdk.ResourceTypeWith
var _ tfsdk.Resource = &greetingResource{}

// greetingResourceType (from Day 2)
type greetingResourceType struct{}

// GetSchema (from Day 2)
func (r greetingResourceType) GetSchema(ctx context.Context) (tfsdk.Schema, diag.Diagnostics) {
	return tfsdk.Schema{
		Description: "Manages a greeting message.",
		Attributes: map[string]tfsdk.Attribute{
			"id": {
				Description: "Unique ID of the greeting.",
				Type:        types.StringType,
				Computed:    true,
			},
			"message": {
				Description: "The greeting message.",
				Type:        types.StringType,
				Required:    true,
			},
			"author": {
				Description: "The author of the greeting.",
				Type:        types.StringType,
				Optional:    true,
				Computed:    true,
			},
		},
	}, nil
}

// NewResource (from Day 2)
func (r greetingResourceType) NewResource(ctx context.Context, p tfsdk.Provider) (tfsdk.Resource, diag.Diagnostics) {
	provider, ok := p.(*greetingProvider)
	if !ok {
		return nil, diag.Diagnostics{
			diag.NewErrorDiagnostic("Unexpected Provider Type", "Expected *greetingProvider."),
		}
	}
	return &greetingResource{
		client: provider.client,
	}, nil
}

// greetingResource (from Day 2)
type greetingResource struct {
	client *apiClient
}

// GreetingModel (from Day 2)
type GreetingModel struct {
	ID      types.String `tfsdk:"id"`
	Message types.String `tfsdk:"message"`
	Author  types.String `tfsdk:"author"`
}

// Create (from Day 3)
func (r *greetingResource) Create(ctx context.Context, req tfsdk.CreateResourceRequest, resp *tfsdk.CreateResourceResponse) {
	var plan GreetingModel
	resp.Diagnostics.Append(req.Plan.Get(ctx, &plan)...)
	if resp.Diagnostics.HasError() {
		return
	}

	message := plan.Message.ValueString()
	author := plan.Author.ValueString()

	createdGreeting, err := r.client.CreateGreeting(ctx, message, author)
	if err != nil {
		resp.Diagnostics.AddError("Error creating greeting", fmt.Sprintf("Could not create greeting: %s", err.Error()))
		return
	}

	plan.ID = types.StringValue(createdGreeting.ID)
	plan.Message = types.StringValue(createdGreeting.Message)
	plan.Author = types.StringValue(createdGreeting.Author)

	log.Printf("[INFO] Created greeting with ID: %s", createdGreeting.ID)
	resp.Diagnostics.Append(resp.State.Set(ctx, &plan)...)
}

// Read (from Day 3)
func (r *greetingResource) Read(ctx context.Context, req tfsdk.ReadResourceRequest, resp *tfsdk.ReadResourceResponse) {
	var state GreetingModel
	resp.Diagnostics.Append(req.State.Get(ctx, &state)...)
	if resp.Diagnostics.HasError() {
		return
	}

	greetingID := state.ID.ValueString()
	if greetingID == "" {
		resp.State.RemoveResource(ctx)
		log.Printf("[WARN] Attempted to read resource with empty ID, removing from state.")
		return
	}

	greeting, err := r.client.GetGreetingByID(ctx, greetingID)
	if err != nil {
		resp.Diagnostics.AddError("Error reading greeting", fmt.Sprintf("Could not read greeting with ID %s: %s", greetingID, err.Error()))
		return
	}

	if greeting == nil {
		resp.State.RemoveResource(ctx)
		log.Printf("[INFO] Greeting with ID %s not found in API, removing from state.", greetingID)
		return
	}

	state.ID = types.StringValue(greeting.ID)
	state.Message = types.StringValue(greeting.Message)
	state.Author = types.StringValue(greeting.Author)

	log.Printf("[INFO] Read greeting with ID: %s", greetingID)
	resp.Diagnostics.Append(resp.State.Set(ctx, &state)...)
}

// Update handles resource updates.
func (r *greetingResource) Update(ctx context.Context, req tfsdk.UpdateResourceRequest, resp *tfsdk.UpdateResourceResponse) {
	var plan, state GreetingModel

	// Read the plan (desired state) and current state into Go models.
	resp.Diagnostics.Append(req.Plan.Get(ctx, &plan)...)
	resp.Diagnostics.Append(req.State.Get(ctx, &state)...)
	if resp.Diagnostics.HasError() {
		return
	}

	greetingID := state.ID.ValueString()
	if greetingID == "" {
		resp.Diagnostics.AddError("Error updating greeting", "Resource ID is missing from state.")
		return
	}

	// Extract values from the plan.
	newMessage := plan.Message.ValueString()
	newAuthor := plan.Author.ValueString()

	// Call the API client to update the greeting.
	// Our simple API's UpdateGreeting takes all fields, it's up to the API to handle partial updates.
	updatedGreeting, err := r.client.UpdateGreeting(ctx, greetingID, newMessage, newAuthor)
	if err != nil {
		resp.Diagnostics.AddError("Error updating greeting", fmt.Sprintf("Could not update greeting with ID %s: %s", greetingID, err.Error()))
		return
	}

	// Update the state based on the API response.
	plan.ID = types.StringValue(updatedGreeting.ID) // ID should remain the same
	plan.Message = types.StringValue(updatedGreeting.Message)
	plan.Author = types.StringValue(updatedGreeting.Author)

	log.Printf("[INFO] Updated greeting with ID: %s", greetingID)

	// Set the updated state for the resource.
	resp.Diagnostics.Append(resp.State.Set(ctx, &plan)...)
}

// Delete handles resource deletion.
func (r *greetingResource) Delete(ctx context.Context, req tfsdk.DeleteResourceRequest, resp *tfsdk.DeleteResourceResponse) {
	var state GreetingModel

	// Read the current state into our Go model.
	resp.Diagnostics.Append(req.State.Get(ctx, &state)...)
	if resp.Diagnostics.HasError() {
		return
	}

	greetingID := state.ID.ValueString()
	if greetingID == "" {
		log.Printf("[WARN] Attempted to delete resource with empty ID, assuming already deleted.")
		return // Nothing to delete if ID is empty
	}

	// Call the API client to delete the greeting.
	err := r.client.DeleteGreeting(ctx, greetingID)
	if err != nil {
		resp.Diagnostics.AddError("Error deleting greeting", fmt.Sprintf("Could not delete greeting with ID %s: %s", greetingID, err.Error()))
		return
	}

	// Remove the resource from Terraform state.
	resp.State.RemoveResource(ctx)
	log.Printf("[INFO] Deleted greeting with ID: %s", greetingID)
}

// ImportState (from Day 2)
func (r *greetingResource) ImportState(ctx context.Context, req tfsdk.ImportResourceStateRequest, resp *tfsdk.ImportResourceStateResponse) {
	tfsdk.State{}.SetAttribute(ctx, resp.State, "id", req.ID)
}

```

**Terraform Configuration Example (`main.tf`):**

```go
terraform {
  required_providers {
    greeting = {
      # The source matches the address in main.go: providerserver.ServeOpts.Address
      source  = "local/greeting"
      version = "1.0.0" # You can define a version in your provider.go later
    }
  }
}

provider "greeting" {
  base_url = "http://localhost:8080" # This should match where your Go API is running
  # api_key = "your_secret_key" # Uncomment if your API uses it, and protect this properly!
}

resource "greeting_message" "hello" {
  message = "Hello from Terraform Framework!"
  author  = "Framework Builder"
}

resource "greeting_message" "welcome" {
  message = "Welcome to the custom provider tutorial with Framework."
}

output "hello_id" {
  value = greeting_message.hello.id
}

output "welcome_message" {
  value = greeting_message.welcome.message
}
```

