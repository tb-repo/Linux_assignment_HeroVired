# Linux Assignment for HeroVired Batch16A - Owned by Thiagarajan Baskarasubramanian - ID 12950

**Repository:** Linux Assignment  
**Environment:** Ubuntu Linux (WSL)

---

# Project Overview

This project demonstrates the implementation of a **secure, monitored, and automated Linux development environment** for two developers: **Sarah** and **Mike**.

The objective of this assignment is to simulate real-world **DevOps operational practices** including:

- System monitoring
- User management and access control
- Password policy enforcement
- Automated backup configuration
- Backup validation and logging
- Scheduled automation using cron

The environment ensures **system reliability, security, and operational efficiency**.

---

# Task 1: System Monitoring Setup

## Objective

Configure monitoring tools to track system health and resource utilization including:

- CPU usage
- Memory usage
- Process activity
- Disk usage

This enables **performance troubleshooting and capacity planning**.

---

# Installing Monitoring Tools

## Install htop and nmon

```bash
sudo apt update

sudo apt install htop -y
```
<img src="images/htop_install.png">

```bash
sudo apt install nmon -y
```
<img src="images/nmon_install.png">

# Verifying Monitoring Tools
## htop Monitoring
```bash
htop
```
<img src="images/htop.png">

## nmon Monitoring
```bash
nmon
```
<img src="images/nmon1.png">
<img src="images/nmon2.png">
<img src="images/nmon3.png">
<img src="images/nmon4.png">

# Disk Monitoring
## Check Disk Usage
```bash
df -kah
```
<img src="images/disk_space_available.png">

## Check Directory Usage
```bash
du -sh /home /opt /var /etc /usr
```
<img src="images/disk_space_usage.png">

# Monitoring Script
## Script Details
The script collects:
Server Uptime and Load average
Compute statistics
Top resource-consuming processes
Disk usage

```bash
sudo nano /usr/local/bin/system_monitor.sh

```bash
#!/bin/bash

LOG_DIR="/var/log/system_monitor"
DATE=$(date "+%Y-%m-%d-%Hh-%Mm")
LOG_FILE="$LOG_DIR/monitor_${DATE}.log"

echo "===================================================" >> $LOG_FILE
echo "SYSTEM MONITORING REPORT - $DATE" >> $LOG_FILE
echo "===================================================" >> $LOG_FILE

# ---------------------------------------------------
# Uptime and System Load
# ---------------------------------------------------

echo "" >> $LOG_FILE
echo "SYSTEM UPTIME AND LOAD AVERAGE:" >> $LOG_FILE
uptime >> $LOG_FILE

# ---------------------------------------------------
# nmon Performance Snapshot
# ---------------------------------------------------

echo "" >> $LOG_FILE
echo "Capturing nmon performance snapshot..." >> $LOG_FILE

nmon -f -s 1 -c 5 -m $LOG_DIR >/dev/null 2>&1

echo "nmon capture saved in $LOG_DIR" >> $LOG_FILE

# ---------------------------------------------------
# Top Processes
# ---------------------------------------------------

echo "" >> $LOG_FILE
echo "TOP CPU CONSUMING PROCESSES:" >> $LOG_FILE
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head -n 6 >> $LOG_FILE

echo "" >> $LOG_FILE
echo "TOP MEMORY CONSUMING PROCESSES:" >> $LOG_FILE
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%mem | head -n 6 >> $LOG_FILE

# ---------------------------------------------------
# Memory Usage
# ---------------------------------------------------

echo "" >> $LOG_FILE
echo "MEMORY DETAILS AND AVAILABILITY:" >> $LOG_FILE
free -h >> $LOG_FILE

# ---------------------------------------------------
# Disk Usage
# ---------------------------------------------------

echo "" >> $LOG_FILE
echo "DISK USAGE:" >> $LOG_FILE
df -kah >> $LOG_FILE

# ---------------------------------------------------
# Home Directory Disk Consumption
# ---------------------------------------------------

echo "" >> $LOG_FILE
echo "HOME DIRECTORY USAGE:" >> $LOG_FILE
du -sh /home/* 2>/dev/null >> $LOG_FILE

echo "" >> $LOG_FILE
echo "MONITORING CHECK COMPLETED" >> $LOG_FILE
echo "" >> $LOG_FILE
```
<img src="images/system_monitor_ss1.png">
<img src="images/system_monitor_ss2.png">
<img src="images/system_monitor_ss3.png">
<img src="images/system_monitor_ss4.png">

# Task 2: User Management and Access Control

## Objective

Create secure system access for two developers:
Sarah
Mike
Each user is assigned an isolated workspace directory

---

## Create User Accounts

```bash
sudo useradd -m sarah
sudo useradd -m mike
```

## Validate the User Home Directories and /etc/passwd File

```bash
cd home
ls -lrth
cat /etc/passwd | grep sarah
cat /etc/passwd | grep mike
```
<img src="images/T2_user_ss1.png">

## Create Workspace Directories and Set Directory Ownership and Permissions

```bash
sudo mkdir -p /home/sarah/workspace
sudo mkdir -p /home/mike/workspace
sudo chown -R sarah:sarah /home/sarah
sudo chowm -R mike:mike /home/mike
sudo chmod 700 /home/sarah/workspace
sudo chmod 700 /home/mike/workspace
```

<img src="images/T2_user_ss2.png">

## Configure Password Complexity and Expiration Policy

Check and install libpam-pwquality package that helps us to configure password complexity for Linux users in Ubuntu:

```bash
dpkg -l | grep libpam-pwquality
```

If the above package is not installed, no results will be shown. Please install it first:

```bash
sudo apt install libpam-pwquality -y
```

<img src="images/T2_user_ss3.png">
<img src="images/T2_user_ss4.png">
<img src="images/T2_user_ss6.png">
<img src="images/T2_user_ss7.png">

## Set the required Password Complexity

This can be set on /etc/security/pwquality.conf and /etc/pam.d/common-password configuration files. For some reason the setting on /etc/security/pwquality.conf did not work for me.

<img src="images/T2_user_ss8.png">
<img src="images/T2_user_ss9.png">

These rules require passwords to include:\
Minimum length of 12 characters\
At least one digit\
One uppercase letter\
One lowercase letter\
One special character\
Only 3 retry attempts allowed before password change fails

After setting the configuration below on /etc/pam.d/common-password the policy worked

<img src="images/T2_user_ss10.png">
<img src="images/T2_user_ss11.png">

## Password Expiration Policy

Passwords are configured to expire every 30 days.

```bash
sudo chage -M 30 sarah
sudo chage -M 30 mike
```
<img src="images/T2_user_ss12.png">

## Verification of the Password Expiration Policy

```bash
sudo chage -l sarah
sudo chage -l mike
```
<img src="images/T2_user_ss13.png">

# Task 3: Backup Configuration for Web Servers

## Objective

Automate backups for the web servers managed by Sarah and Mike.

| Developer | Web Server | Config Directory | Document Root         |
| --------- | ---------- | ---------------- | --------------------- |
| Sarah     | Apache     | /etc/apache2     | /var/www/html         |
| Mike      | Nginx      | /etc/nginx       | /usr/share/nginx/html |


Backups are stored in: /backups directory

Logs are stored in: /backups/logsdirectory

## Install Apache and Nginx (Not mandatory for assignment)

```bash
sudo apt install nginx -y
sudo apt install apache2 -y
```
<img src="images/T3_backup_ss1.png">
<img src="images/T3_backup_ss2.png">

## Identify the required Directories to be Backed up

<img src="images/T3_backup_ss3.png">
<img src="images/T3_backup_ss4.png">

## Create the required backup and log folders

```bash
cd /
sudo mkdir backups
sudo mkdir -p backups/logs
```
<img src="images/T3_backup_ss5.png">

## Create the Apache and Nginx backup scripts

```bash
sudo nano /usr/local/bin/apache_backup.sh

#!/bin/bash

DATE=$(date +%F)
BACKUP_FILE="/backups/apache_backup_${DATE}.tar.gz"
LOG_FILE="/backups/logs/apache_backup_log_${DATE}.txt"
RECIPIENTS="admin@techcorp.com rahul@techcorp.com sarah@techcorp.com"

echo "Backup process started at $(date)" >> ${LOG_FILE}

tar -czvf "$BACKUP_FILE" /etc/apache2 /var/www/html >> ${LOG_FILE} 2>&1
TAR_STATUS=$?

if [ "$TAR_STATUS" -eq 0 ] && [ -f "$BACKUP_FILE" ];
then
    echo "Backup file created successfully: ${BACKUP_FILE}" >> ${LOG_FILE}

    echo "Verifying backup contents:" >> ${LOG_FILE}
    tar -tzf "$BACKUP_FILE" >> ${LOG_FILE} 2>&1

else
    echo "ERROR!!! Backup file was not created. Please fix the reported issues and rerun the job..." >> ${LOG_FILE}
fi

echo "Apache backup completed at $(date)" >> ${LOG_FILE}

mail -s "Apache Backup Status ${DATE}" ${RECIPIENTS} < ${LOG_FILE}
```
<img src="images/T3_backup_ss7.png">

```bash
sudo nano /usr/local/bin/nginx_backup.sh

#!/bin/bash

DATE=$(date +%F)
BACKUP_FILE="/backups/nginx_backup_${DATE}.tar.gz"
LOG_FILE="/backups/logs/nginx_backup_log_${DATE}.txt"
RECIPIENTS="admin@techcorp.com rahul@techcorp.com mike@techcorp.com"

echo "Backup process started at $(date)" >> ${LOG_FILE}

tar -czvf "$BACKUP_FILE" /etc/nginx /usr/share/nginx/html >> ${LOG_FILE} 2>&1
TAR_STATUS=$?

if [ "$TAR_STATUS" -eq 0 ] && [ -f "$BACKUP_FILE" ];
then
    echo "Backup file created successfully: ${BACKUP_FILE}" >> ${LOG_FILE}

    echo "Verifying backup contents:" >> ${LOG_FILE}
    tar -tzf "$BACKUP_FILE" >> ${LOG_FILE} 2>&1

else
    echo "ERROR!!! Backup file was not created. Please fix the reported issues and rerun the job..." >> ${LOG_FILE}
fi

echo "Nginx backup completed at $(date)" >> ${LOG_FILE}

mail -s "Nginx Backup Status ${DATE}" ${RECIPIENTS} < ${LOG_FILE}
```
<img src="images/T3_backup_ss8.png">

Set the required script permissions for it to be executed and scheduled:

```bash
cd /usr/local/bin
sudo chmod +x apache_backup.sh
sudo chmod +x nginx_backup.sh
```
<img src="images/T3_backup_ss9.png">

## Test Executing the Scripts

Make sure that the /backup and /backup/logs folder are having right ownership and permissions for execution and scheduling.

Test running the scripts before scheduling:

```bash
cd /usr/local/bin
sh apache_backup.sh
sh nginx_backup.sh
```
<img src="images/T3_backup_ss10.png">

Validate the log files:

```bash
cd /backup/logs
ls
cat apache_backup_log_<date>.log
cat nginx_backup_log_<date>.log
```

<img src="images/T3_backup_ss11.png">
<img src="images/T3_backup_ss12.png">

## Cron Job Scheduling

Backups are required to be scheduled to run every Tuesday at 12 AM. To do this, setup the Crontab on the Ubuntu server:

```bash
crontab -e
```
<img src="images/T3_backup_ss13.png">

Schedule the job on the Crontab as below:

```bash
0 0 * * 2 /usr/local/bin/apache_backup.sh
0 0 * * 2 /usr/local/bin/nginx_backup.sh
```
<img src="images/T3_backup_ss14.png">

0 0 * * 2\
│ │ │ │ │\
│ │ │ │ └── Tuesday\
│ │ │ └──── Every month\
│ │ └────── Every day\
│ └──────── Hour (0)\
└────────── Minute (0)

Save the changes.

Validate the Crontab entries:

```bash
crontab -l
```
<img src="images/T3_backup_ss15.png">

## Verify the backup files

First list all the backup files:

```bash
ls /backups
```

Run the below command to manually verify the backup contents:

```bash
tar -tzf /backups/apache_backup_<date>.tar.gz
tar -tzf /backups/nginx_backup_<date>.tar.gz
```

<img src="images/T3_backup_ss16.png">
<img src="images/T3_backup_ss17.png">
<img src="images/T3_backup_ss18.png">

If there are no errors on the output, we can confirm that the script is working fine and the backups taken are valid.

# Challenges Encountered

a. Permission issues creating directories and running chmod, chown commands\
b. Identifying the right package and steps for setting password policies\
c. Identifying the right package for installing Apache

# Conclusion

The development environment was successfully configured with:\

System monitoring tools (htop, nmon)\
Secure user management\
Automated backup system\
Scheduled cron jobs\
Backup verification logs\

This setup ensures:\
Improved system visibility\
Secure developer access\
Reliable disaster recovery mechanisms\
Operational stability