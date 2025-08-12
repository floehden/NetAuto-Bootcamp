# Day 05 
## Introduction
Building a Cronjob


## Challenges

### Challenge 1: Understanding Cron and Basic Syntax

**Objective**: Learn what cron is and understand the basic syntax of a cron job.

**Explanation**:
Cron is a time-based job scheduler in Unix-like operating systems. It enables users to schedule jobs (commands or scripts) to run automatically at specified intervals. The cron jobs are defined in a crontab file, where each line represents a single job.

The basic syntax of a cron job is:
```
* * * * * command_to_execute
```
The five asterisks represent time fields in the following order:
1. Minute (0 - 59)
2. Hour (0 - 23)
3. Day of the month (1 - 31)
4. Month (1 - 12)
5. Day of the week (0 - 7, where both 0 and 7 represent Sunday)

**Tasks**:
1. Open the terminal.
2. Type `crontab -e` to edit your crontab file.
3. Add a comment at the top of the file explaining what cron is in your own words.
4. Save and exit the editor.

### Challenge 2: Scheduling a Simple Command

**Objective**: Schedule a simple command to run at a specific time.

**Explanation**:
To schedule a command, you specify the time fields and the command to run. For example, to run a command every day at 3:30 PM, you would use:
```
30 15 * * * /path/to/command
```

**Tasks**:
1. Create a simple shell script named `hello.sh` that prints "Hello, Cron!" to the terminal.
2. Make the script executable with `chmod +x hello.sh`.
3. Edit your crontab file using `crontab -e`.
4. Add a line to schedule `hello.sh` to run every minute.
5. Save the crontab file and monitor the output to ensure the script runs as expected.

### Challenge 3: Using Wildcards and Intervals

**Objective**: Learn to use wildcards and intervals to schedule jobs more flexibly.

**Explanation**:
Wildcards (`*`) can be used to represent "any value." Intervals can be specified using hyphens (`-`) and lists using commas (`,`). For example, to run a command every 10 minutes, you would use:
```
*/10 * * * * /path/to/command
```

**Tasks**:
1. Edit your crontab file using `crontab -e`.
2. Modify the existing line to run `hello.sh` every 5 minutes.
3. Add a new line to run `hello.sh` every Monday at 8:00 AM.
4. Save the crontab file and verify the changes.

### Challenge 4: Redirecting Output to a File

**Objective**: Learn to redirect the output of a cron job to a file.

**Explanation**:
By default, the output of a cron job is emailed to the user. To redirect the output to a file, you can use the `>` operator. For example:
```
* * * * * /path/to/command > /path/to/output.txt 2>&1
```
This redirects both standard output and standard error to `output.txt`.

**Tasks**:
1. Create a new shell script named `date.sh` that prints the current date and time.
2. Make the script executable with `chmod +x date.sh`.
3. Edit your crontab file using `crontab -e`.
4. Add a line to schedule `date.sh` to run every minute and redirect the output to a file named `date_output.txt`.
5. Save the crontab file and check the `date_output.txt` file to ensure the output is being written correctly.

### Challenge 5: Advanced Scheduling with Cron

**Objective**: Practice advanced scheduling techniques with cron.

**Explanation**:
You can combine wildcards, intervals, and lists to create more complex schedules. For example, to run a command at 9:00 AM, 12:00 PM, and 3:00 PM every weekday, you would use:
```
0 9,12,15 * * 1-5 /path/to/command
```

**Tasks**:
1. Edit your crontab file using `crontab -e`.
2. Add a line to run `hello.sh` every weekday (Monday to Friday) at 7:00 AM, 12:00 PM, and 5:00 PM.
3. Add another line to run `date.sh` every 15 minutes on weekends (Saturday and Sunday).
4. Save the crontab file and verify that the jobs run as scheduled.

By completing these five parts, you'll have a solid understanding of how to schedule tasks using cron jobs in Linux. Good luck!

## More interesting script
Since cron jobs are not designed to handle intervals smaller than a minute, you can't directly create a cron job that runs every 10 seconds. However, you can use a workaround by creating a script that runs continuously in the background and handles the 10-second interval itself. You can then use a cron job to ensure that this script is always running, for example, by starting it at system boot or checking periodically to see if it's running and restarting it if necessary.

Here's how you can set this up:

### Step 1: Create the Script

First, create the script that will handle the 10-second interval. Save this script to a file, for example, `funny_logger.sh`.

```bash
#!/bin/bash

# Define an array of funny sentences
funny_sentences=(
    "Why don’t scientists trust atoms? Because they make up everything!"
    "Did you hear about the mathematician who’s afraid of negative numbers? He’ll stop at nothing to avoid them!"
    "Why don’t skeletons fight each other? They don’t have the guts."
    "I told my wife she was drawing her eyebrows too high. She looked surprised."
    "What do you call a fake noodle? An impasta."
)

# Function to log time and a funny sentence
log_funny_sentence() {
    while true; do
        current_time=$(date "+%Y-%m-%d %H:%M:%S")
        random_index=$((RANDOM % ${#funny_sentences[@]}))
        random_sentence=${funny_sentences[$random_index]}
        echo "[$current_time] $random_sentence" >> /path/to/funny_log.txt
        sleep 10
    done
}

# Check if the script is already running
if [ $(pgrep -c -f "funny_logger.sh") -gt 2 ]; then
    echo "Script is already running."
    exit
fi

# Start logging
log_funny_sentence
```

Make sure to replace `/path/to/funny_log.txt` with the actual path where you want to store the log file.

### Step 2: Make the Script Executable

Run the following command to make the script executable:

```bash
chmod +x /path/to/funny_logger.sh
```

### Step 3: Create a Cron Job to Ensure the Script is Running

You can create a cron job to check if the script is running and start it if it's not. For example, you can add the following line to your crontab file to check every 5 minutes:

1. Open your crontab file:

```bash
crontab -e
```

2. Add the following line to check and start the script if it's not running:

```bash
*/5 * * * * /bin/bash -c 'if ! pgrep -f "/path/to/funny_logger.sh" > /dev/null; then /path/to/funny_logger.sh & fi'
```

This cron job will check every 5 minutes if the `funny_logger.sh` script is running. If it's not running, it will start the script.

### Step 4: Start the Script Manually

You can also start the script manually to ensure it's running:

```bash
/path/to/funny_logger.sh &
```

This setup ensures that your script runs continuously, logging the time and a funny sentence every 10 seconds, and the cron job helps to keep it running in case it stops for any reason.</br>
</br>
Stay curious on Day 06 we have a final projects that combines the knowledge of all the days!
