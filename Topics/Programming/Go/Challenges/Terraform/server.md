
## The "Greeting API" Counterpart (Go HTTP Server)

This Go application will provide the REST endpoints that your Terraform provider will call for Create, Read, Update, and Delete operations on "greeting messages."

### Project Structure

You can create a new directory for this API, separate from your provider project.

```
/greeting-api
  ├── main.go
```

### API Endpoints

We'll define the following simple REST endpoints:

  * **`POST /greetings`**: Create a new greeting.
      * Request Body: `{"message": "string", "author": "string"}`
      * Response: `{"id": "string", "message": "string", "author": "string"}`
  * **`GET /greetings/{id}`**: Get a greeting by ID.
      * Response: `{"id": "string", "message": "string", "author": "string"}`
  * **`PUT /greetings/{id}`**: Update an existing greeting.
      * Request Body: `{"message": "string", "author": "string"}`
      * Response: `{"id": "string", "message": "string", "author": "string"}`
  * **`DELETE /greetings/{id}`**: Delete a greeting by ID.
      * Response: `204 No Content`

-----

### Code for the "Greeting API" (`greeting-api/main.go`)

```go
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"strconv"
	"sync" // For concurrent map access
)

// Greeting represents our greeting message structure
type Greeting struct {
	ID      string `json:"id"`
	Message string `json:"message"`
	Author  string `json:"author"`
}

// In-memory "database" for greetings
var greetings = make(map[string]Greeting)
var nextID int // Our simple ID counter
var mu sync.Mutex // Mutex to protect shared map access

func init() {
	nextID = 1
}

func main() {
	http.HandleFunc("/greetings", greetingsHandler)
	http.HandleFunc("/greetings/", greetingByIDHandler) // Handle specific IDs

	port := ":8080"
	log.Printf("Greeting API starting on port %s", port)
	log.Fatal(http.ListenAndServe(port, nil))
}

// greetingsHandler handles POST requests for creating greetings
func greetingsHandler(w http.ResponseWriter, r *http.Request) {
	switch r.Method {
	case http.MethodPost:
		createGreeting(w, r)
	default:
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
	}
}

// greetingByIDHandler handles GET, PUT, DELETE requests for specific greetings by ID
func greetingByIDHandler(w http.ResponseWriter, r *http.Request) {
	id := r.URL.Path[len("/greetings/"):] // Extract ID from URL path

	if id == "" {
		http.Error(w, "Greeting ID is required", http.StatusBadRequest)
		return
	}

	switch r.Method {
	case http.MethodGet:
		getGreetingByID(w, r, id)
	case http.MethodPut:
		updateGreetingByID(w, r, id)
	case http.MethodDelete:
		deleteGreetingByID(w, r, id)
	default:
		http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
	}
}

// createGreeting handles the creation of a new greeting
func createGreeting(w http.ResponseWriter, r *http.Request) {
	var newGreeting Greeting
	if err := json.NewDecoder(r.Body).Decode(&newGreeting); err != nil {
		http.Error(w, "Invalid request body", http.StatusBadRequest)
		return
	}

	if newGreeting.Message == "" {
		http.Error(w, "Message is required", http.StatusBadRequest)
		return
	}

	mu.Lock()
	newGreeting.ID = strconv.Itoa(nextID)
	nextID++
	if newGreeting.Author == "" {
		newGreeting.Author = "Anonymous" // Default author if not provided
	}
	greetings[newGreeting.ID] = newGreeting
	mu.Unlock()

	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(newGreeting)
	log.Printf("Created greeting: %v", newGreeting)
}

// getGreetingByID handles fetching a greeting by ID
func getGreetingByID(w http.ResponseWriter, r *http.Request, id string) {
	mu.Lock()
	greeting, ok := greetings[id]
	mu.Unlock()

	if !ok {
		http.Error(w, "Greeting not found", http.StatusNotFound)
		return
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(greeting)
	log.Printf("Fetched greeting: %v", greeting)
}

// updateGreetingByID handles updating an existing greeting by ID
func updateGreetingByID(w http.ResponseWriter, r *http.Request, id string) {
	var updatedGreeting Greeting
	if err := json.NewDecoder(r.Body).Decode(&updatedGreeting); err != nil {
		http.Error(w, "Invalid request body", http.StatusBadRequest)
		return
	}

	mu.Lock()
	existingGreeting, ok := greetings[id]
	if !ok {
		mu.Unlock()
		http.Error(w, "Greeting not found", http.StatusNotFound)
		return
	}

	// Update only the provided fields
	if updatedGreeting.Message != "" {
		existingGreeting.Message = updatedGreeting.Message
	}
	if updatedGreeting.Author != "" {
		existingGreeting.Author = updatedGreeting.Author
	} else if existingGreeting.Author == "Anonymous" && updatedGreeting.Author == "" {
		// If author was "Anonymous" and now explicitly empty, keep it anonymous
		// or handle as per your API's logic. For simplicity, we'll keep it.
	}

	greetings[id] = existingGreeting // Store the updated greeting
	mu.Unlock()

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(existingGreeting)
	log.Printf("Updated greeting: %v", existingGreeting)
}

// deleteGreetingByID handles deleting a greeting by ID
func deleteGreetingByID(w http.ResponseWriter, r *http.Request, id string) {
	mu.Lock()
	_, ok := greetings[id]
	if !ok {
		mu.Unlock()
		http.Error(w, "Greeting not found", http.StatusNotFound)
		return
	}

	delete(greetings, id)
	mu.Unlock()

	w.WriteHeader(http.StatusNoContent) // 204 No Content for successful deletion
	log.Printf("Deleted greeting with ID: %s", id)
}
```

-----

### How to Run the "Greeting API"

1.  **Save the code:** Save the code above as `main.go` in your `greeting-api` directory.
2.  **Initialize Go module:**
    ```bash
    cd greeting-api
    go mod init greeting-api
    ```
3.  **Run the API:**
    ```bash
    go run main.go
    ```
    You should see output indicating it's "Greeting API starting on port :8080".

-----

### Adjusting Your Terraform Provider to Use the API

Now, you'll need to modify your **Terraform provider's** `resource_greeting_message.go` and `data_source_greeting_message.go` files to make actual HTTP calls to this running API, instead of using the in-memory `greetings` map.

**Key Changes in Your Provider:**

1.  **Remove the in-memory `greetings` map and `nextID` from your provider's files.**
2.  **Update `apiClient` in `main.go`:** Ensure your `apiClient` struct has a `http.Client` and your base URL is configured.
3.  **Modify CRUD/Read functions:**
      * Instead of `greetings[...]`, you'll use `http.Get`, `http.Post`, `http.Put`, `http.Delete`.
      * Marshal Go structs to JSON for request bodies and unmarshal JSON responses into Go structs.
      * Handle HTTP status codes (e.g., 404 Not Found, 200 OK, 201 Created).

Here's a conceptual snippet of how your provider's `createGreeting` function would change (similar changes apply to Read, Update, Delete functions):

```go
// Inside resource_greeting_message.go or a separate api_client.go file in your provider
package main

import (
	"bytes"
	"context"
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"
    "time" // For http client timeout

	"github.com/hashicorp/terraform-plugin-sdk/v2/diag"
	"github.com/hashicorp/terraform-plugin-sdk/v2/helper/schema"
)

// Define Greeting struct in your provider, matching the API's structure
type Greeting struct {
	ID      string `json:"id,omitempty"` // omitempty for creation (ID is computed by API)
	Message string `json:"message"`
	Author  string `json:"author,omitempty"`
}

// In main.go, update apiClient
type apiClient struct {
	BaseURL string
	HTTPClient *http.Client
	APIKey string // If your API required it
}

func providerConfigure(ctx context.Context, d *schema.ResourceData) (interface{}, diag.Diagnostics) {
	apiBaseURL := d.Get("api_base_url").(string)
	apiKey := d.Get("api_key").(string) // Not used by our simple API, but good for real ones

	client := &apiClient{
		BaseURL: apiBaseURL,
		HTTPClient: &http.Client{
			Timeout: 10 * time.Second, // Set a timeout for HTTP requests
		},
		APIKey: apiKey,
	}

	return client, nil
}

// Example modified Create function in your provider
func resourceGreetingMessageCreate(ctx context.Context, d *schema.ResourceData, m interface{}) diag.Diagnostics {
	client := m.(*apiClient) // Get the API client from the meta interface

	message := d.Get("message").(string)
	author := d.Get("author").(string)

	reqBody := Greeting{
		Message: message,
		Author:  author,
	}

	jsonBody, err := json.Marshal(reqBody)
	if err != nil {
		return diag.FromErr(err)
	}

	resp, err := client.HTTPClient.Post(fmt.Sprintf("%s/greetings", client.BaseURL), "application/json", bytes.NewBuffer(jsonBody))
	if err != nil {
		return diag.FromErr(fmt.Errorf("failed to create greeting: %w", err))
	}
	defer resp.Body.Close()

	if resp.StatusCode != http.StatusCreated {
		bodyBytes, _ := ioutil.ReadAll(resp.Body)
		return diag.Errorf("API returned non-201 status for create: %d - %s", resp.StatusCode, string(bodyBytes))
	}

	var createdGreeting Greeting
	if err := json.NewDecoder(resp.Body).Decode(&createdGreeting); err != nil {
		return diag.FromErr(fmt.Errorf("failed to decode create response: %w", err))
	}

	d.SetId(createdGreeting.ID)
	d.Set("message", createdGreeting.Message)
	d.Set("author", createdGreeting.Author)

	return resourceGreetingMessageRead(ctx, d, m) // Read to ensure state is consistent
}

// You'll need similar modifications for resourceGreetingMessageRead, Update, and Delete,
// as well as dataSourceGreetingMessageRead.
```
1