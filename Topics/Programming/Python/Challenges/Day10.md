
# Day 10: File Transfers with SFTP 📂

## **Concept:** 
Paramiko includes a full-featured SFTP (SSH File Transfer Protocol) client for securely uploading and downloading files. You can open an SFTP session from an existing `SSHClient` connection using the `open_sftp()` method. Key methods are `put()` for uploading and `get()` for downloading.

## **Code Example:**
This script uploads a local file to the server and then downloads it back with a new name.

```python
import paramiko
import getpass
import os

# --- Connection Details (replace with yours) ---
hostname = 'your_server_ip_or_hostname'
username = 'your_username'
password = getpass.getpass()

# --- File Details ---
local_file_path = 'upload_this.txt'
remote_dir = f'/home/{username}/'
remote_file_path = os.path.join(remote_dir, 'uploaded_file.txt')
downloaded_file_path = 'downloaded_file.txt'

# Create a dummy local file to upload
with open(local_file_path, 'w') as f:
    f.write("Hello from Paramiko SFTP!\n")
    f.write("This is a test file for Day 3.")

client = paramiko.SSHClient()
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

try:
    client.connect(hostname=hostname, username=username, password=password)
    
    # --- Open an SFTP Session ---
    sftp = client.open_sftp()
    print("✅ SFTP session opened.")

    # --- Upload the File ---
    print(f"Uploading {local_file_path} to {remote_file_path}...")
    sftp.put(local_file_path, remote_file_path)
    print("✅ Upload successful.")

    # --- Download the File ---
    print(f"Downloading {remote_file_path} to {downloaded_file_path}...")
    sftp.get(remote_file_path, downloaded_file_path)
    print("✅ Download successful.")

    # Verify content of downloaded file
    with open(downloaded_file_path, 'r') as f:
        print("\n--- Content of downloaded file ---")
        print(f.read())

    # --- Close SFTP and Client ---
    sftp.close()

except Exception as e:
    print(f"❌ An error occurred: {e}")
finally:
    client.close()
    # Clean up local files
    if os.path.exists(local_file_path):
        os.remove(local_file_path)
    if os.path.exists(downloaded_file_path):
        os.remove(downloaded_file_path)
```

## **Day 10 Challenge:** 🎯
Write a script that connects to the remote server, lists all the files and directories in the user's home directory (`sftp.listdir('.')`), and prints them to the console.

## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%2010%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain