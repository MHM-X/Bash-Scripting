# 📂 File Sync Utility

## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/usr/bin/bash
#File Sync Utility

set -Eeuo pipefail

readonly SCRIPT_NAME="$(basename "$0")"

log_info() {
    printf "[INFO] %s\n" "$1"
}

log_error() {
    printf "[ERROR] %s\n" "$1" >&2
}

usage() {
    printf "Usage: %s <folder1> <folder2>\n" "$SCRIPT_NAME"
}

validate_directory() {
    local directory="$1"

    if [[ ! -d "$directory" ]]; then
        log_error "Directory does not exist: $directory"
        exit 1
    fi
}

sync_directories() {
    local source="$1"
    local destination="$2"

    rsync -a --update "$source/" "$destination/"
}

sync_two_way() {
    local folder1="$1"
    local folder2="$2"

    log_info "Syncing $folder1 -> $folder2"
    sync_directories "$folder1" "$folder2"

    log_info "Syncing $folder2 -> $folder1"
    sync_directories "$folder2" "$folder1"

    log_info "Synchronization completed."
}

main() {
    if [[ $# -ne 2 ]]; then
        usage
        exit 1
    fi

    local folder1="$1"
    local folder2="$2"

    validate_directory "$folder1"
    validate_directory "$folder2"

    sync_two_way "$folder1" "$folder2"
}

main "$@"
```
