# Day 2: Writing and Appending to Files
## Introduction

Writing to Files:
```Python
with open('example.txt', 'w') as file:
    file.write("Hello, World!\n")
    file.write("This is a test file.\n")
```
Appending to Files:
```Python
with open('example.txt', 'a') as file:
    file.write("This line will be appended.\n")
```
Handling Exceptions:
```Python
try:
    with open('nonexistent.txt', 'r') as file:
        content = file.read()
except FileNotFoundError:
    print("File not found. Please check the file name.")
```

## Challenges
### Challenge 1: Write to a File
Write a script that asks the user for input and writes it to a file.
### Challenge 2: Append to a File
Modify the previous script to append user input to the file instead of overwriting it.
### Challenge 3: Error Handling
Enhance your script with exception handling to manage scenarios where the file might not exist or cannot be opened.

## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%202%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain