# Day 04 
## Introduction
Building and running a shell script


## Challenges
### Challenge 1: Hello World and Basic Commands

**Objective**: Write your first shell script and learn basic commands.

**Tasks**:
1. Open your terminal.
2. Create a new directory called `shell_scripting` and navigate into it.
3. Create a file named `hello_world.sh`.
4. Use a text editor (like `nano` or `vim`) to add the following script to the file:
   ```bash
   #!/bin/bash
   echo "Hello, World!"
   ```
5. Make the script executable with the command `chmod +x hello_world.sh`.
6. Run the script using `./hello_world.sh`.
7. Modify the script to also print the current date and time.

### Challenge 2: Variables and User Input

**Objective**: Learn to use variables and take user input in a shell script.

**Tasks**:
1. Create a new file named `greet_user.sh`.
2. Write a script that asks the user for their name and greets them:
   ```bash
   #!/bin/bash
   echo "What is your name?"
   read name
   echo "Hello, $name! Nice to meet you."
   ```
3. Make the script executable and run it.
4. Extend the script to ask for the user's age and print it along with the greeting.

### Challenge 3: Conditional Statements

**Objective**: Use conditional statements to control the flow of your script.

**Tasks**:
1. Create a new file named `check_age.sh`.
2. Write a script that asks the user for their age and checks if they are eligible to vote (age 18 or older):
   ```bash
   #!/bin/bash
   echo "How old are you?"
   read age
   if [ "$age" -ge 18 ]; then
       echo "You are eligible to vote."
   else
       echo "You are not eligible to vote yet."
   fi
   ```
3. Make the script executable and run it.
4. Modify the script to also check if the user is a teenager (age between 13 and 19).

### Challenge 4
Loops and Functions

**Objective**: Learn to use loops and functions in your shell script.

**Tasks**:
1. Create a new file named `countdown.sh`.
2. Write a script that counts down from 5 to 1 and then prints "Blast off!":
   ```bash
   #!/bin/bash
   for ((i=5; i>=1; i--)); do
       echo $i
       sleep 1
   done
   echo "Blast off!"
   ```
3. Make the script executable and run it.
4. Add a function to the script that prints a custom message before the countdown starts. Call this function at the beginning of the script.

By completing these four parts, you'll have a good understanding of the basics of shell scripting, including writing scripts, using variables, taking user input, conditional statements, loops, and functions. Good luck!

### Challenge 5 

Come up with your own script and what would you like a script to do?</br>
We are curious about what you like to run in a script!

## Final ToDo

Post about your journey and what you did as #Challenge5Linux, what you learned on different platforms like [LinkedIn](https://www.linkedin.com/feed/) or [Twitter](https://x.com/intent/post?url=https%3A%2F%2Fgithub.com%2FNetAuto-RheinMain%2FNetAuto-Bootcamp&text=I%20just%20completed%20Day%204%20of%20the%20NetAuto%20Bootcamp%20on%20Linux!&hashtags=NetAutoBootcamp%2CNetworkAutomation) or any other of your favourite platforms. Follow up on your journey and share it with others! Use the Hashtags #NetAutoBootcamp #NetworkAutomation </br>
You can also tag us on LinkedIn with @netauto-group-rheinmain