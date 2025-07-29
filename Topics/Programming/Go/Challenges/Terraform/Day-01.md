# **Day 1: Introduction to Plugin Framework & Setup**

## **Objective:** 
Understand the advantages of the Plugin Framework and set up the development environment.

## **Simple Use Case Reminder: A "Hello World" Service Provider**

We'll continue with our "Greeting" resource and data source, now interacting with the Go HTTP "Greeting API" you set up.

## **Introduction:**

### **Why the Plugin Framework?**

	* **Type Safety:** Leverages Go generics for strong type checking, reducing runtime errors.
	* **Declarative Schema:** Define schemas using Go structs and tags, making them more readable and maintainable.
	* **Improved Developer Experience:** Streamlined API, better diagnostics, and clearer separation of concerns.
	* **Future-Proof:** The recommended path forward by HashiCorp.

### **Core Concepts:**

	* `framework.Provider`: The entry point for your provider.
	* `framework.Resource`: Interface for managing infrastructure objects.
	* `framework.DataSource`: Interface for fetching existing data.
	* `tfsdk.Schema`: Defines the structure of your configuration and state.
	* `tfsdk.Attribute`: Represents individual attributes within the schema (e.g., `tfsdk.StringAttribute`, `tfsdk.Int64Attribute`).
	* `tfsdk.State` & `tfsdk.Plan`: Objects for interacting with Terraform's state and plan.
	* `types.String`, `types.Int64`, `types.Bool`, etc.: Framework-specific types for attribute values.

## Instructions

Clone the template repository
```bash
git clone https://github.com/hashicorp/terraform-provider-scaffolding-framework
```

Rename the folder to our Greetings project
```bash
mv terraform-provider-scaffolding-framework terraform-provider-greetings
```

Rename the `go.mod` module
```bash
cd terraform-provider-greetings
go mod edit -module terraform-provider-greetings
```

Install all dependencies
```bash
go mod tidy
```

Replace the import in `main.go` with
```go
import (
    "context"
    "flag"
    "log"

    "github.com/hashicorp/terraform-plugin-framework/providerserver"

    "terraform-provider-greetings/internal/provider"
)

```

Open the `internal/provider/provider.go` with the following
```go
package provider

import (
	"context"

	"github.com/hashicorp/terraform-plugin-framework/datasource"
	"github.com/hashicorp/terraform-plugin-framework/provider"
	"github.com/hashicorp/terraform-plugin-framework/provider/schema"
	"github.com/hashicorp/terraform-plugin-framework/resource"
)

// Ensure the implementation satisfies the expected interfaces.
var (
	_ provider.Provider = &greetingsProvider{}
)

// New is a helper function to simplify provider server and testing implementation.
func New(version string) func() provider.Provider {
	return func() provider.Provider {
		return &greetingsProvider{
			version: version,
		}
	}
}

// greetingsProvider is the provider implementation.
type greetingsProvider struct {
	// version is set to the provider version on release, "dev" when the
	// provider is built and ran locally, and "test" when running acceptance
	// testing.
	version string
}

// Metadata returns the provider type name.
func (p *greetingsProvider) Metadata(_ context.Context, _ provider.MetadataRequest, resp *provider.MetadataResponse) {
	resp.TypeName = "greetings"
	resp.Version = p.version
}

// Schema defines the provider-level schema for configuration data.
func (p *greetingsProvider) Schema(_ context.Context, _ provider.SchemaRequest, resp *provider.SchemaResponse) {
	resp.Schema = schema.Schema{}
}

// Configure prepares a Greetings API client for data sources and resources.
func (p *greetingsProvider) Configure(ctx context.Context, req provider.ConfigureRequest, resp *provider.ConfigureResponse) {
}

// DataSources defines the data sources implemented in the provider.
func (p *greetingsProvider) DataSources(_ context.Context) []func() datasource.DataSource {
	return nil
}

// Resources defines the resources implemented in the provider.
func (p *greetingsProvider) Resources(_ context.Context) []func() resource.Resource {
	return nil
}
```


## **Challenges:**

  * Struct-based schema definition.
  * Understanding the new `tfsdk.Request` and `tfsdk.Response` patterns for CRUD operations.




## **Code Example (Day 1 - `main.go`):**


```go
package main

import (
	"context"
	"log" // For logging messages

	"github.com/hashicorp/terraform-plugin-framework/providerserver"
	"github.com/hashicorp/terraform-plugin-framework/tfsdk" // New import
)

// main function is the entry point for the provider.
func main() {
	// The providerserver.Serve function starts the gRPC server for the provider.
	// It takes a context and a function that returns a new provider instance.
	// This is the standard way to serve a framework-based provider.
	err := providerserver.Serve(context.Background(), New, providerserver.ServeOpts{
		// A human-friendly name for the provider. This is used in error messages.
		Address: "registry.terraform.io/local/greeting",
	})

	if err != nil {
		log.Fatal(err.Error()) // Log any fatal errors during serving
	}
}

// New is a function that returns a new instance of our provider.
// It's required by providerserver.Serve.
func New() tfsdk.Provider {
	return &greetingProvider{} // We'll define greetingProvider on Day 6
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

// Placeholder for resources (will be defined on Day 6)
func (p *greetingProvider) Resources(ctx context.Context) []tfsdk.ResourceType {
	return []tfsdk.ResourceType{
		greetingResourceType{}, // Placeholder for our resource type
	}
}

// Placeholder for data sources (will be defined on Day 6)
func (p *greetingProvider) DataSources(ctx context.Context) []tfsdk.DataSourceType {
	return []tfsdk.DataSourceType{
		greetingDataSourceType{}, // Placeholder for our data source type
	}
}

// Placeholder for our resource type (will be defined on Day 2)
type greetingResourceType struct{}

// Placeholder for our data source type (will be defined on Day 5)
type greetingDataSourceType struct{}

```