# **Day 5: Data Sources with Framework**

## **Objective:** 
Implement a data source using the Plugin Framework.

## **Data Source Definition:**

  * Similar to resources, but a struct that implements `framework.DataSource`.
  * It will also have a `Model` struct and a `Schema` method.
  * Only a `Read` method is implemented.

## **Implementing a "Greeting" Data Source:**

  * Allow users to look up a greeting by its ID or message.

## **Challenges:**

  * Distinguishing between `req.Config.Get()` (for data sources) and `req.Plan.Get()` (for resources).
  * Handling multiple potential lookup attributes (e.g., ID *or* message).

## **Code Example (Day 5 - Add `data_source_greeting_message.go`):**

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

// Ensure the implementation satisfies the tfsdk.DataSourceTypeWith
// and tfsdk.DataSource interfaces.
var _ tfsdk.DataSourceTypeWith
var _ tfsdk.DataSource = &greetingDataSource{}

// greetingDataSourceType defines the data source type.
type greetingDataSourceType struct{}

// GetSchema defines the schema for the greeting_message data source.
func (d greetingDataSourceType) GetSchema(ctx context.Context) (tfsdk.Schema, diag.Diagnostics) {
	return tfsdk.Schema{
		Description: "Retrieves information about a greeting message.",
		Attributes: map[string]tfsdk.Attribute{
			"id": {
				Description: "The ID of the greeting to retrieve.",
				Type:        types.StringType,
				Optional:    true, // Optional for lookup
				Computed:    true, // Computed from the API
			},
			"message": {
				Description: "The greeting message to retrieve.",
				Type:        types.StringType,
				Optional:    true, // Optional for lookup
				Computed:    true, // Computed from the API
			},
			"author": {
				Description: "The author of the greeting.",
				Type:        types.StringType,
				Computed:    true, // Always computed from the API
			},
		},
	}, nil
}

// NewDataSource creates a new data source instance.
func (d greetingDataSourceType) NewDataSource(ctx context.Context, p tfsdk.Provider) (tfsdk.DataSource, diag.Diagnostics) {
	// Cast the provider interface to our specific provider type to access the client.
	provider, ok := p.(*greetingProvider)
	if !ok {
		return nil, diag.Diagnostics{
			diag.NewErrorDiagnostic("Unexpected Provider Type", "Expected *greetingProvider."),
		}
	}
	return &greetingDataSource{
		client: provider.client, // Pass the configured API client
	}, nil
}

// greetingDataSource implements the tfsdk.DataSource interface.
type greetingDataSource struct {
	client *apiClient // Our API client instance
}

// GreetingDataSourceModel defines the Go model for the greeting_message data source.
// This struct maps directly to the schema attributes.
type GreetingDataSourceModel struct {
	ID      types.String `tfsdk:"id"`
	Message types.String `tfsdk:"message"`
	Author  types.String `tfsdk:"author"`
}

// Read handles data source reading.
func (d *greetingDataSource) Read(ctx context.Context, req tfsdk.ReadDataSourceRequest, resp *tfsdk.ReadDataSourceResponse) {
	var config GreetingDataSourceModel

	// Read the configuration (user's input) into our Go model.
	resp.Diagnostics.Append(req.Config.Get(ctx, &config)...)
	if resp.Diagnostics.HasError() {
		return
	}

	var greeting *Greeting
	var err error

	// Prioritize lookup by ID if provided.
	if !config.ID.IsNull() && !config.ID.IsUnknown() {
		greetingID := config.ID.ValueString()
		greeting, err = d.client.GetGreetingByID(ctx, greetingID)
		if err != nil {
			resp.Diagnostics.AddError("Error reading greeting", fmt.Sprintf("Could not read greeting with ID %s: %s", greetingID, err.Error()))
			return
		}
	} else if !config.Message.IsNull() && !config.Message.IsUnknown() {
		// If no ID, try to find by message.
		// NOTE: Our simple API does not have a "search by message" endpoint.
		// For a real API, you'd call an appropriate search endpoint here.
		// For this example, we'll simulate by fetching all and filtering (less efficient).
		// In a real-world scenario, you'd likely have a dedicated API endpoint for this.
		log.Printf("[WARN] Data source: Searching by message is not efficient with current API. Consider adding an API search endpoint.")

		// This part is a workaround for our simple API.
		// In a real API, you'd call a list endpoint and filter.
		// For now, we'll just error out if message is used without ID for simplicity.
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

	// Set the state based on the API response.
	config.ID = types.StringValue(greeting.ID)
	config.Message = types.StringValue(greeting.Message)
	config.Author = types.StringValue(greeting.Author)

	log.Printf("[INFO] Data source: Found greeting with ID: %s", greeting.ID)

	// Set the updated state for the data source.
	resp.Diagnostics.Append(resp.State.Set(ctx, &config)...)
}
```

**Update `main.go` to include the data source:**

```go
package main

import (
	"context"
	"log"

	"github.com/hashicorp/terraform-plugin-framework/providerserver"
	"github.com/hashicorp/terraform-plugin-framework/tfsdk"
)

// main function is the entry point for the provider.
func main() {
	err := providerserver.Serve(context.Background(), New, providerserver.ServeOpts{
		Address: "registry.terraform.io/local/greeting",
	})

	if err != nil {
		log.Fatal(err.Error())
	}
}

// New is a function that returns a new instance of our provider.
func New() tfsdk.Provider {
	return &greetingProvider{}
}

// Placeholder for our provider struct (will be defined on Day 6)
type greetingProvider struct{}

// Placeholder for the provider's schema (will be defined on Day 6)
func (p *greetingProvider) GetSchema(ctx context.Context) (tfsdk.Schema, diag.Diagnostics) {
	return tfsdk.Schema{}, nil // Empty for now
}

// Placeholder for provider configuration (will be defined on Day 6)
func (p *greetingProvider) Configure(ctx context.Context, req tfsdk.ConfigureProviderRequest, resp *tfsdk.ConfigureProviderResponse) {
	// No configuration yet
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

// greetingResourceType (from Day 2)
type greetingResourceType struct{}

// greetingDataSourceType (from Day 5)
type greetingDataSourceType struct{}

```

**Terraform Configuration Example (`main.tf` with data source):**

```go
terraform {
  required_providers {
    greeting = {
      source  = "local/greeting"
      version = "1.0.0"
    }
  }
}

provider "greeting" {
  base_url = "http://localhost:8080"
}

resource "greeting_message" "hello" {
  message = "Hello from Terraform Framework!"
  author  = "Framework Builder"
}

output "hello_id" {
  value = greeting_message.hello.id
}

output "hello_message_resource" {
  value = greeting_message.hello.message
}

# Data source lookup by ID
data "greeting_message" "specific_hello_data" {
  id = greeting_message.hello.id
}

output "data_hello_message_output" {
  value = data.greeting_message.specific_hello_data.message
}
```