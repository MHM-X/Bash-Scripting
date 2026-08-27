# 📂 Website Availability Checker
## Description
A tool to monitor websites and alert them if they become unavailable. Include response time tracking and support for checking specific page elements or content.
## Code
```bash
#!/bin/bash
# Website Availability Checker


URL="$1"

if [[ -z "$URL" ]]; then
    echo "Usage: $0 <URL>"
    exit 1
fi

echo "Checking: $URL"
echo "-------------------------"

RESULT=$(curl -s -o /dev/null -w "%{http_code} %{time_total}" \
    --max-time 10 "$URL")

STATUS=$(echo "$RESULT" | awk '{print $1}')
TIME=$(echo "$RESULT" | awk '{print $2}')

if [[ "$STATUS" == "200" ]]; then
    echo "Status: UP"
    echo "HTTP Code: $STATUS"
    echo "Response Time: ${TIME}s"
else
    echo "Status: DOWN"
    echo "HTTP Code: $STATUS"
    echo "Response Time: ${TIME}s"
fi
```
