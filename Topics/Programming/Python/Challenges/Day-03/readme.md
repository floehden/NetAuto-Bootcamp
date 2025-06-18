# Day 3: Working with Different File Types and Advanced Operations
## Introduction 

Working with CSV Files:

```Python
import csv

# Writing to a CSV file
with open('example.csv', 'w', newline='') as file:
    writer = csv.writer(file)
    writer.writerow(["Name", "Age"])
    writer.writerow(["Alice", 30])
    writer.writerow(["Bob", 25])

# Reading from a CSV file
with open('example.csv', 'r') as file:
    reader = csv.reader(file)
    for row in reader:
        print(row)
```

Working with JSON Files:
```Python
import json

# Writing to a JSON file
data = {"name": "Alice", "age": 30, "city": "New York"}
with open('example.json', 'w') as file:
    json.dump(data, file)

# Reading from a JSON file
with open('example.json', 'r') as file:
    data = json.load(file)
    print(data)
```

File and Directory Management:

```Python
import os

# Listing files in a directory
files = os.listdir('.')
print(files)

# Renaming a file
os.rename('old_name.txt', 'new_name.txt')

# Checking file existence
if os.path.exists('example.txt'):
    print("File exists.")
else:
    print("File does not exist.")
```

## Challenges
### Challenge 1: CSV File Handling
Write a script that reads data from a CSV file and prints it in a formatted manner.

### Challenge 2: JSON File Handling: 
Create a program that reads JSON data from a file, modifies it, and writes it back to the file.

### Challenge 3: Directory Operations: 
Write a script that lists all files in a directory and identifies which ones are text files.


## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%203%20of%20the%20NetAuto%20Bootcamp%20on%20Python%20Programming!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain