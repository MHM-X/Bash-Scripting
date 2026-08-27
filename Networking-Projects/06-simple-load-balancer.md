# 📂 Simple Load Balancer
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
# Simple Load Balancer

#!/bin/bash

SERVERS=(
    "192.168.1.10:8000"
    "192.168.1.11:8000"
    "192.168.1.12:8000"
)

INDEX=0

while true; do

    SERVER="${SERVERS[$INDEX]}"

    HOST="${SERVER%:*}"
    PORT="${SERVER#*:}"

    if curl -s --max-time 2 "http://$HOST:$PORT" > /dev/null; then

        echo "Forwarding request to $SERVER"

        curl -s "http://$HOST:$PORT"

        INDEX=$(( (INDEX + 1) % ${#SERVERS[@]} ))

    else

        echo "$SERVER is DOWN"

        INDEX=$(( (INDEX + 1) % ${#SERVERS[@]} ))
    fi

    sleep 1
done
```
