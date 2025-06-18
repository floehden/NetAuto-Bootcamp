# Day 05: Query Parameters and POST Requests
## Introduction

Query parameters allow you to send additional information in a GET request. POST requests are used to send data to a server to create or update a resource.

Key Concepts

Query Parameters:

Pass query parameters using the params argument in requests.get().

Making a POST Request:

Use requests.post() to send a POST request with data.
Code Examples

Query Parameters:
```Python
params = {'key1': 'value1', 'key2': 'value2'}
response = requests.get('https://api.example.com/data', params=params)
print(response.url)  # This will show the URL with the query parameters
```
Making a POST Request:
```Python
data = {'key1': 'value1', 'key2': 'value2'}
response = requests.post('https://api.example.com/data', data=data)
print(response.status_code)
print(response.json())
```

## Challenges
### Challenge 1: GET with Query Parameters
Send a GET request with query parameters to a public API and print the response.

### Challenge 2: POST Request
Write a script to send a POST request with JSON data to a public API (e.g., JSONPlaceholder) and print the response.

### Challenge 3: Data Submission
Create a script that submits form data to a server using a POST request.

## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%201%20of%20the%20NetAuto%20Bootcamp%20on%20Linux!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain