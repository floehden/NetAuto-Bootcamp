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


### Challenges
### Challenge 1: Read a File
Write a script to open a text file and print its contents to the console.
### Challenge 2: Count Lines
Create a program that counts the number of lines in a text file.
### Challenge 3: Simple Copy
Write a program that reads the content of one file and writes it to another file.

## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%201%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain