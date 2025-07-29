# **Day 6: Provider Configuration, Advanced Concepts & Best Practices**

## **Objective:** 
Understand provider-level configuration, advanced concepts like `ImportState`, and best practices for the Plugin Framework.

## **Provider Configuration (`framework.Provider`):**

  * Define a `ProviderModel` struct for provider-level configuration (e.g., `base_url`, `api_key`).
  * Implement the `GetSchema` method for the provider to define its configurable attributes.
  * Implement the `Configure` method to read provider configuration and initialize your API client. This client will then be passed to your resources and data sources.

## **Advanced Concepts:**

  * **`ImportState`:** Already partially implemented on Day 2. It allows users to import existing resources into Terraform state using their ID.
  * **`RequiresReplace`:** This attribute property in the framework is similar to `ForceNew` in SDKv2. If an attribute with `RequiresReplace: true` changes, Terraform will plan to destroy and recreate the resource.
  * **`PlanModifiers` / `StateModifiers`:** For more advanced scenarios where you need to modify the plan or state directly (e.g., for complex computed values or diff suppression). (Beyond this basic tutorial, but good to know).

## **Best Practices:**

  * **Structured Logging:** Use Go's `log` package or a structured logger for debugging.
  * **Context:** Pass `context.Context` throughout your API calls for cancellation and timeouts.
  * **Idempotency:** Ensure API operations are idempotent.
  * **Testing:** Unit tests for your API client, acceptance tests for your provider.
  * **Documentation:** Clear examples and attribute descriptions in your `GetSchema` methods.
  * **Version Control:** Use Git.
  * **Release Management:** How to tag and release providers to the Terraform Registry.

## **Challenges:**

  * Correctly setting up the provider's `Configure` method to pass the API client to resources/data sources.
  * Understanding when to use `RequiresReplace` versus simple updates.

## **Code Example (Day 6 - Update `main.go`):**

```go
package main

import (
	"context"
	"fmt"
	"log"
	"net/http"
	"time"

	"github.com/hashicorp/terraform-plugin-framework/diag"
	"github.com/hashicorp/terraform-plugin-framework/providerserver"
	"github.com/hashicorp/terraform-plugin-framework/tfsdk"
	"github.com/hashicorp/terraform-plugin-framework/types"
)

// main function is the entry point for the provider.
func main() {
	err := providerserver.Serve(context.Background(), New, providerserver.ServeOpts{
		// This address is important! It defines how Terraform finds your provider.
		// For local testing, "local/greeting" is common.
		Address: "registry.terraform.io/local/greeting",
	})

	if err != nil {
		log.Fatal(err.Error())
	}
}

// New is a function that returns a new instance of our provider.
// It's required by providerserver.Serve.
func New() tfsdk.Provider {
	return &greetingProvider{}
}

// greetingProvider implements the tfsdk.Provider interface.
// It holds the API client that will be used by resources and data sources.
type greetingProvider struct {
	client *apiClient
	// configured is a flag to ensure configuration only happens once.
	configured bool
}

// ProviderModel defines the Go model for the provider's configuration.
type GreetingProviderModel struct {
	BaseURL types.String `tfsdk:"base_url"`
	APIKey  types.String `tfsdk:"api_key"`
}

// GetSchema defines the schema for the provider itself.
// This is where you define provider-level configuration attributes.
func (p *greetingProvider) GetSchema(ctx context.Context) (tfsdk.Schema, diag.Diagnostics) {
	return tfsdk.Schema{
		Description: "The greeting provider manages simple greeting messages via a custom API.",
		Attributes: map[string]tfsdk.Attribute{
			"base_url": {
				Description: "The base URL for the greeting API.",
				Type:        types.StringType,
				Optional:    true,
				// Set a default value for convenience during local development.
				// In a real provider, this might be required or sourced from env vars.
				Computed: true, // Can be set by user or default
			},
			"api_key": {
				Description: "The API key for authentication with the greeting API.",
				Type:        types.StringType,
				Optional:    true,
				Sensitive:   true, // Mark as sensitive to prevent logging
			},
		},
	}, nil
}

// Configure reads the provider configuration and initializes the API client.
// This client is then stored in the provider struct to be used by resources/data sources.
func (p *greetingProvider) Configure(ctx context.Context, req tfsdk.ConfigureProviderRequest, resp *tfsdk.ConfigureProviderResponse) {
	var config GreetingProviderModel

	// Read provider configuration into the Go model.
	resp.Diagnostics.Append(req.Config.Get(ctx, &config)...)
	if resp.Diagnostics.HasError() {
		return
	}

	// Determine the base URL. Use default if not provided by user.
	baseURL := "http://localhost:8080" // Default
	if !config.BaseURL.IsNull() && !config.BaseURL.IsUnknown() {
		baseURL = config.BaseURL.ValueString()
	}

	// Determine the API key.
	apiKey := ""
	if !config.APIKey.IsNull() && !config.APIKey.IsUnknown() {
		apiKey = config.APIKey.ValueString()
	}

	// Initialize your API client.
	// In a real provider, you would pass the API key for authentication.
	p.client = &apiClient{
		BaseURL: baseURL,
		HTTPClient: &http.Client{
			Timeout: 10 * time.Second,
		},
		APIKey: apiKey, // Pass API key to your client
	}

	p.configured = true // Mark provider as configured
	log.Printf("[INFO] Provider configured with BaseURL: %s", baseURL)
}

// Resources returns a list of resource types supported by this provider.
func (p *greetingProvider) Resources(ctx context.Context) []tfsdk.ResourceType {
	return []tfsdk.ResourceType{
		greetingResourceType{}, // Our greeting_message resource
	}
}

// DataSources returns a list of data source types supported by this provider.
func (p *greetingProvider) DataSources(ctx context.Context) []tfsdk.DataSourceType {
	return []tfsdk.DataSourceType{
		greetingDataSourceType{}, // Our greeting_message data source
	}
}

// GetMetadata returns the provider's metadata.
func (p *greetingProvider) GetMetadata(ctx context.Context) (tfsdk.ProviderMetadata, diag.Diagnostics) {
	return tfsdk.ProviderMetadata{
		// This version should match what you put in terraform { required_providers { ... version = "1.0.0" } }
		// You can automate this with build scripts for real projects.
		Version: "1.0.0",
	}, nil
}

// --- Remaining resource and data source definitions (from previous days) ---
// These are included here for completeness, but their content is the same as above.

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

// Update (from Day 4)
func (r *greetingResource) Update(ctx context.Context, req tfsdk.UpdateResourceRequest, resp *tfsdk.UpdateResourceResponse) {
	var plan, state GreetingModel
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

	newMessage := plan.Message.ValueString()
	newAuthor := plan.Author.ValueString()

	updatedGreeting, err := r.client.UpdateGreeting(ctx, greetingID, newMessage, newAuthor)
	if err != nil {
		resp.Diagnostics.AddError("Error updating greeting", fmt.Sprintf("Could not update greeting with ID %s: %s", greetingID, err.Error()))
		return
	}

	plan.ID = types.StringValue(updatedGreeting.ID)
	plan.Message = types.StringValue(updatedGreeting.Message)
	plan.Author = types.StringValue(updatedGreeting.Author)

	log.Printf("[INFO] Updated greeting with ID: %s", greetingID)
	resp.Diagnostics.Append(resp.State.Set(ctx, &plan)...)
}

// Delete (from Day 4)
func (r *greetingResource) Delete(ctx context.Context, req tfsdk.DeleteResourceRequest, resp *tfsdk.DeleteResourceResponse) {
	var state GreetingModel
	resp.Diagnostics.Append(req.State.Get(ctx, &state)...)
	if resp.Diagnostics.HasError() {
		return
	}

	greetingID := state.ID.ValueString()
	if greetingID == "" {
		log.Printf("[WARN] Attempted to delete resource with empty ID, assuming already deleted.")
		return
	}

	err := r.client.DeleteGreeting(ctx, greetingID)
	if err != nil {
		resp.Diagnostics.AddError("Error deleting greeting", fmt.Sprintf("Could not delete greeting with ID %s: %s", greetingID, err.Error()))
		return
	}

	resp.State.RemoveResource(ctx)
	log.Printf("[INFO] Deleted greeting with ID: %s", greetingID)
}

// ImportState (from Day 2)
func (r *greetingResource) ImportState(ctx context.Context, req tfsdk.ImportResourceStateRequest, resp *tfsdk.ImportResourceStateResponse) {
	tfsdk.State{}.SetAttribute(ctx, resp.State, "id", req.ID)
}

// greetingDataSourceType (from Day 5)
type greetingDataSourceType struct{}

// GetSchema (from Day 5)
func (d greetingDataSourceType) GetSchema(ctx context.Context) (tfsdk.Schema, diag.Diagnostics) {
	return tfsdk.Schema{
		Description: "Retrieves information about a greeting message.",
		Attributes: map[string]tfsdk.Attribute{
			"id": {
				Description: "The ID of the greeting to retrieve.",
				Type:        types.StringType,
				Optional:    true,
				Computed:    true,
			},
			"message": {
				Description: "The greeting message to retrieve.",
				Type:        types.StringType,
				Optional:    true,
				Computed:    true,
			},
			"author": {
				Description: "The author of the greeting.",
				Type:        types.StringType,
				Computed:    true,
			},
		},
	}, nil
}

// NewDataSource (from Day 5)
func (d greetingDataSourceType) NewDataSource(ctx context.Context, p tfsdk.Provider) (tfsdk.DataSource, diag.Diagnostics) {
	provider, ok := p.(*greetingProvider)
	if !ok {
		return nil, diag.Diagnostics{
			diag.NewErrorDiagnostic("Unexpected Provider Type", "Expected *greetingProvider."),
		}
	}
	return &greetingDataSource{
		client: provider.client,
	}, nil
}

// greetingDataSource (from Day 5)
type greetingDataSource struct {
	client *apiClient
}

// GreetingDataSourceModel (from Day 5)
type GreetingDataSourceModel struct {
	ID      types.String `tfsdk:"id"`
	Message types.String `tfsdk:"message"`
	Author  types.String `tfsdk:"author"`
}

// Read (from Day 5)
func (d *greetingDataSource) Read(ctx context.Context, req tfsdk.ReadDataSourceRequest, resp *tfsdk.ReadDataSourceResponse) {
	var config GreetingDataSourceModel
	resp.Diagnostics.Append(req.Config.Get(ctx, &config)...)
	if resp.Diagnostics.HasError() {
		return
	}

	var greeting *Greeting
	var err error

	if !config.ID.IsNull() && !config.ID.IsUnknown() {
		greetingID := config.ID.ValueString()
		greeting, err = d.client.GetGreetingByID(ctx, greetingID)
		if err != nil {
			resp.Diagnostics.AddError("Error reading greeting", fmt.Sprintf("Could not read greeting with ID %s: %s", greetingID, err.Error()))
			return
		}
	} else if !config.Message.IsNull() && !config.Message.IsUnknown() {
		log.Printf("[WARN] Data source: Searching by message is not efficient with current API. Consider adding an API search endpoint.")
		resp.Diagnostics.AddError(
			"Unsupported Data Source Lookup",
			"The 'greeting_message' data source currently only supports lookup by 'id'. "+
				"Searching by 'message' requires an API endpoint that supports it directly or fetching all greetings (less efficient).",
		)
		return
	} else {
		resp.Diagnostics.AddError(
			"Missing Data Source Arguments",
			"Either 'id' or 'message' must be provided for the 'greeting_message' data source.",
		)
		return
	}

	if greeting == nil {
		resp.Diagnostics.AddError(
			"Greeting Not Found",
			"No greeting found with the given criteria.",
		)
		return
	}

	config.ID = types.StringValue(greeting.ID)
	config.Message = types.StringValue(greeting.Message)
	config.Author = types.StringValue(greeting.Author)

	log.Printf("[INFO] Data source: Found greeting with ID: %s", greeting.ID)
	resp.Diagnostics.Append(resp.State.Set(ctx, &config)...)
}

```