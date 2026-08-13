# 📂 Version Control Helper

## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/usr/bin/bash
#Version Control Helper

set -Eeuo pipefail

readonly SCRIPT_NAME="$(basename "$0")"

log_info() {
    printf "[INFO] %s\n" "$1"
}

log_error() {
    printf "[ERROR] %s\n" "$1" >&2
}

usage() {
    printf "Usage: %s <command> [arguments]\n" "$SCRIPT_NAME"
    printf "\n"
    printf "Commands:\n"
    printf "  init                 Initialize a Git repository\n"
    printf "  commit <message>     Commit staged changes\n"
    printf "  branch <name>        Create a new branch\n"
    printf "  switch <name>        Switch to a branch\n"
    printf "  status               Show repository status\n"
}

require_git_repo() {
    if ! git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
        log_error "Not inside a Git repository."
        exit 1
    fi
}

init_repo() {
    if git rev-parse --is-inside-work-tree >/dev/null 2>&1; then
        log_error "A Git repository already exists here."
        return 1
    fi

    git init
    log_info "Git repository initialized."
}

commit_changes() {
    require_git_repo

    local message="$1"

    if [[ -z "$message" ]]; then
        log_error "Commit message cannot be empty."
        return 1
    fi

    git add .
    git commit -m "$message"

    log_info "Changes committed successfully."
}

create_branch() {
    require_git_repo

    local branch_name="$1"

    if [[ -z "$branch_name" ]]; then
        log_error "Branch name cannot be empty."
        return 1
    fi

    git switch -c "$branch_name"

    log_info "Created and switched to branch: $branch_name"
}

switch_branch() {
    require_git_repo

    local branch_name="$1"

    if [[ -z "$branch_name" ]]; then
        log_error "Branch name cannot be empty."
        return 1
    fi

    git switch "$branch_name"

    log_info "Switched to branch: $branch_name"
}

show_status() {
    require_git_repo

    git status
}

main() {
    if [[ $# -eq 0 ]]; then
        usage
        exit 1
    fi

    local command="$1"
    shift

    case "$command" in
        init)
            init_repo
            ;;

        commit)
            if [[ $# -eq 0 ]]; then
                log_error "Commit message is required."
                usage
                exit 1
            fi

            commit_changes "$*"
            ;;

        branch)
            if [[ $# -ne 1 ]]; then
                log_error "Branch name is required."
                usage
                exit 1
            fi

            create_branch "$1"
            ;;

        switch)
            if [[ $# -ne 1 ]]; then
                log_error "Branch name is required."
                usage
                exit 1
            fi

            switch_branch "$1"
            ;;

        status)
            show_status
            ;;

        *)
            log_error "Unknown command: $command"
            usage
            exit 1
            ;;
    esac
}

main "$@"
```
