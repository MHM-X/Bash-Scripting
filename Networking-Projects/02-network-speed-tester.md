# 📂 Network Speed Test
## Description
A basic server that allows many users to chat via the command line. Implement private messaging, chat rooms, and basic user authentication.

## Code
```bash
#!/bin/bash
# Network Speed Test

#!/bin/bash

LOG_FILE="$HOME/speed_history.csv"

# Create log file if it doesn't exist
if [[ ! -f "$LOG_FILE" ]]; then
    echo "date,ping,download,upload" > "$LOG_FILE"
fi

# Date
DATE=$(date '+%Y-%m-%d %H:%M:%S')

# Ping
PING=$(ping -c 4 1.1.1.1 2>/dev/null |
    awk -F'/' '/rtt|round-trip/ {print $5}')

# Download
DOWNLOAD=$(curl -s -o /dev/null \
    -w '%{speed_download}' \
    "https://speed.cloudflare.com/__down?bytes=10000000")

DOWNLOAD=$(awk -v speed="$DOWNLOAD" \
    'BEGIN {printf "%.2f", speed * 8 / 1000000}')

# Upload
UPLOAD=$(dd if=/dev/zero bs=1M count=5 2>/dev/null |
    curl -s -X POST --data-binary @- \
    -w '%{speed_upload}' \
    -o /dev/null \
    "https://speed.cloudflare.com/__up")

UPLOAD=$(awk -v speed="$UPLOAD" \
    'BEGIN {printf "%.2f", speed * 8 / 1000000}')

# Save
echo "$DATE,$PING,$DOWNLOAD,$UPLOAD" >> "$LOG_FILE"

# Display
echo "Date: $DATE"
echo "Ping: $PING ms"
echo "Download: $DOWNLOAD Mbps"
echo "Upload: $UPLOAD Mbps"
```
