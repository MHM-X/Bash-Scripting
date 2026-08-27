# 📂 System Health Monitor
## Description
A script to check and report on system resources and health. Monitor CPU usage, memory use, disk space, and running processes. Generate alerts for abnormal conditions.

## Code
```bash
#!/bin/bash
#System Health Monitor

set -u

# Thresholds
CPU_THRESHOLD=80
MEMORY_THRESHOLD=80
DISK_THRESHOLD=80
PROCESS_THRESHOLD=200

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'


# Get CPU usage
get_cpu_usage() {
    top -bn1 | awk '/Cpu\(s\)/ {
        print 100 - $8
    }'
}
# -b: Batch mode. لا تعرض واجهة top التفاعلية، بل اطبع النتائج كنص.
# -n1:نفذ top مرة واحدة فقط 
#    بدونها توب يستمر في تحديث القيم

# Get memory usage
get_memory_usage() {
    free | awk '/Mem:/ {
        printf "%.0f", ($3 / $2) * 100
    }'
}


# Get disk usage
get_disk_usage() {
    df / | awk 'NR==2 {
        gsub("%", "", $5)
        print $5
    }'
}

# df: disk filesystem
# NR: Number of Record رقم ال ريكورد الحالي الذي يعالجه awk
#    نفّذ الكود فقط عندما نصل إلى السطر الثاني
# gsub: Global Substitution تعني استبدال شيئ بشيئ اخر


# Get number of running processes
get_process_count() {
    ps -e --no-headers | wc -l
}

# ps: Process Status
# -e: اعرض كل البروسيسيس الموجودة على النظام
# --no-headers: حذف سطر عناوين الهيدرز فيبقى عدد البروسيسيس فقط



# Check CPU
check_cpu() {
    local cpu="$1"

    if (( ${cpu%.*} >= CPU_THRESHOLD )); then
        echo -e "${RED}[ALERT]${NC} CPU usage: ${cpu}%"
    else
        echo -e "${GREEN}[OK]${NC} CPU usage: ${cpu}%"
    fi
}


# Check memory
check_memory() {
    local memory="$1"

    if (( memory >= MEMORY_THRESHOLD )); then
        echo -e "${RED}[ALERT]${NC} Memory usage: ${memory}%"
    else
        echo -e "${GREEN}[OK]${NC} Memory usage: ${memory}%"
    fi
}


# Check disk
check_disk() {
    local disk="$1"

    if (( disk >= DISK_THRESHOLD )); then
        echo -e "${RED}[ALERT]${NC} Disk usage: ${disk}%"
    else
        echo -e "${GREEN}[OK]${NC} Disk usage: ${disk}%"
    fi
}


# Check processes
check_processes() {
    local processes="$1"

    if (( processes >= PROCESS_THRESHOLD )); then
        echo -e "${RED}[ALERT]${NC} Running processes: ${processes}"
    else
        echo -e "${GREEN}[OK]${NC} Running processes: ${processes}"
    fi
}


# Collect system information
CPU=$(get_cpu_usage)
MEMORY=$(get_memory_usage)
DISK=$(get_disk_usage)
PROCESSES=$(get_process_count)


# Report
echo "======================================"
echo "       SYSTEM HEALTH MONITOR"
echo "======================================"

echo "Hostname: $(hostname)"
echo "Date:     $(date)"
echo

check_cpu "$CPU"
check_memory "$MEMORY"
check_disk "$DISK"
check_processes "$PROCESSES"

echo "======================================"
```
