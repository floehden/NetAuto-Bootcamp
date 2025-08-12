# Day 06: Handling Authentication and Headers
## Introduction

Many APIs require authentication to access their endpoints. You can also customize the headers of your requests to send additional information.

Key Concepts
* Authentication: *Use the auth parameter to handle basic HTTP authentication.
* Custom Headers: Pass custom headers using the headers parameter.

Basic Authentication:
```Python
from requests.auth import HTTPBasicAuth

response = requests.get('https://api.example.com/secure', auth=HTTPBasicAuth('username', 'password'))
print(response.status_code)
```
Custom Headers:
```Python
headers = {'User-Agent': 'MyApp/1.0'}
response = requests.get('https://api.example.com/data', headers=headers)
print(response.headers)
```

## Challenges
### Challenge 1: Basic Authentication
Send a GET request to an API that requires basic authentication.
### Challenge 2: Custom Headers
Write a script to send a GET request with custom headers and print the response.
### Challenge 3: API Key Authentication
Use an API key to authenticate and retrieve data from a public API. For example at demo.nautobot.com.

