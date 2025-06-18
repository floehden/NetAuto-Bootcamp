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
Use an API key to authenticate and retrieve data from a public API. For example at demo.nautobot.com


## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%206%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain