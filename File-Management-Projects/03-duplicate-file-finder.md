# 📂 Duplicate File Finder

## Description
A script to find and list duplicate files in a folder. Use file size and content comparison to spot duplicates and offer choices to delete or move them.
## Code
```bash
#!/bin/bash
set -euo pipefail

echo "========== Duplicate File Finder =========="
echo
read -rp "Enter directory to scan: " DIR

if [[ ! -d "$DIR" ]]; then
    echo "Directory does not exist."
    exit 1
fi

mkdir -p "$DIR/duplicates"
declare -A  HASHES

echo
echo "Scanning..."
echo

while IFS= read -r -d '' file; do

    hash=$(sha256sum "$file" | awk '{print $1}')
    if [[ -n "${HASHES[$hash]+x}" ]]; then
        original="${HASHES[$hash]}"

        echo "=========================================="
        echo "Duplicate Found"
        echo
        echo "Original : $original"
        echo "Duplicate: $file"
        echo
        echo "Choose an action:"
        echo "  d) Delete duplicate"
        echo "  m) Move duplicate"
        echo "  s) Skip"
        echo "  i) Show file information"
        echo "  q) Quit"
        echo


            while true; do
        read -rp "Choose: " CHOOSE
        case "$CHOOSE" in
            d)
                rm -f "$file"
                echo "Deleted: $file"
                break
                ;;
            m)
                mv "$file" "$DIR/duplicates/"
                echo "Moved: $file -> $DIR/duplicates/"
                break
                ;;
            s)
                echo "Skipped: $file"
                break
                ;;
            i)
                    echo
                    echo "---------- Original ----------"
                    ls -lh "$original"
                    echo
                    echo "---------- Duplicate ---------"
                    ls -lh "$file"
                    echo
                    ;;
            q)
                echo "Quitting..."
                exit 0
                ;;
            *)
                echo "Invalid choice. Try again."
                ;;
        esac
    done
else
    HASHES[$hash]="$file"
fi
done < <(find "$DIR" -type f ! -path "$DIR/duplicates/*" -print0)

echo
echo "========== Scan Complete =========="
echo "All duplicates processed."
```
