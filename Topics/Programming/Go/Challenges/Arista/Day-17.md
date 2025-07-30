
# **Day 17: Building a Simple CLI Tool with `flag`**

## **Introduction:** 
Creating command-line interface (CLI) tools in Go. Using the `flag` package for simple arguments or the `cobra` library for more complex CLIs.

## **Code Example: Basic Flag Usage**

```go
// cli_tool.go
package main

import (
    "flag"
    "fmt"
    "os"
)

func main() {
    hostname := flag.String("host", "", "Hostname or IP address of the device")
    command := flag.String("cmd", "", "Command to execute on the device")
    username := flag.String("user", "admin", "Username for device access")
    password := flag.String("pass", "admin", "Password for device access")

    flag.Parse() // Parse the command-line arguments

    if *hostname == "" || *command == "" {
        fmt.Println("Usage: cli_tool -host <ip> -cmd <command> [-user <user>] [-pass <pass>]")
        os.Exit(1)
    }

    fmt.Printf("Connecting to %s with user %s\n", *hostname, *username)
    fmt.Printf("Executing command: %s\n", *command)
    // In a real tool, you would use goeapi here with the provided host/user/pass
}
```

* To run: `go run cli_tool.go -host 10.0.0.1 -cmd "show version"`

## **Challenge 17:** 
Extend the CLI tool to accept an additional flag, `--config-file`, which takes a path to a configuration file. If the `--config-file` flag is provided, read the content of that file and print it, simulating sending it to the device.

