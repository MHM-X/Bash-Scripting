
# 📂 User Account Manager
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/bin/bash

# User Account Manager

# Root Check, Effective User ID
if [[ $EUID -ne 0 ]]; then
    echo "Error: This script must be run as root."
    exit 1
fi


# ---------- Helper Functions ----------

pause() {
    read -rp "Press Enter to continue..."
}


user_exists() {
    id "$1" &>/dev/null
}


group_exists() {
    getent group "$1" &>/dev/null
}


valid_username() {
    [[ "$1" =~ ^[a-z_][a-z0-9_-]*$ ]]
}


# Add User

add_user() {

    read -rp "Enter username: " username

    if [[ -z "$username" ]]; then
        echo "Error: Username cannot be empty."
        return
    fi

    if ! valid_username "$username"; then
        echo "Error: Invalid username."
        echo "Use lowercase letters, numbers, '_' or '-'."
        return
    fi

    if user_exists "$username"; then
        echo "Error: User '$username' already exists."
        return
    fi

    useradd -m -s /bin/bash "$username"

    if [[ $? -ne 0 ]]; then
        echo "Error: Failed to create user."
        return
    fi

    echo "User '$username' created successfully."

    echo
    echo "Set password for '$username':"
    passwd "$username"
}


# Delete User

delete_user() {

    read -rp "Enter username to delete: " username

    if ! user_exists "$username"; then
        echo "Error: User '$username' does not exist."
        return
    fi

    if [[ "$username" == "root" ]]; then
        echo "Error: You cannot delete root."
        return
    fi

    echo
    read -rp "Delete user '$username'? [y/N]: " confirm

    if [[ "$confirm" != "y" && "$confirm" != "Y" ]]; then
        echo "Operation cancelled."
        return
    fi

    read -rp "Also delete the home directory? [y/N]: " remove_home

    if [[ "$remove_home" == "y" || "$remove_home" == "Y" ]]; then
        userdel -r "$username"
    else
        userdel "$username"
    fi

    if [[ $? -eq 0 ]]; then
        echo "User '$username' deleted successfully."
    else
        echo "Error: Failed to delete user."
    fi
}


# Modify User

modify_user() {

    read -rp "Enter username: " username

    if ! user_exists "$username"; then
        echo "Error: User '$username' does not exist."
        return
    fi

    echo
    echo "1) Change username"
    echo "2) Change shell"
    echo "3) Change home directory"
    echo "4) Lock account"
    echo "5) Unlock account"
    echo "6) Change password"

    read -rp "Choose an option: " choice

    case "$choice" in

        1)
            read -rp "Enter new username: " new_username

            if ! valid_username "$new_username"; then
                echo "Error: Invalid username."
                return
            fi

            if user_exists "$new_username"; then
                echo "Error: Username already exists."
                return
            fi

            usermod -l "$new_username" "$username"

            if [[ $? -eq 0 ]]; then
                echo "Username changed successfully."
            else
                echo "Error: Failed to change username."
            fi
            ;;

        2)
            read -rp "Enter shell path: " shell

            if [[ ! -x "$shell" ]]; then
                echo "Error: Shell does not exist or is not executable."
                return
            fi

            usermod -s "$shell" "$username"

            if [[ $? -eq 0 ]]; then
                echo "Shell changed successfully."
            else
                echo "Error: Failed to change shell."
            fi
            ;;

        3)
            read -rp "Enter new home directory: " home

            usermod -d "$home" "$username"

            if [[ $? -eq 0 ]]; then
                echo "Home directory changed successfully."
            else
                echo "Error: Failed to change home directory."
            fi
            ;;

        4)
            passwd -l "$username"
            echo "User account locked."
            ;;

        5)
            passwd -u "$username"
            echo "User account unlocked."
            ;;

        6)
            passwd "$username"
            ;;

        *)
            echo "Invalid option."
            ;;
    esac
}


# Add User to Group

add_to_group() {

    read -rp "Enter username: " username

    if ! user_exists "$username"; then
        echo "Error: User does not exist."
        return
    fi

    read -rp "Enter group: " group

    if ! group_exists "$group"; then
        echo "Group '$group' does not exist."

        read -rp "Create group '$group'? [y/N]: " create

        if [[ "$create" == "y" || "$create" == "Y" ]]; then
            groupadd "$group"

            if [[ $? -ne 0 ]]; then
                echo "Error: Failed to create group."
                return
            fi

            echo "Group '$group' created."
        else
            return
        fi
    fi

    usermod -aG "$group" "$username"

    if [[ $? -eq 0 ]]; then
        echo "User '$username' added to group '$group'."
    else
        echo "Error: Failed to add user to group."
    fi
}


# Remove User from Group

remove_from_group() {

    read -rp "Enter username: " username

    if ! user_exists "$username"; then
        echo "Error: User does not exist."
        return
    fi

    read -rp "Enter group: " group

    if ! group_exists "$group"; then
        echo "Error: Group does not exist."
        return
    fi

    gpasswd -d "$username" "$group"

    if [[ $? -eq 0 ]]; then
        echo "User '$username' removed from group '$group'."
    else
        echo "Error: Failed to remove user from group."
    fi
}


# Show User Information

show_user_info() {

    read -rp "Enter username: " username

    if ! user_exists "$username"; then
        echo "Error: User does not exist."
        return
    fi

    echo
    echo "================================"
    echo "User Information"
    echo "================================"

    id "$username"

    echo
    echo "Groups:"
    groups "$username"

    echo
    echo "Home Directory:"
    getent passwd "$username" | cut -d: -f6

    echo
    echo "Login Shell:"
    getent passwd "$username" | cut -d: -f7

    echo
    echo "Password Information:"
    chage -l "$username"
}


# Password Policy

password_policy() {

    read -rp "Enter username: " username

    if ! user_exists "$username"; then
        echo "Error: User does not exist."
        return
    fi

    echo
    echo "Password Policy"
    echo "---------------"

    read -rp "Maximum password age (days): " max_days
    read -rp "Minimum password age (days): " min_days
    read -rp "Warning period (days): " warning_days

    if ! [[ "$max_days" =~ ^[0-9]+$ &&
            "$min_days" =~ ^[0-9]+$ &&
            "$warning_days" =~ ^[0-9]+$ ]]; then

        echo "Error: Values must be numbers."
        return
    fi

    chage \
        -M "$max_days" \
        -m "$min_days" \
        -W "$warning_days" \
        "$username"

    if [[ $? -eq 0 ]]; then
        echo
        echo "Password policy updated successfully."
        echo
        chage -l "$username"
    else
        echo "Error: Failed to update password policy."
    fi
}


# Main Menu

while true; do

    clear

    echo "========================================"
    echo "        USER ACCOUNT MANAGER"
    echo "========================================"
    echo
    echo "1) Add user"
    echo "2) Delete user"
    echo "3) Modify user"
    echo "4) Add user to group"
    echo "5) Remove user from group"
    echo "6) Show user information"
    echo "7) Set password policy"
    echo "8) Exit"
    echo

    read -rp "Choose an option [1-8]: " choice

    echo

    case "$choice" in

        1)
            add_user
            pause
            ;;

        2)
            delete_user
            pause
            ;;

        3)
            modify_user
            pause
            ;;

        4)
            add_to_group
            pause
            ;;

        5)
            remove_from_group
            pause
            ;;

        6)
            show_user_info
            pause
            ;;

        7)
            password_policy
            pause
            ;;

        8)
            echo "Goodbye."
            exit 0
            ;;

        *)
            echo "Error: Invalid option."
            pause
            ;;

    esac

done
```
