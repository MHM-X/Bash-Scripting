# 📂 File Encryption Tool

## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/usr/bin/env bash

# File Encryption Tool
#
# Description:
#   Encrypt and decrypt files using GnuPG.
#
# Requirements:
#   - gpg installed
#

set -Eeuo pipefail

readonly SCRIPT_NAME="$(basename "$0")"


# Logging

log_info() {
    printf "[INFO] %s\n" "$*"
}

log_success() {
    printf "[SUCCESS] %s\n" "$*"
}

log_error() {
    printf "[ERROR] %s\n" "$*" >&2
}


# Error handling

trap 'log_error "Command failed on line $LINENO"; exit 1' ERR


# Usage

usage() {

cat <<EOF

Usage:

Encrypt file:
    $SCRIPT_NAME encrypt <file>

Decrypt file:
    $SCRIPT_NAME decrypt <encrypted_file>

Examples:

    $SCRIPT_NAME encrypt document.pdf

    $SCRIPT_NAME decrypt document.pdf.gpg

EOF

}


# Dependency check

check_dependencies() {

    if ! command -v gpg >/dev/null 2>&1
    then
        log_error "gpg is not installed."
        exit 1
    fi

}


# Encrypt

encrypt_file() {

    local file="$1"

    if [[ ! -f "$file" ]]
    then
        log_error "File does not exist: $file"
        exit 1
    fi


    local output="${file}.gpg"


    if [[ -e "$output" ]]
    then
        log_error "Output file already exists: $output"
        exit 1
    fi


    log_info "Encrypting $file"


    gpg \
        --symmetric \
        --cipher-algo AES256 \
        --output "$output" \
        "$file"


    log_success "Encrypted successfully:"
    echo "$output"

}


# Decrypt

decrypt_file() {

    local file="$1"


    if [[ ! -f "$file" ]]
    then
        log_error "Encrypted file does not exist."
        exit 1
    fi


    local output="${file%.gpg}"


    if [[ -e "$output" ]]
    then
        log_error "Output file already exists: $output"
        exit 1
    fi


    log_info "Decrypting $file"


    gpg \
        --output "$output" \
        --decrypt "$file"


    log_success "Decrypted successfully:"
    echo "$output"

}


# Argument parsing


main() {


    check_dependencies


    if [[ $# -ne 2 ]]
    then
        usage
        exit 1
    fi


    local action="$1"
    local file="$2"


    case "$action" in

        encrypt)
            encrypt_file "$file"
            ;;


        decrypt)
            decrypt_file "$file"
            ;;


        *)
            usage
            exit 1
            ;;

    esac

}


main "$@"
```
