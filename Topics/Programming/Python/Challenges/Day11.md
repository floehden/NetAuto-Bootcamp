
# Day 11: Authentication with SSH Keys 🔑

## **Concept:** 
Using SSH keys for authentication is far more secure than using passwords. Paramiko supports this by loading a private key from a file and passing it to the `connect()` method. You must first ensure your public key is in the `~/.ssh/authorized_keys` file on the remote server.

## **Code Example:**
This script connects to a server using a private key file.

```python
import paramiko
import os

# --- Connection Details ---
hostname = 'your_server_ip_or_hostname'
username = 'your_username'
# Path to your private key file (e.g., id_rsa, id_ed25519)
private_key_path = os.path.expanduser('~/.ssh/id_rsa') 

# --- Load Private Key ---
try:
    private_key = paramiko.RSAKey.from_private_key_file(private_key_path)
    # For other key types, use:
    # private_key = paramiko.Ed25519Key.from_private_key_file(private_key_path)
    # private_key = paramiko.ECDSAKey.from_private_key_file(private_key_path)
except paramiko.PasswordRequiredException:
    key_password = getpass.getpass('Enter password for private key: ')
    private_key = paramiko.RSAKey.from_private_key_file(private_key_path, password=key_password)
except Exception as e:
    print(f"❌ Failed to load private key: {e}")
    exit()

client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

try:
    print(f"Connecting to {hostname} using SSH key...")
    # --- Connect using the private key object ---
    client.connect(
        hostname=hostname,
        username=username,
        pkey=private_key
    )
    
    print("✅ Connection successful!")
    
    # Execute a simple command to verify
    stdin, stdout, stderr = client.exec_command('whoami')
    print(f"Authenticated as: {stdout.read().decode('utf-8').strip()}")

except Exception as e:
    print(f"❌ An error occurred: {e}")
finally:
    client.close()
```

## **Day 11 Challenge:** 🎯
Generate a new SSH key pair on your local machine using `ssh-keygen -t ed25519`. Copy the public key (`~/.ssh/id_ed25519.pub`) to your test server's `~/.ssh/authorized_keys` file. Modify today's script to use this new key for authentication.

## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%2011%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain
