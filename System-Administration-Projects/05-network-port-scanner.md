# 📂 Network Port Scanner
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/bin/bash

# ==========================================
# Network Port Scanner
# ==========================================

# ---------- Root Check ----------

if [[ $EUID -ne 0 ]]; then
    echo "Warning: Running as root is recommended."
fi


# ---------- Default Configuration ----------

TIMEOUT=1

COMMON_PORTS=(
    21
    22
    23
    25
    53
    80
    110
    139
    143
    443
    445
    3306
    5432
    6379
    8080
    8443
)


# ---------- Service Identification ----------

get_service() {

    case "$1" in
        21)   echo "FTP" ;;
        22)   echo "SSH" ;;
        23)   echo "Telnet" ;;
        25)   echo "SMTP" ;;
        53)   echo "DNS" ;;
        80)   echo "HTTP" ;;
        110)  echo "POP3" ;;
        139)  echo "NetBIOS" ;;
        143)  echo "IMAP" ;;
        443)  echo "HTTPS" ;;
        445)  echo "SMB" ;;
        3306) echo "MySQL" ;;
        5432) echo "PostgreSQL" ;;
        6379) echo "Redis" ;;
        8080) echo "HTTP-Alt" ;;
        8443) echo "HTTPS-Alt" ;;
        *)    echo "Unknown" ;;
    esac
}


# ---------- Port Scanner ----------

scan_port() {

    local host="$1"
    local port="$2"

    if timeout "$TIMEOUT" bash -c \
        "echo >/dev/tcp/$host/$port" 2>/dev/null; then

        local service
        service=$(get_service "$port")

        printf "%-8s %-15s %s\n" \
            "$port" "OPEN" "$service"

        return 0
    fi

    return 1
}


# ---------- Scan Host ----------

scan_host() {

    local host="$1"

    echo
    echo "========================================"
    echo "Scanning host: $host"
    echo "========================================"

    printf "%-8s %-15s %s\n" \
        "PORT" "STATUS" "SERVICE"

    echo "----------------------------------------"

    local open_ports=0

    for port in "${COMMON_PORTS[@]}"; do

        if scan_port "$host" "$port"; then
            ((open_ports++))
        fi

    done

    echo "----------------------------------------"
    echo "Open ports found: $open_ports"
}


# ---------- Scan Port Range ----------

scan_range() {

    local host="$1"
    local start_port="$2"
    local end_port="$3"

    echo
    echo "========================================"
    echo "Scanning: $host"
    echo "Ports: $start_port-$end_port"
    echo "========================================"

    printf "%-8s %-15s %s\n" \
        "PORT" "STATUS" "SERVICE"

    echo "----------------------------------------"

    local open_ports=0

    for ((port=start_port; port<=end_port; port++)); do

        if scan_port "$host" "$port"; then
            ((open_ports++))
        fi

    done

    echo "----------------------------------------"
    echo "Open ports found: $open_ports"
}


# ---------- Validate IP/Hostname ----------

validate_host() {

    local host="$1"

    if [[ -z "$host" ]]; then
        echo "Error: Host cannot be empty."
        return 1
    fi

    return 0
}


# ---------- Validate Port ----------

validate_port() {

    local port="$1"

    if ! [[ "$port" =~ ^[0-9]+$ ]]; then
        echo "Error: Port must be a number."
        return 1
    fi

    if (( port < 1 || port > 65535 )); then
        echo "Error: Port must be between 1 and 65535."
        return 1
    fi

    return 0
}


# ---------- Scan Single Port ----------

single_port_scan() {

    read -rp "Enter host/IP: " host
    read -rp "Enter port: " port

    validate_host "$host" || return
    validate_port "$port" || return

    echo
    echo "Scanning $host:$port..."

    if scan_port "$host" "$port"; then
        echo
        echo "Result: Port is OPEN."
    else
        echo
        echo "Result: Port is CLOSED or FILTERED."
    fi
}


# ---------- Scan Common Ports ----------

common_port_scan() {

    read -rp "Enter host/IP: " host

    validate_host "$host" || return

    scan_host "$host"
}


# ---------- Scan Port Range ----------

port_range_scan() {

    read -rp "Enter host/IP: " host
    read -rp "Start port: " start_port
    read -rp "End port: " end_port

    validate_host "$host" || return
    validate_port "$start_port" || return
    validate_port "$end_port" || return

    if (( start_port > end_port )); then
        echo "Error: Start port must be smaller than end port."
        return
    fi

    scan_range "$host" "$start_port" "$end_port"
}


# ==========================================
# Main Menu
# ==========================================

while true; do

    echo
    echo "========================================"
    echo "          NETWORK PORT SCANNER"
    echo "========================================"
    echo
    echo "1) Scan common ports"
    echo "2) Scan a single port"
    echo "3) Scan a port range"
    echo "4) Exit"
    echo

    read -rp "Choose an option [1-4]: " choice

    case "$choice" in

        1)
            common_port_scan
            ;;

        2)
            single_port_scan
            ;;

        3)
            port_range_scan
            ;;

        4)
            echo "Goodbye."
            exit 0
            ;;

        *)
            echo "Error: Invalid option."
            ;;

    esac

done
```
