# 📂 Automated Backup Solution

## Description
A tool for scheduled backups of system and user data. Include options for full, partial, and different backups. Implement retention policies and backup verification.

## Code
```bash
#!/usr/bin/env bash

# ============================================================
# Automated Backup Solution
#
# Usage:
#   ./backup.sh full SOURCE BACKUP_DIR
#   ./backup.sh partial SOURCE BACKUP_DIR PATH...
#   ./backup.sh differential SOURCE BACKUP_DIR
#
# Examples:
#   ./backup.sh full /home /backup
#
#   ./backup.sh partial /home/mahmoud /backup \
#       Documents Projects .bashrc
#
#   ./backup.sh differential /home/mahmoud /backup
#
# ============================================================

set -Eeuo pipefail

# ------------------------------------------------------------
# Configuration
# ------------------------------------------------------------

readonly SCRIPT_NAME="$(basename "$0")"
readonly HOSTNAME="$(hostname -s)"
readonly RETENTION_COUNT=7

readonly LOCK_FILE="/var/run/${SCRIPT_NAME}.lock"
readonly LOG_FILE="/var/log/${SCRIPT_NAME}.log"

DATE="$(date '+%Y-%m-%d_%H-%M-%S')"

# ------------------------------------------------------------
# Logging
# ------------------------------------------------------------

log_info() {
    local message="$1"

    printf '[INFO] %s - %s\n' \
        "$(date '+%Y-%m-%d %H:%M:%S')" \
        "$message" | tee -a "$LOG_FILE"
}

log_error() {
    local message="$1"

    printf '[ERROR] %s - %s\n' \
        "$(date '+%Y-%m-%d %H:%M:%S')" \
        "$message" | tee -a "$LOG_FILE" >&2
}

# ------------------------------------------------------------
# Error Handler
# ------------------------------------------------------------

on_error() {
    local exit_code=$?
    local line_number="$1"

    log_error "Command failed on line $line_number with exit code $exit_code"

    exit "$exit_code"
}

trap 'on_error "$LINENO"' ERR

# ------------------------------------------------------------
# Cleanup
# ------------------------------------------------------------

TEMP_FILE=""

cleanup() {
    if [[ -n "$TEMP_FILE" && -f "$TEMP_FILE" ]]; then
        rm -f -- "$TEMP_FILE"
    fi
}

trap cleanup EXIT

# ------------------------------------------------------------
# Usage
# ------------------------------------------------------------

usage() {
    cat <<EOF
Usage:

  $SCRIPT_NAME full SOURCE_DIR BACKUP_DIR

  $SCRIPT_NAME partial SOURCE_DIR BACKUP_DIR PATH...

  $SCRIPT_NAME differential SOURCE_DIR BACKUP_DIR

Examples:

  $SCRIPT_NAME full /home /backup

  $SCRIPT_NAME partial /home/mahmoud /backup Documents Projects

  $SCRIPT_NAME differential /home/mahmoud /backup
EOF
}

# ------------------------------------------------------------
# Validate Arguments
# ------------------------------------------------------------

validate_arguments() {

    if (( $# < 3 )); then
        usage
        exit 2
    fi

    local backup_type="$1"
    local source_dir="$2"
    local backup_dir="$3"

    case "$backup_type" in

        full|differential)
            ;;

        partial)
            if (( $# < 4 )); then
                log_error "Partial backup requires at least one path."
                usage
                exit 2
            fi
            ;;

        *)
            log_error "Unknown backup type: $backup_type"
            usage
            exit 2
            ;;
    esac

    if [[ ! -d "$source_dir" ]]; then
        log_error "Source directory does not exist: $source_dir"
        exit 1
    fi

    mkdir -p -- "$backup_dir"

    if [[ ! -w "$backup_dir" ]]; then
        log_error "Backup directory is not writable: $backup_dir"
        exit 1
    fi

    # w: هل لدي صلاحية الكتابة على هذا المسار؟
}

# ------------------------------------------------------------
# Lock
# ------------------------------------------------------------

acquire_lock() {

    exec 200>"$LOCK_FILE"

    if ! flock -n 200; then
        log_error "Another backup process is already running."
        exit 1
    fi
}

# ------------------------------------------------------------
# Check Required Commands
# ------------------------------------------------------------

check_dependencies() {

    local commands=(
        tar
        gzip
        flock
        find
        date
        hostname
        tee
    )

    for command in "${commands[@]}"; do

        if ! command -v "$command" >/dev/null 2>&1; then
            log_error "Required command not found: $command"
            exit 1
        fi

    done
}

# ------------------------------------------------------------
# Create Backup
# ------------------------------------------------------------

create_full_backup() {

    local source_dir="$1"
    local backup_dir="$2"

    local backup_file
    backup_file="$backup_dir/${HOSTNAME}_full_${DATE}.tar.gz"

    TEMP_FILE="${backup_file}.tmp"

    log_info "Starting full backup."
    log_info "Source: $source_dir"
    log_info "Destination: $backup_file"

    tar \
        --create \
        --gzip \
        --file "$TEMP_FILE" \
        --directory "$source_dir" \
        .

    # or:     tar -czf "$TEMP_FILE" -C "$source_dir" .

    mv -- "$TEMP_FILE" "$backup_file"

    touch "$backup_dir/.full_backup_baseline"

    TEMP_FILE=""

    printf '%s\n' "$backup_file"
}

# ------------------------------------------------------------
# Partial Backup
# ------------------------------------------------------------

create_partial_backup() {

    local source_dir="$1"
    local backup_dir="$2"

    shift 2

    local backup_file
    backup_file="$backup_dir/${HOSTNAME}_partial_${DATE}.tar.gz"

    TEMP_FILE="${backup_file}.tmp"

    log_info "Starting partial backup."

    log_info "Selected paths:"
    printf '  - %s\n' "$@" | tee -a "$LOG_FILE"

    local path

    for path in "$@"; do

        if [[ ! -e "$source_dir/$path" ]]; then
            log_error "Path does not exist: $source_dir/$path"
            exit 1
        fi

    done

    tar \
        --create \
        --gzip \
        --file "$TEMP_FILE" \
        --directory "$source_dir" \
        -- "$@"

    # or: tar -czf "$TEMP_FILE" -C "$source_dir" -- "$@"

    mv -- "$TEMP_FILE" "$backup_file"

    TEMP_FILE=""

    printf '%s\n' "$backup_file"
}

# ------------------------------------------------------------
# Incremental Backup
# ------------------------------------------------------------

create_Incremental_backup() {

    local source_dir="$1"
    local backup_dir="$2"

    local backup_file
    backup_file="$backup_dir/${HOSTNAME}_incremental_${DATE}.tar.gz"

    TEMP_FILE="${backup_file}.tmp"

    local snapshot_file
    snapshot_file="$backup_dir/.incremental.snapshot"
    log_info "Starting incremental backup."

    tar \
        --create \
        --gzip \
        --listed-incremental="$snapshot_file" \
        --file "$TEMP_FILE" \
        --directory "$source_dir" \
        .

    # or: tar -czf "$TEMP_FILE" -g "$snapshot_file" -C "$source_dir" .

    mv -- "$TEMP_FILE" "$backup_file"

    TEMP_FILE=""

    printf '%s\n' "$backup_file"
}

# ------------------------------------------------------------
# Differential Backup
# ------------------------------------------------------------

create_differential_backup() {
    local source_dir="$1"
    local backup_dir="$2"

    local backup_file
    backup_file="$backup_dir/${HOSTNAME}_differential_${DATE}.tar.gz"

    TEMP_FILE="${backup_file}.tmp"

    local baseline_file
    baseline_file="$backup_dir/.full_backup_baseline"

    log_info "Starting differential backup."

    if [[ ! -f "$baseline_file" ]]; then
        log_error "No full backup baseline found."
        exit 1
    fi

    while IFS= read -r -d '' file; do
        printf '%s\0' "$file" >> "$changed_list"
    done < <(
        find "$source_dir" -type f -newer "$baseline_file" -print0
    )

    if [[ ! -s "$changed_list" ]]; then
        log_info "No files have changed since the last full backup."
        rm -f -- "$changed_list"
        TEMP_FILE=""
        return 0
    fi

    tar \
        --create \
        --gzip \
        --null \
        --files-from="$changed_list" \
        --file "$TEMP_FILE" \
        --directory "$source_dir"

    # or:     tar -czf "$TEMP_FILE"  -C "$source_dir" --null -T "$changed_list"

    rm -f -- "$changed_list"

    mv -- "$TEMP_FILE" "$backup_file"

    TEMP_FILE=""

    printf '%s\n' "$backup_file"
}

# ------------------------------------------------------------
# Verify Backup
# ------------------------------------------------------------

verify_backup() {

    local backup_file="$1"

    log_info "Verifying backup: $backup_file"

    if ! tar --list --gzip --file "$backup_file" >/dev/null; then
        log_error "Backup verification failed: $backup_file"
        exit 1
    fi

    log_info "Backup verification successful."
}

# ------------------------------------------------------------
# Calculate Backup Size
# ------------------------------------------------------------

report_backup_size() {

    local backup_file="$1"

    local size
    size="$(du -h "$backup_file" | awk '{print $1}')"

    log_info "Backup size: $size"
}

# ------------------------------------------------------------
# Retention Policy
# ------------------------------------------------------------

apply_retention_policy() {

    local backup_dir="$1"

    log_info "Applying retention policy."
    log_info "Keeping latest $RETENTION_COUNT backups."

    mapfile -t backups < <(
        find "$backup_dir" \
            -maxdepth 1 \
            -type f \
            -name '*.tar.gz' \
            -printf '%T@ %p\n' |
        sort -nr |
        cut -d' ' -f2-
    )

    if (( ${#backups[@]} <= RETENTION_COUNT )); then
        log_info "No old backups need to be removed."
        return
    fi

    local old_backup

    for old_backup in "${backups[@]:RETENTION_COUNT}"; do

        log_info "Removing old backup: $old_backup"

        rm -f -- "$old_backup"

    done
}

# ------------------------------------------------------------
# Main
# ------------------------------------------------------------

main() {

    local backup_type="$1"
    local source_dir="$2"
    local backup_dir="$3"

    validate_arguments "$@"

    check_dependencies

    acquire_lock

    log_info "=========================================="
    log_info "Backup job started."
    log_info "Backup type: $backup_type"
    log_info "Host: $HOSTNAME"

    local backup_file

    case "$backup_type" in

        full)
            backup_file="$(
                create_full_backup \
                    "$source_dir" \
                    "$backup_dir"
            )
            ;;

        partial)
            shift 3

            backup_file="$(
                create_partial_backup \
                    "$source_dir" \
                    "$backup_dir" \
                    "$@"
            )
            ;;

        incremental)
            backup_file="$(
                create_Incremental_backup \
                    "$source_dir" \
                    "$backup_dir"
            )
            ;;

    esac

    verify_backup "$backup_file"

    report_backup_size "$backup_file"

    apply_retention_policy "$backup_dir"

    log_info "Backup completed successfully."
    log_info "=========================================="
}

main "$@"
```
