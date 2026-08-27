# 📂 Code Comment Extractor
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
# Code Comment Extractor

#!/bin/bash

set -euo pipefail

if [[ $# -eq 0 ]]; then
    echo "Usage: $0 <source_file> [source_file ...]"
    exit 1
fi

extract_comments() {
    local file="$1"
    local ext="${file##*.}"

    echo "========================================"
    echo "File: $file"
    echo "========================================"

    case "$ext" in

        # C, C++, Java, JavaScript, TypeScript, Dart, Go, Rust
        c|h|cpp|cc|cxx|hpp|java|js|ts|dart|go|rs)
            awk '
            {
                line = $0

                # Single-line comments
                if (match(line, /\/\/.*/)) {
                    print substr(line, RSTART)
                }

                # Start of multi-line comment
                if (line ~ /\/\*/) {
                    in_comment = 1
                    sub(/^.*\/\*/, "/*", line)
                }

                # Multi-line comment content
                if (in_comment) {
                    if (line ~ /\*\//) {
                        print line
                        in_comment = 0
                    }
                    else if (line !~ /\/\*/) {
                        print line
                    }
                }
            }' "$file"
            ;;

        # Python, Bash, Shell, Ruby, Perl
        py|sh|bash|zsh|rb|pl)
            grep -E '^[[:space:]]*#|[[:space:]]+#' "$file" || true
            ;;

        # SQL
        sql)
            awk '
            {
                if ($0 ~ /--/) {
                    sub(/^.*--/, "--")
                    print
                }
            }' "$file"
            ;;

        # HTML / XML
        html|htm|xml)
            awk '
            {
                if ($0 ~ /<!--/) {
                    in_comment = 1
                }

                if (in_comment) {
                    print
                }

                if ($0 ~ /-->/) {
                    in_comment = 0
                }
            }' "$file"
            ;;

        *)
            echo "Unsupported file type: .$ext"
            ;;
    esac

    echo
}

for file in "$@"; do

    if [[ ! -f "$file" ]]; then
        echo "Error: '$file' is not a file."
        continue
    fi

    extract_comments "$file"

done
```
