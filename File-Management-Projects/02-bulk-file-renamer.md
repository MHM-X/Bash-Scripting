# 📂 Bulk File Renamer

## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/usr/bin/env bash

# Bulk File Renamer

set -Eeuo pipefail

readonly SCRIPT_NAME="$(basename "$0")"

log_info() {
    printf "[INFO] %s\n" "$1"
}

log_error() {
    printf "[ERROR] %s\n" "$1" >&2
}

usage() {
    printf "Usage:\n"
    printf "  %s <directory> <prefix|suffix|counter|date> <value>\n" "$SCRIPT_NAME"
    printf "\n"
    printf "Examples:\n"
    printf "  %s ./photos prefix backup_\n" "$SCRIPT_NAME"
    printf "  %s ./photos suffix _old\n" "$SCRIPT_NAME"
    printf "  %s ./photos counter file_\n" "$SCRIPT_NAME"
    printf "  %s ./photos date backup_\n" "$SCRIPT_NAME"
}

check_directory() {
    local directory="$1"

    if [[ ! -d "$directory" ]]; then
        log_error "Directory does not exist: $directory"
        exit 1
    fi
}

rename_with_prefix() {
    local directory="$1"
    local prefix="$2"

    for file in "$directory"/*; do

        [[ -f "$file" ]] || continue

        local filename
        filename="$(basename "$file")"

        local new_name
        new_name="${prefix}${filename}"

        mv -- "$file" "$directory/$new_name"

        log_info "Renamed: $filename -> $new_name"
    done
}

rename_with_suffix() {
    local directory="$1"
    local suffix="$2"

    for file in "$directory"/*; do

        [[ -f "$file" ]] || continue

        local filename
        filename="$(basename "$file")"

        local extension=""
        local name="$filename"

        if [[ "$filename" == *.* ]]; then
            extension=".${filename##*.}"
            name="${filename%.*}"
        fi

        local new_name
        new_name="${name}${suffix}${extension}"

        mv -- "$file" "$directory/$new_name"

        log_info "Renamed: $filename -> $new_name"
    done
}

rename_with_counter() {
    local directory="$1"
    local prefix="$2"

    local counter=1

    for file in "$directory"/*; do

        [[ -f "$file" ]] || continue

        local filename
        filename="$(basename "$file")"

        local extension=""
        if [[ "$filename" == *.* ]]; then
            extension=".${filename##*.}"
        fi

        local new_name
        new_name="${prefix}${counter}${extension}"

        mv -- "$file" "$directory/$new_name"

        log_info "Renamed: $filename -> $new_name"

        ((counter++))
    done
}

rename_with_date() {
    local directory="$1"
    local prefix="$2"

    local date
    date="$(date +"%Y-%m-%d")"

    for file in "$directory"/*; do

        [[ -f "$file" ]] || continue

        local filename
        filename="$(basename "$file")"

        local new_name
        new_name="${prefix}${date}_${filename}"

        mv -- "$file" "$directory/$new_name"

        log_info "Renamed: $filename -> $new_name"
    done
}

main() {

    if [[ $# -ne 3 ]]; then
        usage
        exit 1
    fi

    local directory="$1"
    local mode="$2"
    local value="$3"

    check_directory "$directory"

    case "$mode" in

        prefix)
            rename_with_prefix "$directory" "$value"
            ;;

        suffix)
            rename_with_suffix "$directory" "$value"
            ;;

        counter)
            rename_with_counter "$directory" "$value"
            ;;

        date)
            rename_with_date "$directory" "$value"
            ;;

        *)
            log_error "Unknown rename mode: $mode"
            usage
            exit 1
            ;;
    esac
}

main "$@"
```
