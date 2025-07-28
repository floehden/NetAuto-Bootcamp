# Day 3: Adding Flags and Persistent Flags

## **Goal:** 
Enhance the `ping` command with a `--timeout` flag and understand persistent flags.

## **Steps:**

1.  **Add a local flag for `ping`:**
* In `cmd/ping.go`, define a variable for the timeout.
* In `init()` function of `pingCmd`, use `pingCmd.Flags().IntVarP` to define the `--timeout` flag.
* Modify the `Run` function to use the timeout value.

2.  **Understand Persistent Flags:**
* Review `rootCmd.PersistentFlags()` in `cmd/root.go`. These flags are available to the root command and all its subcommands. While we won't add a new persistent flag today, understand its concept.

## **Code Example (`cmd/ping.go`):**

```go
package cmd

import (
	"fmt"
	"os/exec"
	"runtime"
	"strconv" // Import for converting int to string

	"github.com/spf13/cobra"
)

var timeout int // Variable to store the timeout value

// pingCmd represents the ping command
var pingCmd = &cobra.Command{
	Use:   "ping <target>",
	Short: "Pings a network device to check reachability.",
	Long: `The ping command sends ICMP echo requests to a specified network
device (IP address or hostname) to determine if it's reachable.
You can specify a custom timeout for the ping operation.`,
	Args: cobra.ExactArgs(1), // Requires exactly one argument: the target
	Run: func(cmd *cobra.Command, args []string) {
		target := args[0]
		fmt.Printf("Pinging %s with timeout %ds...\n", target, timeout)

		// Determine the correct ping command based on OS
		var pingCmdStr string
		var pingArgs []string

		if runtime.GOOS == "windows" {
			pingCmdStr = "ping"
			// -w: timeout in milliseconds
			pingArgs = []string{"-n", "4", "-w", strconv.Itoa(timeout * 1000), target}
		} else {
			pingCmdStr = "ping"
			// -W: timeout in seconds
			pingArgs = []string{"-c", "4", "-W", strconv.Itoa(timeout), target}
		}

		cmdOutput, err := exec.Command(pingCmdStr, pingArgs...).CombinedOutput()
		if err != nil {
			fmt.Printf("Error pinging %s: %v\n", target, err)
			fmt.Println(string(cmdOutput))
			return
		}
		fmt.Println(string(cmdOutput))
	},
}

func init() {
	rootCmd.AddCommand(pingCmd)

	// Define the --timeout flag (local to ping command)
	pingCmd.Flags().IntVarP(&timeout, "timeout", "t", 5, "Timeout for the ping operation in seconds")
}
```

**Run your app:**

```bash
go run main.go ping 8.8.8.8 --timeout 20
go run main.go ping google.com -t 30
```
