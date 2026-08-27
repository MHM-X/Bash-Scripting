# 📂 Network Protocol Analyzer
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
# Network Protocol Analyzer

#!/bin/bash

INTERFACE="$1"

if [[ -z "$INTERFACE" ]]; then
    echo "Usage: $0 <interface>"
    exit 1
fi

echo "Capturing traffic on $INTERFACE..."
echo "Press Ctrl+C to stop."
echo

sudo tcpdump -i "$INTERFACE" -n -l |
while read -r LINE; do

    if [[ "$LINE" == *" TCP "* ]]; then
        echo "TCP"

    elif [[ "$LINE" == *" UDP "* ]]; then
        echo "UDP"

    elif [[ "$LINE" == *" ICMP "* ]]; then
        echo "ICMP"

    elif [[ "$LINE" == *" ARP "* ]]; then
        echo "ARP"

    else
        echo "Other"
    fi

done
```
