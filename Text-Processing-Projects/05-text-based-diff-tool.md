# 📂 Text-based Diff tool
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/bin/bash

set -euo pipefail

RED='\033[31m'
GREEN='\033[32m'
RESET='\033[0m'

SIDE_BY_SIDE=false

usage() {
    echo "Usage: $0 [-s] file1 file2"
    echo
    echo "Options:"
    echo "  -s    Show side-by-side comparison"
    echo "  -h    Show help"
}

while getopts "sh" opt; do
    case "$opt" in
        s)
            SIDE_BY_SIDE=true
            ;;
        h)
            usage
            exit 0
            ;;
        \?)
            usage
            exit 1
            ;;
    esac
done

shift $((OPTIND - 1))

if [[ $# -ne 2 ]]; then
    echo "Error: two files are required."
    usage
    exit 1
fi

FILE1="$1"
FILE2="$2"

if [[ ! -f "$FILE1" ]]; then
    echo "Error: '$FILE1' does not exist."
    exit 1
fi

if [[ ! -f "$FILE2" ]]; then
    echo "Error: '$FILE2' does not exist."
    exit 1
fi

if [[ "$SIDE_BY_SIDE" == true ]]; then

    diff -y --width=100 "$FILE1" "$FILE2" || true

else

    diff -u "$FILE1" "$FILE2" |
    while IFS= read -r line; do

        case "$line" in
            +*)
                printf "${GREEN}%s${RESET}\n" "$line"
                ;;

            -*)
                printf "${RED}%s${RESET}\n" "$line"
                ;;

            *)
                printf "%s\n" "$line"
                ;;
        esac

    done

fi
```
