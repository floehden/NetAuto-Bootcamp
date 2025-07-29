# Day 07: Error Handling and Session Management
## Introduction

Handling errors gracefully is crucial for robust applications. The requests library also supports sessions to persist certain parameters across requests.

Key Concepts
* Error Handling: Use try-except blocks to handle exceptions that may occur during requests.
* Session Management: Use requests.Session() to create a session object for persisting settings across requests.

Error Handling:
```Python
try:
    response = requests.get('https://api.example.com/data', timeout=5)
    response.raise_for_status()  # Raises an HTTPError for bad responses
except requests.exceptions.RequestException as e:
    print(f"An error occurred: {e}")
else: 
    pass
finally:
    pass
```
Session Management:
```Python
session = requests.Session()
session.headers.update({'User-Agent': 'MyApp/1.0'})
response = session.get('https://api.example.com/data')
print(response.text)
```

## Challenges
### Challenge1: Error Handling
Enhance a script to handle various exceptions that may occur during HTTP requests.

### Challenge 2: Session Usage
Use a session to send multiple requests to a public API and print the responses.

### Challenge 3: Persistent Headers
Create a session with custom headers and use it to interact with an API.

## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%207%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain