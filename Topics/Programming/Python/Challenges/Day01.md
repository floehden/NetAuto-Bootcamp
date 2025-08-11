# Day 1: Basics of File Handling
## Introduction

Opening and Closing Files:
```Python
# Opening a file
file = open('example.txt', 'r')

# Closing a file
file.close()
```

Reading Files:

Using read():
```Python 
with open('example.txt', 'r') as file:
    content = file.read()
    print(content)
```
Using readline():
```Python
with open('example.txt', 'r') as file:
    line = file.readline()
    while line:
        print(line, end='')
        line = file.readline()
```

Using readlines():
```Python
with open('example.txt', 'r') as file:
    lines = file.readlines()
    for line in lines:
        print(line, end='')
```
File Modes:

* 'r' for reading
* 'w' for writing
* 'a' for appending
* 'r+' for both reading and writing.


## Challenges
### Challenge 1: Read a File
Write a script to open a text file and print its contents to the console.
### Challenge 2: Count Lines
Create a program that counts the number of lines in a text file.
### Challenge 3: Simple Copy
Write a program that reads the content of one file and writes it to another file.
