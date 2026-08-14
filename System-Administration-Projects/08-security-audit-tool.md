# 📂 Security Audit Tool
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/bin/bash

# Security Audit Tool

LOG_FILE="./security-audit.log"
REPORT_FILE="./security-report.txt"

# ---------- Colors ----------

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'


# ---------- Logging ----------

log_message() {

    local message="$1"

    echo "$(date '+%Y-%m-%d %H:%M:%S') - $message" \
        | tee -a "$LOG_FILE"
}


# ---------- Check Root ----------

check_root() {

    if [[ $EUID -ne 0 ]]; then

        echo "Error: Run this script as root."

        exit 1

    fi
}


# ---------- Check Password Policy ----------

check_password_policy() {

    log_message "Checking password policy..."

    local min_len

    min_len=$(awk '/^[[:space:]]*PASS_MIN_LEN/ {print $2}' /etc/login.defs)

    if [[ -z "$min_len" ]]; then

        echo -e "${YELLOW}[WARNING]${NC} PASS_MIN_LEN is not configured."

    elif (( min_len < 8 )); then

        echo -e "${RED}[HIGH]${NC} Minimum password length is only $min_len."

    else

        echo -e "${GREEN}[OK]${NC} Minimum password length is $min_len."

    fi
}


# ---------- Check Empty Passwords ----------

check_empty_passwords() {

    log_message "Checking for accounts without passwords..."

    local accounts

    accounts=$(awk -F: '$2 == "" {print $1}' /etc/shadow)

    if [[ -n "$accounts" ]]; then

        echo -e "${RED}[HIGH]${NC} Accounts with empty passwords:"
        echo "$accounts"

    else

        echo -e "${GREEN}[OK]${NC} No accounts with empty passwords found."

    fi
}


# ---------- Check UID 0 Accounts ----------

check_root_accounts() {

    log_message "Checking for additional UID 0 accounts..."

    local root_accounts

    root_accounts=$(awk -F: '$3 == 0 {print $1}' /etc/passwd)

    if [[ "$root_accounts" != "root" ]]; then

        echo -e "${RED}[HIGH]${NC} Additional UID 0 accounts found:"
        echo "$root_accounts"

    else

        echo -e "${GREEN}[OK]${NC} Only root has UID 0."

    fi
}


# ---------- Check SSH Configuration ----------

check_ssh_config() {

    log_message "Checking SSH configuration..."

    local ssh_config="/etc/ssh/sshd_config"

    if [[ ! -f "$ssh_config" ]]; then

        echo -e "${YELLOW}[WARNING]${NC} SSH configuration not found."

        return

    fi

    # or: if sshd -T 2>/dev/null | grep -q '^permitrootlogin yes$'; then
    if grep -Eq '^[[:space:]]*PermitRootLogin[[:space:]]+yes' "$ssh_config"; then

        echo -e "${RED}[HIGH]${NC} SSH root login is enabled."

    else

        echo -e "${GREEN}[OK]${NC} SSH root login is not explicitly enabled."

    fi

    # or: if sshd -T 2>/dev/null | grep -q '^passwordauthentication yes$'; then
    if grep -Eq '^[[:space:]]*PasswordAuthentication[[:space:]]+yes' "$ssh_config"; then

        echo -e "${YELLOW}[WARNING]${NC} SSH password authentication is enabled."

    else

        echo -e "${GREEN}[OK]${NC} SSH password authentication is not explicitly enabled."

    fi
}


# ---------- Check Open Ports ----------

check_open_ports() {

    log_message "Checking listening network ports..."

    echo

    echo "Listening TCP/UDP ports:"

    ss -tuln

    echo

}


# ---------- Check SUID Files ----------

check_suid_files() {

    log_message "Searching for SUID files..."

    echo "SUID files:"

    find / \
        -xdev \
        -type f \
        -perm -4000 \
        2>/dev/null

    echo

}


# ---------- Check Firewall ----------

check_firewall() {

    log_message "Checking firewall status..."
    # Uncomplicated Firewall
    if command -v ufw &>/dev/null; then

        if ufw status | grep -q "Status: active"; then

            echo -e "${GREEN}[OK]${NC} UFW firewall is active."

        else

            echo -e "${RED}[HIGH]${NC} UFW firewall is not active."

        fi

    elif command -v firewall-cmd &>/dev/null; then

        if firewall-cmd --state &>/dev/null; then

            echo -e "${GREEN}[OK]${NC} firewalld is active."

        else

            echo -e "${RED}[HIGH]${NC} firewalld is not active."

        fi

    else

        echo -e "${YELLOW}[WARNING]${NC} No supported firewall tool found."

    fi
}


# ---------- Check Updates ----------

check_updates() {

    log_message "Checking for available updates..."

    if command -v apt &>/dev/null; then
        # يعني quiet جدًا، فيقلل كمية الـ output الذي يطبعه apt.
        apt update -qq

        local updates

        updates=$(apt list --upgradable 2>/dev/null | tail -n +2)

        if [[ -n "$updates" ]]; then

            echo -e "${YELLOW}[WARNING]${NC} Available package updates:"
            echo "$updates"

        else

            echo -e "${GREEN}[OK]${NC} System packages are up to date."

        fi

    else

        echo -e "${YELLOW}[WARNING]${NC} apt package manager not found."

    fi
}


# ---------- Main ----------

main() {

    check_root

    log_message "=========================================="
    log_message "Security audit started."
    log_message "=========================================="

    echo
    echo "========== SECURITY AUDIT =========="
    echo

    check_password_policy

    echo

    check_empty_passwords

    echo

    check_root_accounts

    echo

    check_ssh_config

    echo

    check_open_ports

    check_suid_files

    check_firewall

    echo

    check_updates

    echo

    log_message "Security audit completed."

    echo "===================================="
    echo "Audit completed."
    echo "Log: $LOG_FILE"
    echo "===================================="
}


main
```
