# 📂 Text-based Search Engine
## Description
A program to search for words or phrases in multiple text files. Implement ranking of results based on relevance and support for boolean search operators.
## Code
```bash
#!/bin/bash
# Text-based Search Engine

SEARCH_DIR="${1:-./documents}"

if [[ ! -d "$SEARCH_DIR" ]]; then
    echo "Directory not found: $SEARCH_DIR"
    exit 1
fi

if [[ $# -lt 2 ]]; then
    echo "Usage: $0 <directory> <search query>"
    echo "Example: $0 ./documents linux"
    echo "Example: $0 ./documents linux AND server"
    echo "Example: $0 ./documents linux OR docker"
    echo "Example: $0 ./documents linux NOT windows"
    exit 1
fi

shift

QUERY="$*"

echo "Searching for: $QUERY"
echo "Directory: $SEARCH_DIR"
echo "----------------------------------------"

# Normalize query
QUERY=$(echo "$QUERY" | tr '[:upper:]' '[:lower:]')

# Split query into words
read -ra TOKENS <<< "$QUERY"

# Find all text files
find "$SEARCH_DIR" -type f -name "*.txt" | while read -r file; do

    content=$(tr '[:upper:]' '[:lower:]' < "$file")

    score=0
    matched=true
    i=0

    while [[ $i -lt ${#TOKENS[@]} ]]; do

        token="${TOKENS[$i]}"

        case "$token" in

            # grep -qw: q: Quiet ابحث لكن لا تطبع النتيجة , w: Whole Word ابحث عن الكلمة كاملة وليس مجرد جزء منها
            # grep -o -iw: o: Only matching يطبع الـ الكلمة المشابهة للمراد البحث عنها فقط وليس السطر الي هي فيه بأكمله
            #              i: Ignore case

            and)
                ((i++))
                word="${TOKENS[$i]}"

                if ! grep -qw "$word" <<< "$content"; then
                    matched=false
                    break
                fi

                count=$(grep -o -iw "$word" <<< "$content" | wc -l)
                score=$((score + count))
                ;;

            or)
                ((i++))
                word="${TOKENS[$i]}"

                if grep -qw "$word" <<< "$content"; then
                    count=$(grep -o -iw "$word" <<< "$content" | wc -l)
                    score=$((score + count))
                fi
                ;;

            not)
                ((i++))
                word="${TOKENS[$i]}"

                if grep -qw "$word" <<< "$content"; then
                    matched=false
                    break
                fi
                ;;

            *)
                if grep -qw "$token" <<< "$content"; then
                    count=$(grep -o -iw "$token" <<< "$content" | wc -l)
                    score=$((score + count))
                else
                    matched=false
                    break
                fi
                ;;

        esac

        ((i++))
    done

    if [[ "$matched" == true && $score -gt 0 ]]; then
        echo "$score|$file"
    fi

done | sort -t'|' -k1,1nr
```
