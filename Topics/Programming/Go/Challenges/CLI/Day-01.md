# Day 1: Getting Started with Go and Cobra

## **Goal:** Set up your Go environment and create a basic Cobra CLI application.

## **Steps:**

1.  **Install Go:** If you don't have Go installed, follow the official instructions: `https://golang.org/doc/install`
2.  **Install Cobra CLI Generator:**
    ```bash
    go install github.com/spf13/cobra-cli@latest
    ```
3.  **Create a new Go module:**
    ```bash
    mkdir netcli
    cd netcli
    go mod init github.com/yourusername/netcli # Replace with your GitHub username
    ```
4.  **Initialize Cobra project:**
    ```bash
    cobra-cli init
    ```
    This will create a `cmd` directory with `root.go` and `main.go`.

## **Code Example (`main.go`):**

```go
package main

import "github.com/yourusername/netcli/cmd" // Replace with your module path

func main() {
	cmd.Execute()
}
```

**Code Example (`cmd/root.go`):**

```go
package cmd

import (
	"os"
	"github.com/spf13/cobra"
)

var cfgFile string

// rootCmd represents the base command when called without any subcommands
var rootCmd = &cobra.Command{
	Use:   "netcli",
	Short: "A CLI tool for network automation tasks.",
	Long: `netcli is a powerful command-line interface tool designed to
streamline various network automation tasks, from checking device reachability
to managing configurations.`,
	// Uncomment the following line if your bare application
	// has an action associated with it:
	// Run: func(cmd *cobra.Command, args []string) { },
}

// Execute adds all child commands to the root command and sets flags appropriately.
// This is called by main.main(). It only needs to happen once to the rootCmd.
func Execute() {
	err := rootCmd.Execute()
	if err != nil {
		os.Exit(1)
	}
}

func init() {
	// Here you will define your flags and configuration settings.
	// Cobra supports persistent flags, which, if defined here,
	// will be available to all subcommands.
	rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.netcli.yaml)")

	// Cobra also supports local flags, which will only run
	// when this action is called directly.
	// rootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}
```

## **Run your app:**

```bash
go run main.go
./netcli --help
```
