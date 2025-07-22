# Day 5: Error Handling and Exit Codes

## **Goal:** Improve error handling and ensure correct exit codes for CLI operations.

## **Steps:**

1.  **Review existing error handling:**
* In `main.go`, `cmd.Execute()` already handles errors by exiting with `1`.
* In `pingCmd.Run` and `multiCmd.Run`, we print errors.

2.  **Return errors from `Run` functions:** Cobra allows `RunE` instead of `Run` to return an error. This error will then be handled by `cmd.Execute()`.

3.  **Refactor `pingHost` to return errors more gracefully.**

**Code Example (`cmd/ping.go` - updated to use `RunE`):**

```go
package cmd

import (
	"fmt"
	"os/exec"
	"runtime"
	"strconv"

	"github.com/spf13/cobra"
)

var timeout int

// pingCmd represents the ping command
var pingCmd = &cobra.Command{
	Use:   "ping <target>",
	Short: "Pings a network device to check reachability.",
	Long: `The ping command sends ICMP echo requests to a specified network
device (IP address or hostname) to determine if it's reachable.
You can specify a custom timeout for the ping operation.`,
	Args: cobra.ExactArgs(1),
	RunE: func(cmd *cobra.Command, args []string) error { // Changed to RunE
		target := args[0]
		fmt.Printf("Pinging %s with timeout %ds...\n", target, timeout)

		if err := pingHost(target, timeout); err != nil {
			return fmt.Errorf("ping failed for %s: %w", target, err) // Wrap the error
		}
		return nil
	},
}

func init() {
	rootCmd.AddCommand(pingCmd)
	pingCmd.Flags().IntVarP(&timeout, "timeout", "t", 5, "Timeout for the ping operation in seconds")
}

// pingHost remains the same as in Day 4 (returns error)
func pingHost(target string, timeout int) error {
	var pingCmdStr string
	var pingArgs []string

	if runtime.GOOS == "windows" {
		pingCmdStr = "ping"
		pingArgs = []string{"-n", "4", "-w", strconv.Itoa(timeout * 1000), target}
	} else {
		pingCmdStr = "ping"
		pingArgs = []string{"-c", "4", "-W", strconv.Itoa(timeout), target}
	}

	cmdOutput, err := exec.Command(pingCmdStr, pingArgs...).CombinedOutput()
	if err != nil {
		return fmt.Errorf("ping command failed: %v\n%s", err, string(cmdOutput))
	}
	fmt.Println(string(cmdOutput))
	return nil
}
```

**Code Example (`cmd/multi.go` - updated to use `RunE`):**

```go
package cmd

import (
	"bufio"
	"fmt"
	"os"
	"strings"

	"github.com/spf13/cobra"
)

var targetsFile string

// multiCmd represents the multi command
var multiCmd = &cobra.Command{
	Use:   "multi [targets...]",
	Short: "Pings multiple network devices.",
	Long: `The multi command allows you to ping multiple network devices.
You can provide IP addresses or hostnames as arguments directly,
or specify a file containing one target per line using the --file flag.`,
	RunE: func(cmd *cobra.Command, args []string) error { // Changed to RunE
		var targets []string

		if targetsFile != "" {
			file, err := os.Open(targetsFile)
			if err != nil {
				return fmt.Errorf("error opening targets file: %w", err)
			}
			defer file.Close()

			scanner := bufio.NewScanner(file)
			for scanner.Scan() {
				target := strings.TrimSpace(scanner.Text())
				if target != "" {
					targets = append(targets, target)
				}
			}
			if err := scanner.Err(); err != nil {
				return fmt.Errorf("error reading targets file: %w", err)
			}
		} else if len(args) > 0 {
			targets = args
		} else {
			return fmt.Errorf("no targets provided. Use 'netcli ping multi <targets...>' or 'netcli ping multi --file <filename>'")
		}

		if len(targets) == 0 {
			return fmt.Errorf("no targets to ping")
		}

		for _, target := range targets {
			fmt.Printf("\n--- Pinging %s ---\n", target)
			if err := pingHost(target, timeout); err != nil { // timeout is from parent (pingCmd)
				fmt.Printf("Ping failed for %s: %v\n", target, err)
				// Decide if you want to stop on first error or continue and report all errors
				// For this example, we'll continue but print the error.
			}
		}
		return nil
	},
}

func init() {
	pingCmd.AddCommand(multiCmd)
	multiCmd.Flags().StringVarP(&targetsFile, "file", "f", "", "File containing network targets (one per line)")
}
```

**Run your app:**

```bash
go run main.go ping non_existent_host # Should return error code 1
echo "" > empty.txt
go run main.go ping multi --file empty.txt # Should return error code 1
```

