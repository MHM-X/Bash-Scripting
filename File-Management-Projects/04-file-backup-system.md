# 📂 File Backup System

## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/bin/bash
# File Backup System

# backup-system/
# │
# ├── backup.sh
# ├── config.conf
# ├── logs/
# │     backup.log
# │
# ├── backups/
# │
# └── README.md

source ./config.conf
TIMESTAMP=$(date+"%Y-%-%d_%H-%M-%S")
LOG_FILE="$LOG_DIR/backup.log"

mkdir -p "$DESTINATION"
mkdir -p "$LOG_DIR"

log(){
    printf "[%s] %s" "$TIMESTAMP" "$1" | tee -a "$LOG_FILE"
}

die(){
    log "ERROR: $1"
    exit 1
}

# Check dependencies
for cmd in rsync tar find
do
    "$cmd" -v >/dev/null || die "$cmd not found."
done

# Menu
echo
echo "1) Full Backup"
echo "2) Incremental Backup"
echo "3) Exit"
echo

read -rp "Choice: " choice

case "$choice" in

1)

    TYPE="full"

;;

2)

    TYPE="incremental"

;;

*)

    exit 0

;;

esac


# Backup filename
ARCHIVE="$DESTINATION/${TYPE}_backup_${TIMESTAMP}.tar.gz"
TMP_DIR=$(mktemp -d)
trap 'rm -rf "$TMP_DIR"' EXIT

# Full Backup
if [["$TYPE" == "full"]]; then
    log "Starting full backup..."
    rsync -a --delete "$SOURCE/" "$TMP_DIR"



# Incremental
else

    log "Starting incremental backup..."
    LAST_BACKUP=$(find "$DESTINATION/" -name "incremental_backup_*.tar.gz" | sort | tail -1)
    if [[ -z "$LAST_BACKUP" ]]; then
        log "No previous incremental backup found."

        rsync -a "$SOURCE/" "$TMP_DIR/"
        else
# Updates the temporary directory with new or modified files while keeping existing ones
        rsync -au "$SOURCE/" "$TMP_DIR/"
# or we can use:
# rsync -a --link-dest="$LAST_BACKUP" "$SOURCE/" "$TMP_DIR/"
# Creates a true incremental backup by linking unchanged files to the last backup (Hard link)
    fi
fi


# Compression
if [[ "$COMPRESSION" == true ]]; then
    tar -czf "$ARCHIVE" -C "$TMP_DIR" .

    else

    tar -cf "${ARCHIVE%.gz}" -C "$TMP_DIR" .

fi

log "Backup created: $ARCHIVE"


# Cleanup old backups
find "$DESTINATION" -type f -mtime +"$RETENTION_DAYS" -delete

log "Old backups removed."

log "Backup completed successfully."
```
