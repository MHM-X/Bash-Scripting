# 📂 File Metadata Editor

## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/usr/bin/env bash
# File Metadata Editor

set -Eeuo pipefail

readonly SCRIPT_NAME="$(basename "$0")"

log_info() {
    printf "[INFO] %s\n" "$1"
}

log_error() {
    printf "[ERROR] %s\n" "$1" >&2
}

check_exiftool() {
    if ! command -v exiftool >/dev/null 2>&1; then
        log_error "exiftool is not installed."
        exit 1
    fi
}

check_file() {
    local file="$1"

    if [[ ! -f "$file" ]]; then
        log_error "File does not exist: $file"
        return 1
    fi
}

view_metadata() {
    local file="$1"

    if ! check_file "$file"; then
        return 1
    fi

    printf "\n"
    log_info "Metadata for: $file"
    printf "\n"

    exiftool "$file"

    printf "\n"
}

edit_metadata() {
    local file="$1"
    local tag="$2"
    local value="$3"

    if ! check_file "$file"; then
        return 1
    fi

    exiftool "-${tag}=${value}" "$file"

    log_info "Metadata updated successfully."
}

show_menu() {
    printf "\n"
    printf "=============================\n"
    printf "     File Metadata Editor\n"
    printf "=============================\n"
    printf "\n"
    printf "1) View metadata\n"
    printf "2) Edit metadata\n"
    printf "3) Exit\n"
    printf "\n"
}

main() {
    check_exiftool

    while true; do
        show_menu

        read -rp "Choose an option: " choice

        case "$choice" in
            1)
                read -rp "Enter file path: " file

                if ! view_metadata "$file"; then
                    continue
                fi
                ;;

            2)
                read -rp "Enter file path: " file

                if ! check_file "$file"; then
                    continue
                fi

                printf "\n"
                printf "Common metadata tags:\n"
                printf "  Comment\n"
                printf "  Title\n"
                printf "  Artist\n"
                printf "  Description\n"
                printf "\n"

                read -rp "Enter metadata tag: " tag
                read -rp "Enter new value: " value

                edit_metadata "$file" "$tag" "$value"
                ;;

            3)
                log_info "Goodbye."
                exit 0
                ;;

            *)
                log_error "Invalid option."
                ;;
        esac
    done
}

main "$@"
```
