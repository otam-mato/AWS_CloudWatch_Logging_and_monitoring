# Logging and monitoring a user activity on a server with AWS Cloudwatch

## Let's assume we want to monitor a user 'Transmogrifier' and his activity at regular intervals and send the results to AWS Cloudwatch, including:

- Processes currently running on the machine launched by a user called "transmogrifier".
- Contents of the Transmogrified/ folder.


## Process flow:
![image](https://github.com/otam-mato/AWS_CloudWatch_Logging_and_monitoring/assets/113034133/cb2af34a-a608-406d-94fc-06d4df91f00e)


## We will use this script to log the current processes and contents of the user's folder:

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

## Launch the script in the background to start logging:

```sh
# This line grants execute permission to the transmogrifier-monitor.sh script, allowing it to be run as a command
sudo chmod +x /usr/local/bin/transmogrifier-monitor.sh  

# This line runs the transmogrifier-monitor.sh script in the background as a root user using the Bash shell
sudo nohup ./transmogrifier-monitor.sh &  

```

## Set up the logs to be regularly sent to AWS CloudWatch by installing the CloudWatch agent to the controlled machine
<br>

To configure the CloudWatch agent to send the required logs to CloudWatch, follow these steps:

**1. SSH into the EC2 instance hosting the servers that will run the Transmogrifier.**

**2. Install the awslogs package. This is the recommended method for installing awslogs on Amazon Linux instances.**
``` sh
sudo yum update -y

sudo yum install -y awslogs
```

**3. Once installed, open the CloudWatch Logs agent configuration file located at /etc/awslogs/awslogs.conf.**

**4. Add log files that you want to monitor to the configuration file, specifying the log file location, log format, and destination log group in CloudWatch.**<br> Here is an example configuration entry:
``` sh
[/var/log/transmogrifier_process.log]
datetime_format = %b %d %H:%M:%S
file = /var/log/transmogrifier_process.log
buffer_duration = 5000
log_stream_name = {instance_id}
initial_position = start_of_file
log_group_name = transmogrifier_demo_processes

[/var/log/transmogrifier_files.log]
datetime_format = %b %d %H:%M:%S
file = /var/log/transmogrifier_files.log
buffer_duration = 5000
log_stream_name = {instance_id}
initial_position = start_of_file
log_group_name = transmogrifier_demo_files
```
In this example, we're monitoring the **/var/log/transmogrifier_process.log** file and sending its contents to log groups named **transmogrifier_demo_processes** and **transmogrifier_demo_files** in **CloudWatch**. The **log_stream_name** parameter will automatically include the instance ID in the log stream name, allowing you to distinguish between logs from different instances.

**5. By default, the /etc/awslogs/awscli.conf points to the us-east-1 Region. To push your logs to a different Region, edit the awscli.conf file and specify that Region.**

**6. If you are running Amazon Linux 2, start the awslogs service with the following command:**

```
sudo systemctl start awslogsd
```
**7. (Optional) Run the following command to start the awslogs service at each system boot:**

```
sudo systemctl enable awslogsd.service
```
> Note that installing and configuring CloudWatch Logs on an existing Ubuntu Server, CentOS, or Red Hat instance will vary. In more details here:
>
> https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/QuickStartEC2Instance.html

<br>

## Create the AWS IAM role to acess CloudWatch and attach it to the controlled EC2 instance.

<br>

<img width="800" alt="Screenshot 2023-02-22 at 15 41 41" src="https://user-images.githubusercontent.com/104728608/220776314-cd01c708-c7ce-4ccf-800f-24849c7e7e4f.png">
<img width="800" alt="Screenshot 2023-02-22 at 15 46 44" src="https://user-images.githubusercontent.com/104728608/220776307-baed37eb-1e62-488e-840c-363688bbceb3.png">
<img width="800" alt="Screenshot 2023-02-22 at 15 44 30" src="https://user-images.githubusercontent.com/104728608/220776309-f6091d7f-9d44-4069-872f-427af098c9f5.png">

<img width="1024" alt="Screenshot 2023-02-22 at 16 03 00" src="https://user-images.githubusercontent.com/104728608/220776317-e6bb667a-7dc6-4f7f-9544-9641fe10e086.png">

<br>

## Set up AWS CloudWatch:

<br>

Create the log groups and dashboards and start monitoring:

<br>


<img width="800" alt="Screenshot 2023-02-23 at 13 26 54" src="https://user-images.githubusercontent.com/104728608/220920574-ec4bdd8e-f049-4550-abdb-68e0c2816d89.png">

<img width="800" alt="Screenshot 2023-02-22 at 23 24 02" src="https://user-images.githubusercontent.com/104728608/220786281-1cfdb499-0c22-45d6-9721-85f43d308217.png">

<img width="800" alt="Screenshot 2023-02-22 at 22 51 09" src="https://user-images.githubusercontent.com/104728608/220778614-9de1ab07-db85-4865-997c-2ffe4ce4c51b.png">
<img width="800" alt="Screenshot 2023-02-22 at 22 56 10" src="https://user-images.githubusercontent.com/104728608/220782101-fcac7808-53ad-484f-81f5-2ef7460aebfa.png">
<img width="800" alt="Screenshot 2023-02-22 at 22 56 57" src="https://user-images.githubusercontent.com/104728608/220782133-654ee0a7-cc6b-45cc-b0c6-3317da39065f.png">


<img width="800" alt="Screenshot 2023-02-22 at 22 26 39" src="https://user-images.githubusercontent.com/104728608/220776238-646d4958-9d7c-47bd-8c57-84cc40cd5680.png">
<img width="800" alt="Screenshot 2023-02-22 at 17 14 05" src="https://user-images.githubusercontent.com/104728608/220776248-c4874465-5852-4fcf-a305-208ecd2becc8.png">

<img width="800" alt="Screenshot 2023-02-22 at 22 28 50" src="https://user-images.githubusercontent.com/104728608/220776255-150e4fe7-0384-4522-bbef-66d47f28dfc7.png">
<img width="800" alt="Screenshot 2023-02-22 at 22 14 46" src="https://user-images.githubusercontent.com/104728608/220776257-99044f53-2226-48b3-a596-3c652c93643e.png">
<img width="800" alt="Screenshot 2023-02-22 at 22 30 02" src="https://user-images.githubusercontent.com/104728608/220776258-8d02d3f9-b474-4e79-86f2-7233c0bc5fa0.png">
<img width="800" alt="Screenshot 2023-02-22 at 22 24 50" src="https://user-images.githubusercontent.com/104728608/220776263-f8a3391d-43d0-4372-9192-8c43fc1a4dd3.png">

<img width="800" alt="Screenshot 2023-02-22 at 16 54 30" src="https://user-images.githubusercontent.com/104728608/220776271-25df14e6-c862-4df9-8adc-30d8a57ae449.png">
<img width="800" alt="Screenshot 2023-02-22 at 16 27 56" src="https://user-images.githubusercontent.com/104728608/220776278-68a5f21e-c4b4-4a2f-a68b-d4780f57acbc.png">

<img width="800" alt="Screenshot 2023-02-23 at 17 12 02" src="https://user-images.githubusercontent.com/104728608/220980313-8bf4a736-39bc-4fcc-bdce-681918ccaa77.png">


<img width="800" alt="Screenshot 2023-02-22 at 22 34 38" src="https://user-images.githubusercontent.com/104728608/220776269-509fe0c2-6316-4b36-af25-dfa7b4caa752.png">
