# Version Control Challenges


## Day 1: Setup and Basic Commands

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

## Day 2: More Basic Commands and History

**Goal:** Learn more basic commands and how to view history.

1. **Make Changes and Commit:**
   - Modify the `README.md` file.
   - Check the status and commit the changes:
     ```bash
     git status
     git add README.md
     git commit -m "Updated README"
     ```

2. **View History:**
   - View the commit history:
     ```bash
     git log
     ```

3. **Create a `.gitignore` File:**
   - Create a `.gitignore` file and add some files or directories to ignore.
   - Add and commit the `.gitignore` file:
     ```bash
     git add .gitignore
     git commit -m "Added gitignore file"
     ```

## Day 3: Branching

**Goal:** Learn how to create and switch branches.

1. **Create a New Branch:**
   - Create a new branch called `feature-branch`:
     ```bash
     git branch feature-branch
     ```

2. **Switch to the New Branch:**
   - Switch to the `feature-branch`:
     ```bash
     git checkout feature-branch
     ```

3. **Make Changes and Commit:**
   - Create a new file called `feature.txt` and add some text to it.
   - Add and commit the file:
     ```bash
     git add feature.txt
     git commit -m "Added feature.txt"
     ```

4. **Switch Back to the Main Branch:**
   - Switch back to the main branch:
     ```bash
     git checkout main
     ```

## Day 4: Merging Branches

**Goal:** Learn how to merge branches.

1. **Merge `feature-branch` into `main`:**
   - Make sure you are on the `main` branch:
     ```bash
     git checkout main
     ```
   - Merge the `feature-branch` into `main`:
     ```bash
     git merge feature-branch
     ```

2. **Resolve Merge Conflicts (if any):**
   - If there are any conflicts, resolve them manually by editing the conflicting files.
   - Add and commit the resolved files:
     ```bash
     git add <file>
     git commit -m "Resolved merge conflicts"
     ```

## Day 5: Remote Repositories

**Goal:** Learn how to work with remote repositories.

1. **Create a Remote Repository:**
   - Create a new repository on GitHub or any other Git hosting service.

2. **Add a Remote:**
   - Add the remote repository to your local repository:
     ```bash
     git remote add origin <remote-repository-URL>
     ```

3. **Push to the Remote Repository:**
   - Push your local repository to the remote repository:
     ```bash
     git push -u origin main
     ```

4. **Clone a Repository:**
   - Clone a remote repository to a new directory:
     ```bash
     git clone <remote-repository-URL> new-directory
     ```

By the end of these 5 days, you should have a good understanding of basic Git commands, branching, merging, and working with remote repositories. Happy coding!