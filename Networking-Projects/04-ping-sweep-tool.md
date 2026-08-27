# 📂 Ping Sweep Tool
## Description
A program to check which IP addresses in a range are active. Allow customization of ping parameters and provide options for parallel scanning to improve speed.

## Code
```bash
# Ping Sweep Tool

#!/bin/bash

NETWORK="$1"

if [[ -z "$NETWORK" ]]; then
    echo "Usage: $0 <network>"
    exit 1
fi

echo "Scanning $NETWORK.1 - $NETWORK.254"
echo "-------------------------"

for i in {1..254}; do
    IP="$NETWORK.$i"

    (
        if ping -c 1 -W 1 "$IP" > /dev/null 2>&1; then
            echo "$IP is UP"
        fi
    ) &
done

wait
```
