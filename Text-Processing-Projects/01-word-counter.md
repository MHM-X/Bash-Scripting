# 📂 Word Counter
## Description
A tool to count words, lines, and characters in a text file. Include options for excluding certain words or patterns and handling multiple files.
## Code
```bash
#!/bin/bash

set -euo pipefail

exclude_patterns=()

usage() {
    echo "Usage: $0 [-e pattern1,pattern2,...] file1 [file2 ...]"
    echo
    echo "Options:"
    echo "  -e PATTERNS    Comma-separated regex patterns to exclude"
    echo "  -h             Show this help message"
    echo
    echo "Examples:"
    echo "  $0 file.txt"
    echo "  $0 -e 'the,and' file.txt"
    echo "  $0 -e '^test.*,error$' file.txt"
    exit 0
}

# Parse command-line options
while getopts ":e:h" opt; do
    case "$opt" in
        e)
            IFS=',' read -ra exclude_patterns <<< "$OPTARG"
            ;;
        h)
            usage
            ;;
        \?)
            echo "Error: Invalid option -$OPTARG" >&2
            usage
            ;;
        :)
            echo "Error: Option -$OPTARG requires an argument." >&2
            exit 1
            ;;
    esac
done

# Remove processed options
shift $((OPTIND - 1))

# Check that at least one file was provided
if [[ $# -eq 0 ]]; then
    echo "Error: No input files provided." >&2
    usage
fi

# Check whether a word matches any exclusion regex
is_excluded() {
    local word="$1"

    for pattern in "${exclude_patterns[@]}"; do
        if printf '%s\n' "$word" | grep -Eq "$pattern"; then
            return 0
        fi
    done

    return 1
}

# Process every file
for file in "$@"; do

    if [[ ! -f "$file" ]]; then
        echo "Error: '$file' is not a regular file." >&2
        continue
    fi

    lines=$(wc -l < "$file")
    characters=$(wc -m < "$file")
    words=0

    while IFS= read -r line || [[ -n "$line" ]]; do

        for word in $line; do


            # معناه:
            # s → substitute، استبدال
            # ^ → بداية السطر
            # [[:punct:]] → أي علامة ترقيم
            # * → صفر أو أكثر
            # الجزء الأخير الفارغ // → احذف ما وجدناه
            # الـ ; يفصل أمرين sed.
            # Remove punctuation from beginning and end
            word=$(printf '%s\n' "$word" | sed 's/^[[:punct:]]*//; s/[[:punct:]]*$//')

            # or: using awk
            word=$(awk '{gsub(/^[[:punct:]]+|[[:punct:]]+$/, ""); print}' <<< "$word")

            # Skip empty strings
            [[ -z "$word" ]] && continue

            # Count only words that do not match exclusion patterns
            if ! is_excluded "$word"; then
                ((words++))
            fi

        done

    done < "$file"

    echo "File: $file"
    echo "Lines: $lines"
    echo "Words: $words"
    echo "Characters: $characters"
    echo "-------------------------"

done
```
