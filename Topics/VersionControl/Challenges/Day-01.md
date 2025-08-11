# Day 1: Setup and Basic Commands

**Goal:** Set up Git and learn basic commands.

1. **Install Git:**
   - Download and install Git from [git-scm.com](https://git-scm.com/).

2. **Configure Git:**
   - Open your shell and set up your username and email:
     ```bash
     git config --global user.name "Your Name"
     git config --global user.email "your.email@example.com"
     ```

3. **Create a New Repository:**
   - Create a new directory for your project and initialize a Git repository:
     ```bash
     mkdir git-challenge
     cd git-challenge
     git init
     ```

4. **Basic Commands:**
   - Create a new file called `README.md` and add some text to it.
   - Check the status of your repository:
     ```bash
     git status
     ```
   - Add the file to the staging area:
     ```bash
     git add README.md
     ```
   - Commit the file:
     ```bash
     git commit -m "Initial commit with README"
     ```
