# Day 6: Advanced Network Interaction (Placeholder for a Real-World Scenario)

## **Goal:** 
Understand how to integrate actual network automation libraries. We won't implement a full SSH client, but show where it would fit.

## **Steps:**
1.  **Identify Go Network Libraries:** Research libraries for network device interaction, e.g., `go-telnet`, `go.netconf`, `go.ssh`. For SSH, `golang.org/x/crypto/ssh` is the standard.

2.  **Create a new command, e.g., `show` to get device info.**
```bash
cobra-cli add show
```

3.  **Modify `cmd/show.go`:**
* Add flags for hostname, username, password.
* In the `RunE` function, demonstrate a placeholder for establishing an SSH connection and executing a command.

## **Code Example (`cmd/show.go`):**

```go
package cmd

import (
	"fmt"
	"time"

	"github.com/spf13/cobra"
	// "golang.org/x/crypto/ssh" // Uncomment if you actually implement SSH
)

var (
	hostname string
	username string
	password string
)

// showCmd represents the show command
var showCmd = &cobra.Command{
	Use:   "show <command>",
	Short: "Executes a show command on a network device.",
	Long: `The show command connects to a specified network device via SSH
and executes a given command, displaying its output. This is a placeholder
for actual SSH interaction.`,
	Args: cobra.ExactArgs(1),
	RunE: func(cmd *cobra.Command, args []string) error {
		showCommand := args[0]
		fmt.Printf("Attempting to connect to %s@%s to execute '%s'\n", username, hostname, showCommand)

		// --- Placeholder for actual SSH connection and command execution ---
		// In a real application, you would use a library like golang.org/x/crypto/ssh
		// to establish an SSH connection, authenticate, and run the command.

		// Example (conceptual) SSH client logic:
		/*
			config := &ssh.ClientConfig{
				User: username,
				Auth: []ssh.AuthMethod{
					ssh.Password(password),
				},
				Timeout: 10 * time.Second,
				HostKeyCallback: ssh.InsecureIgnoreHostKey(), // DANGER: Insecure for production
			}

			client, err := ssh.Dial("tcp", hostname+":22", config)
			if err != nil {
				return fmt.Errorf("failed to dial: %w", err)
			}
			defer client.Close()

			session, err := client.NewSession()
			if err != nil {
				return fmt.Errorf("failed to create session: %w", err)
			}
			defer session.Close()

			output, err := session.CombinedOutput(showCommand)
			if err != nil {
				return fmt.Errorf("failed to run command '%s': %w\nOutput:\n%s", showCommand, err, string(output))
			}

			fmt.Printf("Output from %s:\n%s\n", hostname, string(output))
		*/
		// --- End Placeholder ---

		fmt.Println("Simulating command execution...")
		time.Sleep(2 * time.Second) // Simulate network delay
		fmt.Printf("Simulated output for '%s' on %s:\n", showCommand, hostname)
		fmt.Println("Interface GigabitEthernet0/1 is up, line protocol is up")
		fmt.Println("  Hardware is Gigabit Ethernet, address is aabb.ccdd.eeff (bia aabb.ccdd.eeff)")

		return nil
	},
}

func init() {
	rootCmd.AddCommand(showCmd)

	showCmd.Flags().StringVarP(&hostname, "host", "H", "", "Target network device hostname or IP address")
	showCmd.Flags().StringVarP(&username, "username", "u", "", "Username for device authentication")
	showCmd.Flags().StringVarP(&password, "password", "p", "", "Password for device authentication")

	// Mark host, username, and password as required
	showCmd.MarkFlagRequired("host")
	showCmd.MarkFlagRequired("username")
	showCmd.MarkFlagRequired("password")
}
```

**Run your app:**

```bash
go run main.go show "show ip int brief" -H 192.168.1.10 -u admin -p mypassword
```

*(Note: This will only show simulated output as the SSH integration is commented out.)*
