# 📂 Archive Manager

## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/usr/bin/bash
#Archive Manager

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
    printf "  %s create <format> <source> <output> [--password]\n" "$SCRIPT_NAME"
    printf "  %s extract <archive> <destination> [--password]\n" "$SCRIPT_NAME"
    printf "\n"
    printf "Formats:\n"
    printf "  tar\n"
    printf "  gz\n"
    printf "  bz2\n"
    printf "  xz\n"
}

check_source() {
    local source="$1"

    if [[ ! -e "$source" ]]; then
        log_error "Source does not exist: $source"
        exit 1
    fi
}

check_archive() {
    local archive="$1"

    if [[ ! -f "$archive" ]]; then
        log_error "Archive does not exist: $archive"
        exit 1
    fi
}

create_archive() {
    local format="$1"
    local source="$2"
    local output="$3"
    local password_protected="${4:-false}"

    check_source "$source"

    case "$format" in
        tar)
            tar -cf "$output" "$source"
            ;;

        gz)
            tar -czf "$output" "$source"
            ;;

        bz2)
            tar -cjf "$output" "$source"
            ;;

        xz)
            tar -cJf "$output" "$source"
            ;;

        *)
            log_error "Unsupported archive format: $format"
            exit 1
            ;;
    esac

    if [[ "$password_protected" == true ]]; then
        # GNU Privacy Guard.
        gpg --symetric --cipher-algo AES256 --output "${output}.gpg" "$output"
        rm "$output"

        log_info "Password-protected archive created: ${output}.gpg"

    else
        log_info "Archive created: $output"

    fi
}

extract_archive() {
    local archive="$1"
    local destination="$2"
    local password_protected="${3:-false}"

    check_archive "$archive"

    mkdir -p "$destination"

    if [[ "$password_protected" == true]]; then
        local decrypted_archive="${archive%.gpg}"

        gpg --output "$decrypted_archive" --decrypt "$archive"
        archive="$decrypted_archive"

        case "$archive" in
            *.tar)
                tar -xf "$archive" -C "$destination"
                ;;

            *.tar.gz|*.tgz)
                tar -xzf "$archive" -C "$destination"
                ;;

            *.tar.bz2|*.tbz2)
                tar -xjf "$archive" -C "$destination"
                ;;

            *.tar.xz|*.txz)
                tar -xJf "$archive" -C "$destination"
                ;;

            *)
                log_error "Unsupported archive format: $archive"
                rm -f "$decrypted_archive"
                exit 1
                ;;
        esac

        rm -f "$decrypted_archive"

    else
    case "$archive" in
        *.tar)
            tar -xf "$archive" -C "$destination"
            ;;

        *.tar.gz|*.tgz)
            tar -xzf "$archive" -C "$destination"
            ;;

        *.tar.bz2|*.tbz2)
            tar -xjf "$archive" -C "$destination"
            ;;

        *.tar.xz|*.txz)
            tar -xJf "$archive" -C "$destination"
            ;;

        *)
            log_error "Unsupported archive format: $archive"
            exit 1
            ;;
    esac

    log_info "Archive extracted to: $destination"
}

main() {
    if [[ $# -eq 0 ]]; then
        usage
        exit 1
    fi

    local command="$1"
    shift

    case "$command" in
        create)
            if [[ $# -lt 3 || $# -gt 4]]; then
                usage
                exit 1
            fi

            local password_protected=false
            if [[ $# -eq 4 ]]; then
                if [[ "$4" != "--password" ]]; then
                    log_error "Unknown option: $4"
                    exit 1
                fi

                password_protected=true
            fi


            create_archive "$1" "$2" "$3" "$password_protected"
            ;;

        extract)
            if [[ $# -lt 2 || $# -gt 3 ]]; then
                usage
                exit 1
            fi

            local password_protected=false

            if [[ $# -eq 3 ]]; then
                if [[ "$3" != "--password" ]]; then
                    log_error "Unknown option: $3"
                    exit 1
                fi

                password_protected=true
            fi

            extract_archive "$1" "$2" "$password_protected"
            ;;

        *)
            log_error "Unknown command: $command"
            usage
            exit 1
            ;;
    esac
}

main "$@"
```
