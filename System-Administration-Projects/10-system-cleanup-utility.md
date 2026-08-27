# 📂 System Cleanup Utility
## Description
A script to remove unnecessary files and free up space. Target temporary files, old logs, and cache folders. Implement safe cleanup practices.

## Code
```bash
#!/usr/bin/env bash

set -euo pipefail

# ============================================================
# System Cleanup Utility
#
# Cleans:
#   - APT package cache
#   - Temporary files
#   - Old system logs
#   - Old files in /var/tmp
#
# Usage:
#   sudo ./cleanup.sh          # Dry run
#   sudo ./cleanup.sh --clean  # Perform cleanup
# ============================================================

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

DRY_RUN=true
LOG_AGE=30

# ------------------------------------------------------------
# Output functions
# ------------------------------------------------------------

info() {
    echo -e "${BLUE}[INFO]${NC} $1"
}

success() {
    echo -e "${GREEN}[OK]${NC} $1"
}

warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1"
}

error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# ------------------------------------------------------------
# Parse arguments
# ------------------------------------------------------------

if [[ "${1:-}" == "--clean" ]]; then
    DRY_RUN=false
elif [[ "${1:-}" == "--help" ]]; then
    echo "Usage:"
    echo "  $0          Show what can be cleaned"
    echo "  $0 --clean  Perform cleanup"
    exit 0
elif [[ -n "${1:-}" ]]; then
    error "Unknown option: $1"
    exit 1
fi

# ------------------------------------------------------------
# Root check
# ------------------------------------------------------------

if [[ $EUID -ne 0 ]]; then
    error "Please run this script as root."
    exit 1
fi

# ------------------------------------------------------------
# Check required commands
# ------------------------------------------------------------

for cmd in find du df; do
    if ! command -v "$cmd" &>/dev/null; then
        error "Required command not found: $cmd"
        exit 1
    fi
done

# ------------------------------------------------------------
# Show disk usage
# ------------------------------------------------------------

show_disk_usage() {

    echo
    info "Disk usage:"
    df -h /
    echo

}

# ------------------------------------------------------------
# Calculate directory size
# ------------------------------------------------------------

get_size() {

    local path="$1"

    if [[ -d "$path" ]]; then
        du -sh "$path" 2>/dev/null | awk '{print $1}'
    else
        echo "0"
    fi

}

# ------------------------------------------------------------
# APT cache
# ------------------------------------------------------------

clean_apt_cache() {

    if ! command -v apt-get &>/dev/null; then
        return
    fi

    info "APT cache: $(get_size /var/cache/apt/archives)"

    if [[ "$DRY_RUN" == true ]]; then
        echo "  Would run: apt-get clean"
    else
        apt-get clean
        success "APT cache cleaned."
    fi

}

# ------------------------------------------------------------
# Temporary files
# ------------------------------------------------------------

clean_tmp() {

    info "Checking /tmp..."

    local size
    size=$(get_size /tmp)

    echo "  Current size: $size"

    if [[ "$DRY_RUN" == true ]]; then

        echo "  Would remove files older than ${LOG_AGE} days."

    else

        find /tmp \
            -xdev \
            -type f \
            -mtime +"$LOG_AGE" \
            -delete

        success "Old temporary files removed."

    fi

}

# ------------------------------------------------------------
# /var/tmp cleanup
# ------------------------------------------------------------

clean_var_tmp() {

    info "Checking /var/tmp..."

    local size
    size=$(get_size /var/tmp)

    echo "  Current size: $size"

    if [[ "$DRY_RUN" == true ]]; then

        echo "  Would remove files older than ${LOG_AGE} days."

    else

        find /var/tmp \
            -xdev \
            -type f \
            -mtime +"$LOG_AGE" \
            -delete

        success "Old /var/tmp files removed."

    fi

}

# ------------------------------------------------------------
# Old system logs
# ------------------------------------------------------------

clean_logs() {

    if ! command -v journalctl &>/dev/null; then
        warning "journalctl not found. Skipping journal cleanup."
        return
    fi

    info "Checking system journal..."

    if [[ "$DRY_RUN" == true ]]; then

        journalctl --disk-usage

        echo
        echo "  Would remove journal logs older than ${LOG_AGE} days."

    else

        journalctl --vacuum-time="${LOG_AGE}d"

        success "Old journal logs removed."

    fi

}

# ------------------------------------------------------------
# Find large files
# ------------------------------------------------------------

show_large_files() {

    info "Largest files/directories under /var:"

    du -ah /var 2>/dev/null |
        sort -rh |
        head -n 10

}

# ------------------------------------------------------------
# Confirmation
# ------------------------------------------------------------

confirm_cleanup() {

    echo
    warning "Cleanup will permanently delete old files and cached data."
    read -rp "Continue? [y/N]: " answer

    case "$answer" in
        y|Y|yes|YES)
            return 0
            ;;
        *)
            echo "Cleanup cancelled."
            exit 0
            ;;
    esac

}

# ============================================================
# Main
# ============================================================

echo "========================================"
echo "        System Cleanup Utility"
echo "========================================"

echo
info "Cleanup age: ${LOG_AGE} days"

echo
info "Disk usage BEFORE cleanup:"
df -h /

echo
show_large_files

if [[ "$DRY_RUN" == true ]]; then

    echo
    info "Cleanup preview:"
    clean_apt_cache
    clean_tmp
    clean_var_tmp
    clean_logs

    echo
    warning "DRY RUN: Nothing has been deleted."
    echo "Run with --clean to perform cleanup."

    exit 0
fi

confirm_cleanup

echo
info "Starting cleanup..."

clean_apt_cache
clean_tmp
clean_var_tmp
clean_logs

echo
info "Disk usage AFTER cleanup:"
df -h /

success "Cleanup completed."
```
