# Day 4: Handling Multiple Targets with a Subcommand

## **Goal:** Create a subcommand `ping multi` that reads targets from a file or takes multiple arguments.

## **Steps:**

1.  **Add `multi` subcommand under `ping`:**
```bash
cobra-cli add multi -p pingCmd
```
This creates `cmd/multi.go` and sets its parent to `pingCmd`.

2.  **Modify `cmd/multi.go`:**
* Remove `Args: cobra.ExactArgs(1)` from `pingCmd` (or make it optional if desired).
* In `multiCmd`, add a flag `--file` to specify a file containing targets.
* If no file is provided, treat remaining arguments as targets.
* Iterate through targets and call the ping logic (perhaps refactor the ping logic into a separate function).

## **Code Example (`cmd/ping.go` - updated):**
No changes needed if `multiCmd` handles its own argument parsing. `pingCmd` retains its `ExactArgs(1)` for single pings.

**Code Example (`cmd/multi.go`):**

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
	Run: func(cmd *cobra.Command, args []string) {
		var targets []string

		if targetsFile != "" {
			// Read targets from file
			file, err := os.Open(targetsFile)
			if err != nil {
				fmt.Printf("Error opening targets file: %v\n", err)
				return
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
				fmt.Printf("Error reading targets file: %v\n", err)
				return
			}
		} else if len(args) > 0 {
			// Use targets from arguments
			targets = args
		} else {
			fmt.Println("Error: No targets provided. Use 'netcli ping multi <targets...>' or 'netcli ping multi --file <filename>'")
			return
		}

		if len(targets) == 0 {
			fmt.Println("No targets to ping.")
			return
		}

		// Re-use the ping logic from Day 3
		for _, target := range targets {
			// This will use the 'timeout' global variable from pingCmd
			// as it's a persistent flag on the parent (pingCmd)
			fmt.Printf("\n--- Pinging %s ---\n", target)

			// The 'pingCmd' from Day 3 directly executes `exec.Command`
			// For reusability, we can extract the core ping logic into a function.
			// For simplicity, we'll call the `pingCmd.Run` function directly,
			// but in a real-world scenario, you'd refactor `pingHost` function.
			// Let's create a helper function for pinging:
			if err := pingHost(target, timeout); err != nil {
				fmt.Printf("Ping failed for %s: %v\n", target, err)
			}
		}
	},
}

func init() {
	pingCmd.AddCommand(multiCmd) // Add multiCmd as a subcommand of pingCmd

	// Define the --file flag for multi command
	multiCmd.Flags().StringVarP(&targetsFile, "file", "f", "", "File containing network targets (one per line)")
	multiCmd.Flags().IntVarP(&timeout, "timeout", "t", 5, "Timeout for the ping operation in seconds")
}

// Helper function to encapsulate ping logic
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

**Run your app:**

```bash
go run main.go ping multi 192.168.1.1 8.8.8.8
echo "192.168.1.1" > targets.txt
echo "google.com" >> targets.txt
go run main.go ping multi --file targets.txt -t 30
```
