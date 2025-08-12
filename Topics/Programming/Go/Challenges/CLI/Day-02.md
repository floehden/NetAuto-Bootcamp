# Day 2: Adding a `ping` Command

## **Goal:** 
Implement a basic `ping` command that takes a single IP address/hostname as an argument.

## **Steps:**

1.  **Add `ping` command:**
```bash
cobra-cli add ping
```
This creates `cmd/ping.go`.

2.  **Modify `cmd/ping.go`:**
* Add `Args: cobra.ExactArgs(1)` to ensure exactly one argument (the target).
* Implement the `Run` function to execute a simple ping using `exec.Command`.

## **Code Example (`cmd/ping.go`):**

```go
package cmd

import (
	"fmt"
	"os/exec"
	"runtime"

	"github.com/spf13/cobra"
)

// pingCmd represents the ping command
var pingCmd = &cobra.Command{
	Use:   "ping <target>",
	Short: "Pings a network device to check reachability.",
	Long: `The ping command sends ICMP echo requests to a specified network
device (IP address or hostname) to determine if it's reachable.`,
	Args: cobra.ExactArgs(1), // Requires exactly one argument: the target
	Run: func(cmd *cobra.Command, args []string) {
		target := args[0]
		fmt.Printf("Pinging %s...\n", target)

		// Determine the correct ping command based on OS
		var pingCmdStr string
		var pingArgs []string

		if runtime.GOOS == "windows" {
			pingCmdStr = "ping"
			pingArgs = []string{"-n", "4", target} // 4 packets on Windows
		} else {
			pingCmdStr = "ping"
			pingArgs = []string{"-c", "4", target} // 4 packets on Linux/macOS
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
}
```

**Run your app:**

```bash
go run main.go ping 8.8.8.8
go run main.go ping google.com
```
