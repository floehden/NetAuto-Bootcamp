# Day 12: Final Project - Simple Deployment Script 🚀

## **Concept:** 
Let's combine everything we've learned to create a practical automation script. The goal is to deploy a simple application by creating a directory, uploading files, installing dependencies, and running the app. This demonstrates a realistic use case for Paramiko.

## **Code Example & Final Challenge:**
Your final task is to write a single Python script that performs the following steps. This *is* the challenge. Use the concepts and code from the previous days as your building blocks.

## **Project Requirements:**

1.  **Local Setup:**

      * Create a local folder named `my_app`.
      * Inside `my_app`, create a file `app.py` with the content: `print("🎉 My application is running successfully!")`
      * Create another file `requirements.txt` with the content: `cowsay`

2.  **Remote Automation Script:**

      * Connect to your remote server using **SSH key authentication**.
      * Define a remote path, e.g., `remote_app_path = '/tmp/my_deployed_app'`.
      * **Execute a command** to create this directory on the remote server (`mkdir -p /tmp/my_deployed_app`).
      * Use **SFTP** to upload `app.py` and `requirements.txt` into the remote directory.
      * **Execute a command** to install the dependencies using pip: `python3 -m pip install -r /tmp/my_deployed_app/requirements.txt`.
      * **Execute a final command** to run the application and show its output: `python3 /tmp/my_deployed_app/app.py | cowsay`.
      * Print the output from the previous command to your local console.
      * **Cleanup:** Execute a command to remove the remote directory (`rm -rf /tmp/my_deployed_app`).
      * Close the connection and print a "Deployment complete\!" message.
      * Wrap your logic in `try...finally` to ensure the connection is always closed.

Good luck\! This final challenge will solidify your understanding of Paramiko's core workflow.

## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%2012%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain