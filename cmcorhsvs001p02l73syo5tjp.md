---
title: "Infra Health Monitoring Suite"
datePublished: Fri Jul 04 2025 11:59:26 GMT+0000 (Coordinated Universal Time)
cuid: cmcorhsvs001p02l73syo5tjp
slug: infra-health-monitoring-suite
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1752665898031/114c322c-0cf6-4a06-8a1b-cee33d83ed41.png
tags: docker, json, bash, slack, logging, logs, cronjob, production-ready, shell-script, log-rotation, website-health-check-tools

---

---

## Ever wondered how big tech companies ensure their containers stay healthy, websites stay up, logs stay clean, and alerts don’t spam your team - all in an automated, self-healing way?

In the real world, infrastructure monitoring isn’t just about running a few `docker ps` or pinging a website. It’s about combining smart health checks, meaningful alerts, automated log handling, and real-time status dashboards — seamlessly and reliably.

That’s exactly why I built this **complete Docker & Website Health Monitoring Suite**, fully written in shell scripting.

The goal?  
🟣Automate container and website health checks.  
🟣Send intelligent, non-spammy alerts directly to Slack.  
🟣Rotate and archive logs before they grow out of control.  
🟣 Maintain a JSON-based dashboard for clean visibility.  
🟣Schedule it all to run hands-free via cron jobs.  
🟣Combine these into one unified, production-ready toolkit you can plug into any system.

Through this project, I got to solve real DevOps challenges, simulate real-world failures, and design a system that feels like a mini in-house monitoring service — but with no extra tools, just pure Linux magic.

In this blog, I’ll break down every piece — from container health logic to JSON dashboards — in a practical, step-by-step way so you can understand, learn, and build your own.

---

**Now, let’s dive deep into the architecture, folder structure, and scripts behind this monitoring powerhouse.**

# Folder Structure

```txt
docker-health-monitor/
├── logs/
│   ├── container-health.log
│   └── site-uptime.log
├── archive/
│   └── rotated-logs/
├── config/
│   └── services.txt      # list of websites to monitor
├── scripts/
│   ├── check_docker_health.sh
│   ├── check_website_uptime.sh
│   ├── alert.sh
│   ├── log_rotate.sh
│   └── main.sh           # Entry point CLI
├── cron/
│   └── cronjob.txt       # cron entry for automation
├── README.md
└── setup.sh              # to setup project (permissions, envs)
```

---

# Create Docker Container With Health Checks

`Without this, container inspect wont produce container status`

```bash
docker run -d \
    --name my-nginx \
    --health-cmd='curl -f http://localhost:80/ || exit 1' \
    --health-interval=30s \
    --health-timeout=5s \
    --health-retries=3 \
    --health-start-period=10s \
    nginx
```

#### **Docker exerts an excellent command** `filter='{{.Name}}` -- used to fetch particular things from docker cmds

```bash
docker inspect --format='{{.Name}}' $id | cut -d'/' -f2
docker ps --format='{{.ID}} {{.Name}}'
```

## Creating Container Health Checking Script

```bash
#!/bin/bash
set -e  # if anything return other than 0 -- exit the script

echo "Checking All Containers Health......"

logs_dir="$(dirname "$0")/../logs"  # we dont want inside scripts we want in root
log_file="$logs_dir/container-health.log"

mkdir -p "$logs_dir"
  
# Function to fetch name and health check from docker inspect command
fetch_container_status(){
    for id in $(docker ps -q); do
        name=$(docker inspect --format='{{.Name}}' $id | cut -d'/' -f2)
        health=$(docker inspect --format='{{.State.Health.Status}}' $id)
        if [ $health == "healthy" ]; then
            echo -e "\e[32m$name is healthy\e[0m"
        elif [ $health == "starting" ]; then
            echo -e "\e[36m$name is starting\e[0m"
        elif [ $health == "unhealthy" ]; then
            echo -e "\e[31m$name is UNHEALTHY\e[0m"
        else
            echo -e "\e[33m$name has no health check defined\e[0m]"
        fi
    done
}

fetch_container_status

# Log output without color codes
log_output=$(fetch_container_status | sed 's/\x1b\[[0-9]*m//g') # x1b is ESC(e)
echo -e "[$(date +"%d-%m-%Y_%H:%M:%S")] Checking Containers Status...\n$log_output\n****************" >> $log_file
```

---

## Creating Website Health Check Script

### Writing Websites inside /config/services.txt

```bash
my_portfolio="daksh-portfolio.cctlds.online"
google="www.google.com"
youtube="www.youtube.com"
```

## check\_website\_[health.sh](http://health.sh)

```bash
#!/bin/bash

website_file_path="$(dirname "$0")/../config"
website_file="$website_file_path/services.txt"

log_file_path="$(dirname "$0")/../logs"
log_file="$log_file_path/website-health.log"

mkdir -p "$log_file_path"

while IFS= read -r line; do     # using IFS to read because for loop will break word into pieces
    website=$(echo "$line" | cut -d'=' -f2 | sed 's/"//g' | tr -d '\r')
    if [ -n "$website" ]; then
        status_code=$(curl -Is --max-time 5 "https://$website" | head -n 1 | awk '{print $2}')
        if [[ $status_code -eq 200 ]]; then
            echo -e "\e[32m$website is UP (HTTP $status_code)\e[0m"
        else
            echo -e "\e[31m$website is DOWN (HTTP $status_code)\e[0m"
        fi

        # Sending Text to Log File
        {
            echo "$(date +"%d-%m-%Y_%H:%M:%S") ---- $website returned HTTP $status_code"
        } >> "$log_file"
    fi

done < "$website_file"
echo -e "\e[35mWebsite check completed. Logs saved to: $log_file\e[0m"
echo "===========================================" >> "$log_file"
# echo "$website_file"  # Uncomment for debugging: prints the path to the services.txt file
```

---

## Creating Alert is site is down and sending to slack

### Fetch the Webhook from Slack

### **Create Webhook**

#### For Slack:

1. Go to: [https://api.slack.com/apps](https://api.slack.com/apps)
    
2. Create new app → Add "Incoming Webhooks"
    
3. Enable Webhooks → Create a Webhook URL for your channel
    
4. Copy the webhook URL — we’ll store it securely
    

### **Store Webhook URL in Config**

Create a file: `config/webhook.env`

Add: `SLACK_WEBHOOK_URL="`[`https://hooks.slack.com/services/xxxx/yyyy/zzzz`](https://hooks.slack.com/services/xxxx/yyyy/zzzz)`"`

## Building [alert.sh](http://alert.sh) script

```bash
#!/bin/bash

# fetching Slack Webhook URL from ../config/webhook.env
source "$(dirname "$0")/../config/webhook.env"

# We gonna pass this aler.sh script in webhook so passing things through arg
severity="$1"   # Expected values: "critical", "warning", "info"
message="$2"    # message we wanted to send to slack

# payload accepts json format #! \" means " is treated as " not simple text
payload="{
    \"text\": \"[$severity] $message\"      
}"

# Check if SLACK_WEBHOOK_URL is set and not empty
if [ -z "$SLACK_WEBHOOK_URL" ]; then
    echo "Error: SLACK_WEBHOOK_URL is not set. Please check ../config/webhook.env."
    exit 1
fi

# Sending payload to slack via curl
# -s=silent -X=HTTP_Method_Specify -H=Content_type
curl -s -X POST -H 'Content-type: application/json' \
    --data "$payload" "$SLACK_WEBHOOK_URL"
```

### Add this into website\_uptime\_script

```bash
echo "Triggering alert for $website with status code $status_code"
# Run this script
bash "$(dirname "$0")/alert.sh" "CRITICAL" "Website is DOWN: $website returned HTTP ${status_code:-404}"
```

---

# Only send alert once per X mins per issue --- Avoid spamming channels

If a website stays down for 30 minutes, and your cron runs every 5 minutes — you’ll get `6 identical alerts`. `In a game, if a player keeps getting hit, you don’t want to spam "YOU TOOK DAMAGE" every millisecond`

### Approach: "Last Alert Cache"

1️⃣ When you send an alert, you **record** that you sent it (e.g., to a small file per website/container).  
2️⃣ Before sending a new alert, you **check** if there’s already a file indicating an active alert.  
3️⃣ If yes → **skip sending**.  
4️⃣ If service becomes healthy again → **delete the alert flag file**.

```bash
alert_path="$(dirname "$0")/../alert_flags" # Alert Flags Folder Path - which contain flag for down site
flag_file="$alert_path/$(echo "$website" | md5sum | awk '{print $1}').flag"

        if [[ $status_code -eq 200 ]]; then
            echo -e "\e[32m$website is UP (HTTP $status_code)\e[0m"

            # if flag is present, remove it because now website is up
            if [ -f "$flag_file" ];then
                # echo "Remove the flag"
                rm -rf "$flag_file"
            fi

        else
            echo -e "\e[31m$website is DOWN (HTTP ${status_code:-404})\e[0m"

            # If Flag exists do nothing and if not -- create and send notif to slack
            if [ ! -f "$flag_file" ]; then
                # echo "Send Notification to Slack" echo "Create Flag"
                touch "$flag_file"
                echo -e "\nTriggering alert for $website with status code ${status_code:-404}\n"
                # :- means send status code and if it is not present send 404 as status code
                bash "$(dirname "$0")/alert.sh" "CRITICAL" "Website is DOWN: $website returned HTTP ${status_code:-404}"
            fi
```

---

# **Log Rotation and Archiving**

Prevent logs from growing infinitely large, avoid disk overflow, and keep old logs archived neatly.

> Create an archive\_file if Log\_File Size &gt; (Set Size), then send data to archive folder with timestamp Make a new log file

```bash
#!/bin/bash

log_file_path="$(dirname "$0")/../logs"
log_file="$log_file_path/website-health.log"


time_stamp=$(date +"%d-%m-%Y_%H-%M-%S")

archive_path="$(dirname "$0")/../logs/website-archive-path"
archive_file="$archive_path/"$time_stamp.log""

mkdir -p "$archive_path"

file_size=$(du -k "$log_file" | cut -f1)

# if file size exceeds 15KB, rotate it
if [[ "$file_size" -gt 15 ]]; then
    echo "LogFile got above the size limit. Log File Rotated"
    mv "$log_file" "$archive_file"
    touch "$log_file"
    echo "New log file created at: $log_file"
fi

echo "$file_size KB"
```

---

# Adding Cron Job

```bash
crontab -e
```

### Add these scripts as cronjob

```bash
* * * * * /bin/bash /mnt/c/Studies/Linux/docker-health-monitor/scripts/log_rotate.sh
* * * * * /bin/bash /mnt/c/Studies/Linux/docker-health-monitor/scripts/check_website_uptime.sh
```

---

# Sending Heart Beat to slack stating cron job is working

```bash
#!/bin/bash

source "$(dirname "$0")/../config/webhook.env"

message="Heartbeat: All cron jobs are running fine at $(date +"%d-%m-%Y %H:%M:%S")."

payload="{
    \"text\": \"$message\"
}"

curl -s -X POST -H 'Content-type: application/json' \
    --data "$payload" "$SLACK_WEBHOOK_URL" &> /dev/null
```

---

# Creating JSON Log File

```bash
#!/bin/bash

json_file_path="$(dirname "$0")/../logs/json_logs"
json_file="$json_file_path/logs.json"

mkdir -p "$json_file_path"
if [ ! -f "$json_file" ]; then
    touch "$json_file"
fi

name="$1"
status="$2"
http_status="$3"
latency="$4"

# Creating JSON snippet
service_entry="{
    \"Name\": \"$name\",
    \"Status\": \"$status\",
    \"HTTP_Status\": \"$http_status\",
    \"Latency\": \"$latency\"
}"

# Check if json_file is not empty - then append and if it is, append {} into mt file
if [ ! -s "$json_file" ]; then
    echo "{
        \"last_checked\": \"$(date +"%d-%m-%Y %H:%M:%S")\",
        \"services\": [
            $service_entry
        ]
    }" > "$json_file"
else
    # File is not empty, we need to append new service
    # We read old content (excluding last closing brackets)
    # Creating a temp json file for doing operations in it
    temp_file="$json_file_path/temp.json"
    touch "$temp_file"

    #! Remove Last 2 lines from the file and send all to temp to append other
    head -n -2 "$json_file" > "$temp_file"

    # Append New Service
    echo ",$service_entry" >> "$temp_file"
    echo "      ]" >> "$temp_file"
    echo "}" >> "$temp_file"

    # Move the file to $json_file
    mv "$temp_file" "$json_file"
fi
```

---

# Adding Dynamic Changing of Date in JSON Log file using SED

```bash
# Changing Or Appending new date using sed into json file
sed -i "s/\"last_checked\":\".*\"/\"last_checked\":\"$(date +"%d-%m-%Y_%H:%M:%S")\"/" "$json_file"
echo "Date Changed"
```

---

# Visualizing Data using JQ with help of json log file

```bash
#!/bin/bash

json_file="$(dirname "$0")/../logs/json_logs/logs.json"

if [ ! -f "$json_file" ]; then
    echo "Create JSON File First"
    exit 1
fi

last_checked=$(jq -r '.last_checked' "$json_file")
echo -e "\nLast Checked: $last_checked\n"
echo "--- Services Dashboard --\n"

# Storing JSON Objects Items as variables
jq -c '.services[]' "$json_file" | while read -r line; do
    name=$(echo "$line" | jq -r '.Name')
    status=$(echo "$line" | jq -r '.Status')
    http_status=$(echo "$line" | jq -r '.HTTP_Status')
    latency=$(echo "$line" | jq -r '.Latency')

echo -e "Name: $name\nStatus: $status\nHTTP Status: $http_status\nLatency: $latency seconds\n---------------------\n"
done
```