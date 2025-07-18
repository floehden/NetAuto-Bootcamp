### \#\# Day 8: Introduction & Basic SSH Connection

**Concept:** The first step is learning how to establish a basic, password-authenticated SSH connection to a remote server. The core of Paramiko's client functionality is the `SSHClient` object. For security, an SSH client must verify the identity of the server it connects to. We'll use `AutoAddPolicy` for simplicity, which automatically trusts unknown host keys (convenient for testing, but insecure for production).

**Code Example:**
This script connects to a remote server and prints the connection banner.

```python
import paramiko
import getpass # To securely ask for a password

# --- Connection Details ---
hostname = 'your_server_ip_or_hostname'
port = 22
username = 'your_username'

# Use getpass to avoid hardcoding the password
try:
    password = getpass.getpass(f"Enter password for {username}@{hostname}: ")
except Exception as error:
    print('ERROR', error)
    exit()

# --- Create SSH Client ---
client = paramiko.SSHClient()

# !! IMPORTANT !!
# This line automatically adds the server's host key.
# In a real-world application, you should verify the host key.
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

try:
    # --- Connect to the Server ---
    print(f"Connecting to {hostname}...")
    client.connect(hostname=hostname, port=port, username=username, password=password)
    
    # If connection is successful, the server often sends a banner
    banner = client.get_transport().get_banner()
    if banner:
        print("\n--- Server Banner ---")
        print(banner)
        print("---------------------\n")
    
    print("✅ Connection successful!")

except paramiko.AuthenticationException:
    print("❌ Authentication failed. Please check your credentials.")
except paramiko.SSHException as sshException:
    print(f"❌ Unable to establish SSH connection: {sshException}")
except Exception as e:
    print(f"❌ An error occurred: {e}")
finally:
    # --- Close the Connection ---
    print("Closing connection.")
    client.close()
```

**Day 8 Challenge:** 🎯
Set up a simple SSH server using a Docker container or a virtual machine (like Vagrant with VirtualBox). Alternatively, you can use a public test server like `telehack.com` (user/pass not required). Modify the script above to connect to your test server.



