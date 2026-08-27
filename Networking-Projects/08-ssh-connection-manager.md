# 📂 SSH Connection Manager
## Description
A tool to manage and connect to many SSH servers easily. Store and organize connection details securely and support key-based authentication.

## Code
```bash
# SSH Connection Manager

#!/bin/bash

CONFIG="$HOME/.ssh_manager.conf"

# Create config file if it does not exist
touch "$CONFIG"

# Make config file accessible only by the user
chmod 600 "$CONFIG"


add_server() {

    read -p "Server name: " NAME
    read -p "IP address: " IP
    read -p "Username: " USER
    read -p "SSH key path (optional): " KEY

    if [[ -z "$KEY" ]]; then
        KEY="$HOME/.ssh/id_ed25519"
    fi

    echo "$NAME|$IP|$USER|$KEY" >> "$CONFIG"

    echo "Server added successfully."
}


list_servers() {

    echo
    echo "Available Servers"
    echo "-----------------"

    if [[ ! -s "$CONFIG" ]]; then
        echo "No servers found."
        return
    fi

    while IFS='|' read -r NAME IP USER KEY; do
        echo "$NAME -> $USER@$IP"
    done < "$CONFIG"
}


connect_server() {

    read -p "Enter server name: " CHOICE

    while IFS='|' read -r NAME IP USER KEY; do

        if [[ "$NAME" == "$CHOICE" ]]; then

            echo "Connecting to $USER@$IP..."

            ssh -i "$KEY" "$USER@$IP"

            return
        fi

    done < "$CONFIG"

    echo "Server not found."
}


delete_server() {

    read -p "Enter server name to delete: " CHOICE

    if grep -q "^$CHOICE|" "$CONFIG"; then

        grep -v "^$CHOICE|" "$CONFIG" > "${CONFIG}.tmp"
        # -v: اعكس البحث؛ أي اطبع الأسطر التي لا تطابق.
        mv "${CONFIG}.tmp" "$CONFIG"

        echo "Server deleted."

    else

        echo "Server not found."

    fi
}


while true; do

    echo
    echo "===== SSH Connection Manager ====="
    echo "1. Add server"
    echo "2. List servers"
    echo "3. Connect to server"
    echo "4. Delete server"
    echo "5. Exit"
    echo

    read -p "Choose an option: " OPTION

    case "$OPTION" in

        1)
            add_server
            ;;

        2)
            list_servers
            ;;

        3)
            connect_server
            ;;

        4)
            delete_server
            ;;

        5)
            echo "Goodbye!"
            exit 0
            ;;

        *)
            echo "Invalid option."
            ;;

    esac

done
```
