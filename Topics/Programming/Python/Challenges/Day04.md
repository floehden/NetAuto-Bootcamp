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
Write a script to send a GET request to a public API (e.g., [No as a Service](https://github.com/hotheadhacker/no-as-a-service/tree/main), [JSONPlaceholder](https://jsonplaceholder.typicode.com/)) and print the response.

### Challenge 2: Status Code Check
Modify the script to check the status code and print a success or failure message.

### Challenge 3: Response Headers
Print the response headers of the GET request.
