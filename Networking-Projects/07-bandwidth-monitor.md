# 📂 Bandwidth Usage Monitor
## Description
A script to track and report on network bandwidth usage. Break down usage by application or user, and set up alerts for unusual activity.
## Code
```bash
# Bandwidth Usage Monitor

#!/bin/bash

INTERFACE="$1"

if [[ -z "$INTERFACE" ]]; then
    echo "Usage: $0 <interface>"
    echo "Example: $0 eth0"
    exit 1
fi

RX_FILE="/sys/class/net/$INTERFACE/statistics/rx_bytes"
TX_FILE="/sys/class/net/$INTERFACE/statistics/tx_bytes"

if [[ ! -f "$RX_FILE" ]]; then
    echo "Interface not found: $INTERFACE"
    exit 1
fi

START_RX=$(cat "$RX_FILE")
START_TX=$(cat "$TX_FILE")

echo "Monitoring $INTERFACE..."
echo "Press Ctrl+C to stop."

while true; do

    sleep 1

    CURRENT_RX=$(cat "$RX_FILE")
    CURRENT_TX=$(cat "$TX_FILE")

    RX=$((CURRENT_RX - START_RX))
    TX=$((CURRENT_TX - START_TX))

    RX_MB=$(awk -v bytes="$RX" 'BEGIN {printf "%.2f", bytes / 1024 / 1024}')
    TX_MB=$(awk -v bytes="$TX" 'BEGIN {printf "%.2f", bytes / 1024 / 1024}')

    echo "Received: ${RX_MB} MB | Sent: ${TX_MB} MB"

done
```
