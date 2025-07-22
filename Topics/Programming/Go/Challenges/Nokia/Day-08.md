
# **Day 8: Building a More Robust SSH Client**

## **Concept:** 
Enhancing the SSH client with functions for executing multiple commands, handling command prompts, and structured output parsing.

## **Challenge:** 
Modify the Day 7 code to accept a slice of commands and execute them sequentially, printing the output for each. Consider how you might "expect" certain prompts if doing interactive CLI.

## **Code Example (Conceptual - highly simplified interactive SSH):**
```go
// main.go
package main

import (
    "bytes"
    "fmt"
    "io"
    "log"
    "time"

    "golang.org/x/crypto/ssh"
)

// SSHClient represents an SSH client connection.
type SSHClient struct {
    client *ssh.Client
    session *ssh.Session
    stdin io.WriteCloser
    stdout io.Reader
    stderr io.Reader
}

// NewSSHClient establishes an SSH connection.
func NewSSHClient(addr, user, pass string) (*SSHClient, error) {
    config := &ssh.ClientConfig{
        User: user,
        Auth: []ssh.AuthMethod{
            ssh.Password(pass),
        },
        HostKeyCallback: ssh.InsecureIgnoreHostKey(),
        Timeout:         10 * time.Second,
    }

    client, err := ssh.Dial("tcp", addr, config)
    if err != nil {
        return nil, fmt.Errorf("failed to dial: %w", err)
    }

    session, err := client.NewSession()
    if err != nil {
        client.Close()
        return nil, fmt.Errorf("failed to create session: %w", err)
    }

    // Set up stdin, stdout, and stderr for interactive use
    stdin, _ := session.StdinPipe()
    stdout, _ := session.StdoutPipe()
    stderr, _ := session.StderrPipe()

    if err := session.Shell(); err != nil {
        session.Close()
        client.Close()
        return nil, fmt.Errorf("failed to start shell: %w", err)
    }

    return &SSHClient{
        client: client,
        session: session,
        stdin: stdin,
        stdout: stdout,
        stderr: stderr,
    }, nil
}

// RunCommand sends a command and returns the output.
func (c *SSHClient) RunCommand(cmd string) (string, error) {
    // Simple write and read, in a real scenario you'd need to handle prompts
    // and read until a known prompt is seen.
    _, err := fmt.Fprintf(c.stdin, "%s\n", cmd)
    if err != nil {
        return "", fmt.Errorf("failed to write command: %w", err)
    }

    // Give the device time to process and send output
    time.Sleep(500 * time.Millisecond)

    var buf bytes.Buffer
    _, err = io.CopyN(&buf, c.stdout, 1024) // Read up to 1KB, adjust as needed
    if err != nil && err != io.EOF {
            // EOF is expected if the command finishes before the buffer is full
        return "", fmt.Errorf("failed to read output: %w", err)
    }
    return buf.String(), nil
}

// Close closes the SSH session and client.
func (c *SSHClient) Close() {
    if c.session != nil {
        c.session.Close()
    }
    if c.client != nil {
        c.client.Close()
    }
}

func main() {
    // Ensure srl1 is deployed via Containerlab
    client, err := NewSSHClient("clab-srl-single-srl1:22", "admin", "NokiaSrl1!")
    if err != nil {
        log.Fatalf("Error creating SSH client: %v", err)
    }
    defer client.Close()

    commands := []string{
        "sr_cli show system information",
        "sr_cli show interface brief",
    }

    for _, cmd := range commands {
        fmt.Printf("\n--- Executing command: %s ---\n", cmd)
        output, err := client.RunCommand(cmd)
        if err != nil {
            log.Printf("Error running command '%s': %v", cmd, err)
        } else {
            fmt.Println(output)
        }
    }
}
```
**Note:** This `RunCommand` is a simplified example. For robust CLI automation, you'd typically use a library that handles "expect" patterns to wait for prompts, like `go-expect`.

