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
