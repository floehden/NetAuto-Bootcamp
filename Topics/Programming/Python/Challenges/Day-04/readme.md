# Day 04: Introduction to HTTP Requests
## Introduction

HTTP (HyperText Transfer Protocol) is the foundation of any data exchange on the Web. The requests library in Python allows you to send HTTP requests easily and handle the responses.

Key Concepts

Installing the Requests Library:

You can install the requests library using pip:
```sh
pip install requests
```
Making a GET Request:

Use requests.get() to send a GET request to a specified URL.

Handling Responses:

Check the status code of the response.
Access the response content.

Making a GET Request:
```Python
import requests

response = requests.get('https://api.example.com/data')
print(response.status_code)
print(response.text)
```
Handling Responses:
```Python
if response.status_code == 200:
    data = response.json()  # Assuming the response is JSON
    print(data)
else:
    print("Failed to retrieve data")
```
## Challenges
### Challenge 1: Basic GET Request
Write a script to send a GET request to a public API (e.g., JSONPlaceholder) and print the response.

### Challenge 2: Status Code Check
Modify the script to check the status code and print a success or failure message.

### Challenge 3: Response Headers
Print the response headers of the GET request.


## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%204%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain