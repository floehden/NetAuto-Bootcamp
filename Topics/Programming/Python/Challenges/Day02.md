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
