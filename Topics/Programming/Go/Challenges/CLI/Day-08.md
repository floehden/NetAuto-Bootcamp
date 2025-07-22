### Day 8: Building and Best Practices

**Goal:** Learn how to build your CLI application and discuss best practices for larger projects.

**Steps:**

1.  **Build the executable:**
```bash
go build -o netcli main.go
```
Now you have a single `netcli` executable.

2.  **Test the executable:**
```bash
./netcli --help
./netcli ping 8.8.8.8
```

3.  **Best Practices:**
* **Project Structure:** For larger projects, organize your `cmd` directory with sub-directories for related commands (e.g., `cmd/device`, `cmd/config`).
* **Modular Code:** Extract complex logic into separate packages (e.g., `pkg/network`, `pkg/auth`) to keep `cmd` files clean and focused on CLI interaction.
* **Interface-based Design:** Define interfaces for network device interactions (e.g., `NetworkDevice interface { Connect() error; RunCommand(cmd string) (string, error) }`) to allow for different device types or protocols.
* **Context for Cancellation/Timeouts:** Use `context.Context` for long-running operations or network calls to handle cancellations and timeouts gracefully.
* **Testing:** Write unit tests for your core logic and integration tests for CLI commands.
* **Linter/Static Analysis:** Use `go vet` and tools like `golangci-lint` to maintain code quality.
* **Binary Size Optimization:** For very small CLIs, consider `go build -ldflags "-s -w"` to strip debug symbols and reduce binary size.
* **Cross-Compilation:** Go's strength is cross-compilation:
```bash
GOOS=linux GOARCH=amd64 go build -o netcli_linux_amd64 main.go
GOOS=windows GOARCH=amd64 go build -o netcli_windows_amd64.exe main.go
```
