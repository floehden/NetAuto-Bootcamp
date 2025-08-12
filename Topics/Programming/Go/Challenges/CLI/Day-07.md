# Day 7: Configuration Management with Viper

## **Goal:** 
Integrate Viper for flexible configuration management (e.g., storing default credentials or device lists).

## **Steps:**

1.  **Install Viper:**
    ```bash
    go get github.com/spf13/viper
    ```
2.  **Modify `cmd/root.go`:**
      * Import `github.com/spf13/viper`.
      * Add an `initConfig()` function.
      * Call `initConfig()` in `init()`.
      * Set up Viper to read config files (e.g., `netcli.yaml` in user's home directory or current directory).
      * Bind flags to Viper for easy overriding.
3.  **Modify `cmd/show.go`:** Use Viper to get default values for username/password if not provided via flags.

## **Code Example (`cmd/root.go` - updated with Viper):**

```go
package cmd

import (
	"fmt"
	"os"

	"github.com/spf13/cobra"
	"github.com/spf13/viper" // Import Viper
)

var cfgFile string

// rootCmd represents the base command when called without any subcommands
var rootCmd = &cobra.Command{
	Use:   "netcli",
	Short: "A CLI tool for network automation tasks.",
	Long: `netcli is a powerful command-line interface tool designed to
streamline various network automation tasks, from checking device reachability
to managing configurations.`,
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
	cobra.OnInitialize(initConfig) // Initialize Viper when Cobra is initialized

	rootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.netcli.yaml)")

	// Define persistent flags that Viper can bind to
	rootCmd.PersistentFlags().StringP("username", "u", "", "Default username for device authentication")
	rootCmd.PersistentFlags().StringP("password", "p", "", "Default password for device authentication")
	viper.BindPFlag("username", rootCmd.PersistentFlags().Lookup("username"))
	viper.BindPFlag("password", rootCmd.PersistentFlags().Lookup("password"))
}

// initConfig reads in config file and ENV variables if set.
func initConfig() {
	if cfgFile != "" {
		// Use config file from the flag.
		viper.SetConfigFile(cfgFile)
	} else {
		// Find home directory.
		home, err := os.UserHomeDir()
		cobra.CheckErr(err)

		// Search config in home directory with name ".netcli" (without extension).
		viper.AddConfigPath(home)
		viper.AddConfigPath(".") // Look in current directory as well
		viper.SetConfigType("yaml")
		viper.SetConfigName(".netcli")
	}

	viper.AutomaticEnv() // read in environment variables that match

	// If a config file is found, read it in.
	if err := viper.ReadInConfig(); err == nil {
		fmt.Fprintln(os.Stderr, "Using config file:", viper.ConfigFileUsed())
	}
}
```

**Code Example (`cmd/show.go` - updated to use Viper):**

```go
package cmd

import (
	"fmt"
	"time"

	"github.com/spf13/cobra"
	"github.com/spf13/viper" // Import Viper
)

var (
	showHostname string // Renamed to avoid conflict with `hostname` global in root.go
	showUsername string // Renamed
	showPassword string // Renamed
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

		// Get values from flags or Viper
		host := showHostname
		user := showUsername
		pass := showPassword

		if host == "" {
			return fmt.Errorf("hostname not provided. Use --host or configure in .netcli.yaml")
		}
		if user == "" {
			user = viper.GetString("username") // Get from Viper
		}
		if pass == "" {
			pass = viper.GetString("password") // Get from Viper
		}

		if user == "" || pass == "" {
			return fmt.Errorf("username and/or password not provided. Use flags or configure in .netcli.yaml")
		}

		fmt.Printf("Attempting to connect to %s@%s to execute '%s'\n", user, host, showCommand)

		// ... (Same placeholder SSH logic as Day 6, using 'host', 'user', 'pass') ...

		fmt.Println("Simulating command execution...")
		time.Sleep(2 * time.Second) // Simulate network delay
		fmt.Printf("Simulated output for '%s' on %s:\n", showCommand, host)
		fmt.Println("Interface GigabitEthernet0/1 is up, line protocol is up")
		fmt.Println("  Hardware is Gigabit Ethernet, address is aabb.ccdd.eeff (bia aabb.ccdd.eeff)")

		return nil
	},
}

func init() {
	rootCmd.AddCommand(showCmd)

	// Local flags for the show command
	showCmd.Flags().StringVarP(&showHostname, "host", "H", "", "Target network device hostname or IP address")
	showCmd.Flags().StringVarP(&showUsername, "username", "u", "", "Username for device authentication (overrides config)")
	showCmd.Flags().StringVarP(&showPassword, "password", "p", "", "Password for device authentication (overrides config)")

	// Host is always required for the show command
	showCmd.MarkFlagRequired("host")
}
```

**Create a config file (`~/.netcli.yaml` or `netcli.yaml` in current directory):**

```yaml
username: my_default_user
password: my_default_password
```

**Run your app:**

```bash
go run main.go show "show version" -H 192.168.1.10 # Uses credentials from config
go run main.go show "show version" -H 192.168.1.11 -u specific_user -p specific_pass # Overrides config
```
