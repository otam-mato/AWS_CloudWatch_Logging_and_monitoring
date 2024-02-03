# Logging and monitoring a server with AWS Cloudwatch

## A solution which monitors at regular intervals and sends the results to AWS Cloudwatch:

- Processes currently running on the machine launched by a user called "transmogrifier".
- Contents of the Transmogrified/ folder.


## Process flow:
![image](https://github.com/otam-mato/AWS_CloudWatch_Logging_and_monitoring/assets/113034133/cb2af34a-a608-406d-94fc-06d4df91f00e)


## The script which we will use to log current processes and content of the folder:

```sh
#!/bin/bash

# This script is to run indefinitely and periodically log information about the transmogrifier process and its associated files.

while true; do  # Start an infinite loop

  # Log the list of processes running the transmogrifier command, along with the hostname and current date/time, to a file called transmogrifier_process.log
  sudo bash -c "sudo printf '\n%s %s %s\n\n%s\n' 'Processes lists for transmogrifier:' '$(hostname)' '$(date +'%Y-%m-%d %H:%M:%S')' '$(ps -u transmogrifier -f)' >> /var/log/transmogrifier_process.log"

  # Log the list of files in the Transmogrified directory, along with the hostname and current date/time, to a file called transmogrifier_files.log
  sudo bash -c "sudo printf '\n%s %s %s\n\n%s\n' 'File list of transmogrifier:' '$(hostname)' '$(date +'%Y-%m-%d %H:%M:%S')' '$(sudo ls -la /home/transmogrifier/Transmogrified/)' >> /var/log/transmogrifier_files.log"


  sleep 300  # Wait for 300 seconds (5 minutes) before running the loop again
done

```

## Launch the script in the background:

```sh
# This line grants execute permission to the transmogrifier-monitor.sh script, allowing it to be run as a command
sudo chmod +x /usr/local/bin/transmogrifier-monitor.sh  

# This line runs the transmogrifier-monitor.sh script in the background as a root user using the Bash shell
sudo nohup ./transmogrifier-monitor.sh &  

```
