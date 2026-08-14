# 📂 Log File Analyzer
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/bin/bash

log_file="${1:-sample.log}"

if [[ ! -f "$log_file" ]]; then
    echo "Error: Log file not found: $log_file"
    exit 1
fi

total_lines=$(wc -l < "$log_file")
errors=$(grep -c "ERROR" "$log_file")
warnings=$(grep -c "WARNING" "$log_file")

echo "=============================="
echo "       LOG FILE ANALYZER"
echo "=============================="

echo "Log file       : $log_file"
echo "Total entries  : $total_lines"
echo "Errors         : $errors"
echo "Warnings       : $warnings"

echo
echo "Top IP Addresses"
echo "----------------"

grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' "$log_file" |
    sort |
    uniq -c |
    sort -nr |
    head -5

echo
echo "Error Messages"
echo "--------------"

grep "ERROR" "$log_file"

echo
echo "Security Events"
echo "---------------"

grep -Ei "failed|authentication|login" "$log_file"
```
