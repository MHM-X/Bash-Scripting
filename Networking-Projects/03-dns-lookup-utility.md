# 📂 DNS Lookup Utility
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
# DNS Lookup Utility
#!/bin/bash

# Reverse DNS lookup
if [[ "$1" == "-r" ]]; then

    IP="$2"

    if [[ -z "$IP" ]]; then
        echo "Usage: $0 -r <IP>"
        exit 1
    fi

    echo "Reverse DNS Lookup"
    echo "IP: $IP"
    echo "-------------------------"

    dig -x "$IP" +short

    exit 0
fi


# Normal DNS lookup
DOMAIN="$1"
TYPE="${2:-A}"

if [[ -z "$DOMAIN" ]]; then
    echo "Usage: $0 <domain> [record_type]"
    echo "Example: $0 google.com MX"
    echo "Reverse: $0 -r 8.8.8.8"
    exit 1
fi

echo "DNS Lookup"
echo "Domain: $DOMAIN"
echo "Record: $TYPE"
echo "-------------------------"

dig "$DOMAIN" "$TYPE" +short
```
