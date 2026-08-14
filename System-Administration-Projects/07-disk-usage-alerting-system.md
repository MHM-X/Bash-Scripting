# 📂 Disk Usage Alerting System
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/bin/bash

# Disk Usage Alerting System

# ---------- Configuration ----------

THRESHOLD=80

CHECK_INTERVAL=60

LOG_FILE="/var/log/disk-alert.log"

EMAIL="admin@example.com"

FILESYSTEMS=(
    "/"
    "/home"
)


# ---------- Logging ----------

log_message() {

    local message="$1"

    echo "$(date '+%Y-%m-%d %H:%M:%S') - $message" \
        | tee -a "$LOG_FILE"
}


# ---------- Check Disk Usage ----------

get_disk_usage() {

    local filesystem="$1"

    df -P "$filesystem" |
        awk 'NR==2 {gsub("%", "", $5); print $5}'
}


# ---------- Send Alert ----------

send_alert() {

    local filesystem="$1"
    local usage="$2"

    local subject="Disk Usage Alert: $filesystem"
    local message="WARNING: Disk usage on $filesystem is ${usage}%."

    log_message "$message"

    # Email notification
    mail -s "$subject" "$EMAIL" <<< "$message"
}


# ---------- Monitor Filesystem ----------

monitor_filesystem() {

    local filesystem="$1"

    local usage

    usage=$(get_disk_usage "$filesystem")


    if [[ -z "$usage" ]]; then

        log_message "ERROR: Could not determine disk usage for $filesystem."

        return 1

    fi


    log_message "Filesystem '$filesystem' usage: ${usage}%."


    if (( usage >= THRESHOLD )); then

        log_message \
            "WARNING: '$filesystem' exceeded threshold of ${THRESHOLD}%."

        send_alert "$filesystem" "$usage"

    fi
}


# ---------- Main ----------

main() {

    if [[ $EUID -ne 0 ]]; then

        echo "Error: This script must be run as root."

        exit 1

    fi


    log_message "Disk monitoring started."


    while true; do

        for filesystem in "${FILESYSTEMS[@]}"; do

            monitor_filesystem "$filesystem"

        done

        sleep "$CHECK_INTERVAL"

    done
}


main
```
