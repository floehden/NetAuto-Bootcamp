# Day 09: Executing Remote Commands 💻

## **Concept:** 
Once connected, the most common task is executing shell commands on the remote server. The `exec_command()` method is used for this. It returns three file-like objects: `stdin`, `stdout`, and `stderr`, which correspond to the standard input, output, and error streams of the command you executed.

## **Code Example:**
This script connects to a server, executes the `ls -l` command, and prints the standard output and any potential errors.

```python
import paramiko
import getpass

# --- Connection Details (replace with yours) ---
hostname = 'your_server_ip_or_hostname'
username = 'your_username'
password = getpass.getpass()

client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

try:
    client.connect(hostname=hostname, username=username, password=password)
    
    # --- Execute a Command ---
    command = 'ls -l /home'
    print(f"\nExecuting command: '{command}'")
    stdin, stdout, stderr = client.exec_command(command)
    
    # Read and print the output
    output = stdout.read().decode('utf-8')
    print("\n--- Command Output (stdout) ---")
    if output:
        print(output)
    else:
        print("No output.")
    
    # Read and print any errors
    error = stderr.read().decode('utf-8')
    print("\n--- Command Error (stderr) ---")
    if error:
        print(error)
    else:
        print("No errors.")
        
    # Get the exit status of the command
    exit_status = stdout.channel.recv_exit_status()
    print(f"\nExit Status: {exit_status}")


except Exception as e:
    print(f"❌ An error occurred: {e}")
finally:
    client.close()
```

## **Day 09 Challenge:** 🎯
Write a script that executes two commands:

1.  `uname -a` to get the system's kernel information and print it.
2.  A deliberately incorrect command, like `list_files`, to see how `stderr` captures the "command not found" error. Print the error message.

## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%209%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain