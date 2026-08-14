# 📂 Software Update Manager
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/usr/bin/bash

set -euo pipefail

# ============================================================
# Software Update Manager
# Supports:
#   - APT       (Debian / Ubuntu)
#   - DNF       (Fedora / RHEL)
#   - YUM       (older RHEL / CentOS)
#   - Pacman    (Arch Linux)
# ============================================================

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

print_info() {
    echo -e "${BLUE}[INFO]${NC} $1"
}

print_success() {
    echo -e "${GREEN}[OK]${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}[WARNING]${NC} $1"
}

print_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# ------------------------------------------------------------
# Check root privileges
# ------------------------------------------------------------

if [[ $EUID -ne 0 ]]; then
    print_error "Please run this script as root."
    exit 1
fi

# ------------------------------------------------------------
# Detect package manager
# ------------------------------------------------------------

if command -v apt-get &>/dev/null; then
    PACKAGE_MANAGER="apt"

elif command -v dnf &>/dev/null; then
    PACKAGE_MANAGER="dnf"

elif command -v yum &>/dev/null; then
    PACKAGE_MANAGER="yum"

elif command -v pacman &>/dev/null; then
    PACKAGE_MANAGER="pacman"

else
    print_error "No supported package manager found."
    exit 1
fi

print_info "Detected package manager: $PACKAGE_MANAGER"

# ------------------------------------------------------------
# Update package indexes
# ------------------------------------------------------------

update_package_index() {

    print_info "Updating package indexes..."

    case "$PACKAGE_MANAGER" in

        apt)
            apt-get update -qq
            ;;

        dnf)
            dnf makecache -q
            ;;

        yum)
            yum makecache -q
            ;;

        pacman)
            pacman -Sy --noconfirm
            ;;

    esac

    print_success "Package indexes updated."
}

# ------------------------------------------------------------
# Find available updates
# ------------------------------------------------------------

get_updates() {

    case "$PACKAGE_MANAGER" in

        apt)
            apt list --upgradable 2>/dev/null | tail -n +2
            ;;

        dnf)
            dnf check-update 2>/dev/null || true
            ;;

        yum)
            yum check-update 2>/dev/null || true
            ;;

        pacman)
            pacman -Qu 2>/dev/null || true
            ;;

    esac
}

# ------------------------------------------------------------
# Install updates
# ------------------------------------------------------------

install_updates() {

    print_info "Installing available updates..."

    case "$PACKAGE_MANAGER" in

        apt)
            apt-get upgrade -y
            ;;

        dnf)
            dnf upgrade -y
            ;;

        yum)
            yum update -y
            ;;

        pacman)
            pacman -Su --noconfirm
            ;;

    esac

    print_success "Software updates installed successfully."
}

# ------------------------------------------------------------
# Main
# ------------------------------------------------------------

update_package_index

print_info "Checking for available updates..."

updates=$(get_updates)

if [[ -z "$updates" ]]; then

    print_success "System is already up to date."
    exit 0

fi

echo
echo "========================================"
echo "        Available Software Updates"
echo "========================================"
echo

echo "$updates"

echo
echo "========================================"

read -rp "Do you want to install these updates? [y/N]: " answer

case "$answer" in
    y|Y|yes|YES)
        install_updates
        ;;

    *)
        print_warning "Update installation cancelled."
        exit 0
        ;;
esac
```
