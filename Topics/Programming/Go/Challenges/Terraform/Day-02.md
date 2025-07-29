# **Day 2: Defining Resource Schema**

## **Objective:** 
Learn to define a resource's schema using Go structs and `tfsdk.Attribute` tags.

## **Resource Definition:**
  * A resource is represented by a struct that implements `framework.Resource`.
  * You'll define a `Model` struct that represents the Terraform state and configuration for your resource. This struct will use `types.String`, `types.Int64`, etc.
  * The `Schema` method of your resource type will define the attributes.

## **Schema Definition with `tfsdk.Attribute`:**
  * Attributes are defined directly within your `Model` struct using `tfsdk.Attribute` tags.
  * `tfsdk.StringAttribute`, `tfsdk.Int64Attribute`, `tfsdk.BoolAttribute`, `tfsdk.ListAttribute`, `tfsdk.SetAttribute`, `tfsdk.MapAttribute`.
  * Properties like `Required`, `Optional`, `Computed`, `Description`, `Sensitive`, `RequiresReplace` (similar to `ForceNew` in SDKv2) are set directly on the attribute.

## **Implementing the "Greeting" Resource Schema:**

  * `ID` (Computed, String): Unique identifier.
  * `Message` (Required, String): The greeting message.
  * `Author` (Optional, Computed, String): Who created the greeting.

## **Challenges:**

  * Getting used to the `tfsdk.Attribute` syntax and the different `types` (e.g., `types.String` vs `string`).
  * Understanding how `tfsdk.State.Get` and `tfsdk.Plan.Get` work with the `Model` struct.

## **Code Example (Day 2 - `resource_greeting_message.go`):**

```go
package main

import (
	"context"

	"github.com/hashicorp/terraform-plugin-framework/diag" // For diagnostics
	"github.com/hashicorp/terraform-plugin-framework/tfsdk"
	"github.com/hashicorp/terraform-plugin-framework/types" // For framework types
)

// Ensure the implementation satisfies the tfsdk.ResourceTypeWith
// and tfsdk.Resource interfaces.
var _ tfsdk.ResourceTypeWith
var _ tfsdk.Resource = &greetingResource{}

// greetingResourceType defines the resource type.
type greetingResourceType struct{}

// GetSchema defines the schema for the greeting_message resource.
func (r greetingResourceType) GetSchema(ctx context.Context) (tfsdk.Schema, diag.Diagnostics) {
	return tfsdk.Schema{
		// This description is visible to users in Terraform documentation.
		Description: "Manages a greeting message.",
		Attributes: map[string]tfsdk.Attribute{
			"id": {
				// ID is a computed attribute, meaning it's set by the provider after creation.
				Description: "Unique ID of the greeting.",
				Type:        types.StringType,
				Computed:    true,
			},
			"message": {
				// Message is a required attribute, meaning users must provide it.
				Description: "The greeting message.",
				Type:        types.StringType,
				Required:    true,
			},
			"author": {
				// Author is optional, but can also be computed (e.g., if the API sets a default).
				Description: "The author of the greeting.",
				Type:        types.StringType,
				Optional:    true,
				Computed:    true,
			},
		},
	}, nil // No diagnostics on schema definition errors for now
}

// NewResource creates a new resource instance.
// This function is part of the tfsdk.ResourceTypeWith interface.
func (r greetingResourceType) NewResource(ctx context.Context, p tfsdk.Provider) (tfsdk.Resource, diag.Diagnostics) {
	// We'll pass the provider's configured client to the resource instance.
	// This allows the resource to make API calls using the configured client.
	return &greetingResource{
		provider: (*p.(*greetingProvider)), // Cast the provider interface to our specific provider type
	}, nil
}

// greetingResource implements the tfsdk.Resource interface.
type greetingResource struct {
	provider greetingProvider // Store the provider instance to access its client
}

// GreetingModel defines the Go model for the greeting_message resource.
// This struct maps directly to the schema attributes.
type GreetingModel struct {
	ID      types.String `tfsdk:"id"`
	Message types.String `tfsdk:"message"`
	Author  types.String `tfsdk:"author"`
}

// Placeholder CRUD functions (will be implemented on Day 3 & 4)
func (r *greetingResource) Create(ctx context.Context, req tfsdk.CreateResourceRequest, resp *tfsdk.CreateResourceResponse) {
	resp.Diagnostics.AddError("Create not implemented", "The Create function for greeting_message is not yet implemented.")
}

func (r *greetingResource) Read(ctx context.Context, req tfsdk.ReadResourceRequest, resp *tfsdk.ReadResourceResponse) {
	resp.Diagnostics.AddError("Read not implemented", "The Read function for greeting_message is not yet implemented.")
}

func (r *greetingResource) Update(ctx context.Context, req tfsdk.UpdateResourceRequest, resp *tfsdk.UpdateResourceResponse) {
	resp.Diagnostics.AddError("Update not implemented", "The Update function for greeting_message is not yet implemented.")
}

func (r *greetingResource) Delete(ctx context.Context, req tfsdk.DeleteResourceRequest, resp *tfsdk.DeleteResourceResponse) {
	resp.Diagnostics.AddError("Delete not implemented", "The Delete function for greeting_message is not yet implemented.")
}

// ImportState is used to import existing resources into Terraform state.
func (r *greetingResource) ImportState(ctx context.Context, req tfsdk.ImportResourceStateRequest, resp *tfsdk.ImportResourceStateResponse) {
	// For simple resources, setting the ID is often enough.
	tfsdk.State{}.SetAttribute(ctx, resp.State, "id", req.ID)
}

```