# 📂 Batch text replacer
## Description
A program to find and replace text across multiple files. Support regular expressions and provide options for backup before making changes.

## Code
```bash
#!/bin/bash

set -euo pipefail

DIRECTORY=""
FIND_TEXT=""
REPLACE_TEXT=""
BACKUP=false
REGEX=false

usage() {
    echo "Usage: $0 -d directory -f search -r replacement [options]"
    echo
    echo "Options:"
    echo "  -d DIR    Directory to search"
    echo "  -f TEXT   Text or regex to find"
    echo "  -r TEXT   Replacement text"
    echo "  -b        Create backup before modifying files"
    echo "  -E        Treat search text as a regular expression"
    echo "  -h        Show help"
}

while getopts "d:f:r:bEh" opt; do
    case "$opt" in
        d)
            DIRECTORY="$OPTARG"
            ;;
        f)
            FIND_TEXT="$OPTARG"
            ;;
        r)
            REPLACE_TEXT="$OPTARG"
            ;;
        b)
            BACKUP=true
            ;;
        E)
            REGEX=true
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

if [[ -z "$DIRECTORY" || -z "$FIND_TEXT" ]]; then
    echo "Error: directory and search text are required."
    usage
    exit 1
fi

if [[ ! -d "$DIRECTORY" ]]; then
    echo "Error: directory '$DIRECTORY' does not exist."
    exit 1
fi

if [[ "$REGEX" == true ]]; then
    PATTERN="$FIND_TEXT"
else
    PATTERN=$(printf '%s' "$FIND_TEXT" | sed 's/[][\/.^$*+?{}()|\\]/\\&/g')

    # s / pattern / replacement / g
    # /g: استبدل كل occurrences وليس أول واحدة فقط.
    # \\&: & تعني الرمز الذي تم العثور عليه

fi

find "$DIRECTORY" -type f -print0 |
while IFS= read -r -d '' file; do

    if grep -qE "$PATTERN" "$file"; then

        echo "Updating: $file"

        if [[ "$BACKUP" == true ]]; then
            cp "$file" "$file.bak"
            echo "  Backup: $file.bak"
        fi

        sed -i -E "s|$PATTERN|$REPLACE_TEXT|g" "$file"

       # s|$PATTERN|$REPLACE_TEXT|g
  # s             → substitute
  # $PATTERN      → ماذا أبحث عنه؟
  # $REPLACE_TEXT → بماذا أستبدله؟
  #g              → استبدل جميع التطابقات

    fi

done

echo "Replacement completed."
```
