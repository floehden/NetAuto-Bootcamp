# Day 01
Welcome to the start of Challnges on the Topic Linux!

## Introduction
In todays challenge we will use folders and files with the following commands
* [man](https://manpages.ubuntu.com/)
* [mkdir](https://www.geeksforgeeks.org/mkdir-command-in-linux-with-examples/)
* [cd](https://tutorials.codebar.io/command-line/introduction/tutorial.html#:~:text=cd%20or%20change%20directory,command%20is%20cd%20your%2Ddirectory%20.&text=Now%20that%20we%20moved%20to,again%2C%20then%20cd%20into%20it.)
* [touch](https://www.explainshell.com/explain/1/touch)
* [vi]()
* [cat]()
* [awk]()
* [chmod]()
* [chown]()
* [sh]()
* [rm]()

## Prerequisites
* an installed Ubuntu system or Ubuntu in WSL

## Challenge

### Challenge 1
At first we look at the man pages. 

### Challenge 2
* Create a project folder with mkdir
```sh
mkdir project
```

Make the current user owner of the folder with chown
```sh
chown $USER project
```

Change into the folder with cd
```sh
cd project/
```

### Challenge 3
 Create a file with touch
```sh
touch file.txt
```

Open the file in the Texteditor vi
```sh
vi file.txt
```

Press i to be able to edit the file
Write some text to the file
```
This is some text.
```
Exit the file by pressing Escape and :wq
* Escape stops the insert mode. 
* w is for write
* q is for quit

print the text from the file to the terminal with cat
```sh
cat file.txt
```

It should now display something like
```sh
user@host:~$ cat file.txt
This is some text.
```

We can also just get single words with awk0
```sh
cat file.txt | awk '{print $3}'
```

```sh
user@host:~$ cat file.txt | awk '{print $3}'
some
```


### Challenge 4

* Analyze the Textfile `bible.txt`
* What is the first book of the bible? print it out



## Final ToDo

Post about your journey, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/), [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%201%20of%20the%20NetAuto%20Bootcamp%20on%20Linux!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @NetAutoGroupRM (or something else)