# 📂 IP Address Tracker
## Description
A script to log and track changes in your public IP address. Set up notifications for IP changes and maintain a history of past addresses. Include geolocation information for each IP.
## Code
```bash
#!/bin/bash
# IP Address Tracker

set -euo pipefail

# Configuration

LOG_FILE="$HOME/ip_tracker.log"
LAST_IP_FILE="$HOME/.last_public_ip"

IP_API="https://api.ipify.org"

# Get current public IP

CURRENT_IP=$(curl -s --fail "$IP_API")

if [[ -z "$CURRENT_IP" ]]; then
    echo "Error: Failed to get public IP."
    exit 1
fi

# Get previous IP

if [[ -f "$LAST_IP_FILE" ]]; then
    LAST_IP=$(cat "$LAST_IP_FILE")
else
    LAST_IP=""
fi

# Compare IP addresses

if [[ "$CURRENT_IP" == "$LAST_IP" ]]; then
    echo "IP has not changed: $CURRENT_IP"
    exit 0
fi

# Get geolocation information

GEO_DATA=$(curl -s --fail "https://ipinfo.io/$CURRENT_IP/json")

if [[ -z "$GEO_DATA" ]]; then
    echo "Error: Failed to get geolocation data."
    exit 1
fi

# Extract information from JSON

COUNTRY=$(echo "$GEO_DATA" | jq -r '.country // "Unknown"')
CITY=$(echo "$GEO_DATA" | jq -r '.city // "Unknown"')
REGION=$(echo "$GEO_DATA" | jq -r '.region // "Unknown"')
LOCATION=$(echo "$GEO_DATA" | jq -r '.loc // "Unknown"')
ISP=$(echo "$GEO_DATA" | jq -r '.org // "Unknown"')

# Get current date and time

TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')

# Create log entry

LOG_ENTRY="$TIMESTAMP | IP: $CURRENT_IP | Country: $COUNTRY | City: $CITY | Region: $REGION | ISP: $ISP | Location: $LOCATION"

echo "$LOG_ENTRY" >> "$LOG_FILE"

# Save current IP

echo "$CURRENT_IP" > "$LAST_IP_FILE"

# Display information

echo "=========================================="
echo "PUBLIC IP CHANGED"
echo "=========================================="
echo "Old IP : ${LAST_IP:-None}"
echo "New IP : $CURRENT_IP"
echo "Country: $COUNTRY"
echo "City   : $CITY"
echo "Region : $REGION"
echo "ISP    : $ISP"
echo "Location: $LOCATION"
echo "Time   : $TIMESTAMP"
echo "=========================================="

# Desktop notification

if command -v notify-send >/dev/null 2>&1; then
    notify-send \
        "Public IP Changed" \
        "New IP: $CURRENT_IP\nLocation: $CITY, $COUNTRY"
fi
```
