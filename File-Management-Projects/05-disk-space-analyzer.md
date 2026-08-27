# 📂 Disk Space Analyzer

## Description
A tool to show which folders and files use the most space. Create a tree-like structure to display disk usage and offer options to sort and filter results.
## Code
```bash
#!/usr/bin/env bash
#
# disk-usage-analyzer.sh
#
# A production-style disk usage analyzer.
# Shows a tree of the biggest space consumers under a given path, with
# sorting and filtering options.
#
# Usage:
#   ./disk-usage-analyzer.sh [OPTIONS] [PATH]
#
# Run with -h/--help for full usage.
#

set -euo pipefail
IFS=$'\n\t'

# --- Constants --------------------------------------------------------------

readonly SCRIPT_NAME="${0##*/}"     # basename of this script, no external `basename` call needed
readonly VERSION="1.0.0"

# Exit codes (documented so callers / other scripts can rely on them)
readonly EXIT_OK=0
readonly EXIT_ERROR=1
readonly EXIT_USAGE=2

# --- Default configuration (overridable via CLI flags) -----------------------

TARGET_PATH="."
MAX_DEPTH=2
TOP_N=15
SORT_KEY="size"     # size | name
REVERSE=false
FILTER=""           # extended regex (awk ERE), empty = no filtering
DIRS_ONLY=false
SHOW_BYTES=false    # false = human-readable (via numfmt), true = raw bytes
USE_COLOR=true
DEBUG=false

# --- Helpers ------------------------------------------------------------------

# die MESSAGE [EXIT_CODE]
# Print an error to stderr and exit. Centralizing this keeps error output
# consistent and makes exit codes easy to audit.
die() {
    local message="$1"
    local code="${2:-$EXIT_ERROR}"
    printf '%s: error: %s\n' "$SCRIPT_NAME" "$message" >&2
    exit "$code"
}

usage() {
    cat <<EOF
$SCRIPT_NAME v$VERSION
Show which folders and files use the most disk space, as a tree.

USAGE:
    $SCRIPT_NAME [OPTIONS] [PATH]

    PATH defaults to the current directory.

OPTIONS:
    -d, --depth N        Max depth to descend into the tree (default: $MAX_DEPTH)
    -n, --top N          Show only the top N entries per directory level
                          (default: $TOP_N). Use 0 for unlimited.
    -s, --sort KEY        Sort by "size" or "name" (default: $SORT_KEY)
    -r, --reverse         Reverse the sort order
    -f, --filter PATTERN  Only include entries whose name matches this
                          extended regular expression (e.g. '\.log\$')
        --dirs-only       Only show directories, skip individual files
        --bytes           Show raw byte counts instead of human-readable sizes
        --no-color        Disable colored output
        --debug           Print each command as it runs (set -x)
    -h, --help            Show this help and exit
    -v, --version         Show version and exit

EXAMPLES:
    $SCRIPT_NAME                          # analyze current directory, depth 2
    $SCRIPT_NAME -d 3 -n 10 /var/log       # top 10 per level, 3 levels deep
    $SCRIPT_NAME -s name /home/user        # sort alphabetically instead of by size
    $SCRIPT_NAME -f '\.mp4\$' ~/Videos      # only show .mp4 files/dirs matching pattern
    $SCRIPT_NAME --dirs-only -d 1 /          # quick top-level breakdown of /
EOF
}

# check_dependencies
# Fail early with one clear message instead of dying midway through the
# tree walk with a confusing "command not found".
check_dependencies() {
    local cmd
    for cmd in du awk sort grep; do
        command -v "$cmd" >/dev/null 2>&1 || die "required command '$cmd' not found in PATH" "$EXIT_ERROR"
    done
    # numfmt is used for human-readable sizes but is not strictly required;
    # we fall back to raw bytes if it's missing (see format_size()).
}

# format_size BYTES
# Convert a byte count to a human-readable string (e.g. 1.2G), unless
# --bytes was requested or numfmt isn't available.
format_size() {
    local bytes="$1"
    if [[ "$SHOW_BYTES" == true ]] || ! command -v numfmt >/dev/null 2>&1; then
        printf '%s' "$bytes"
        return
    fi
    numfmt --to=iec-i --suffix=B --format='%.1f' -- "$bytes"
}

# is_valid_depth / is_valid_number
# Basic input validation helpers, used while parsing arguments.
is_positive_integer() {
    [[ "$1" =~ ^[0-9]+$ ]]
}

# --- Argument parsing -----------------------------------------------------
#
# getopts alone can't parse long options like --depth, so we roll a manual
# parser. Each iteration consumes one flag (and its value, if any), then
# shifts past it. `--` ends option parsing so a path like "-weird-dir"
# can still be passed safely as the final positional argument.

parse_args() {
    while [[ $# -gt 0 ]]; do
        case "$1" in
            -d|--depth)
                [[ $# -ge 2 ]] || die "'$1' requires a value" "$EXIT_USAGE"
                is_positive_integer "$2" || die "--depth expects a non-negative integer, got '$2'" "$EXIT_USAGE"
                MAX_DEPTH="$2"
                shift 2
                ;;
            --depth=*)
                MAX_DEPTH="${1#*=}"
                is_positive_integer "$MAX_DEPTH" || die "--depth expects a non-negative integer" "$EXIT_USAGE"
                shift
                ;;
            -n|--top)
                [[ $# -ge 2 ]] || die "'$1' requires a value" "$EXIT_USAGE"
                is_positive_integer "$2" || die "--top expects a non-negative integer, got '$2'" "$EXIT_USAGE"
                TOP_N="$2"
                shift 2
                ;;
            --top=*)
                TOP_N="${1#*=}"
                is_positive_integer "$TOP_N" || die "--top expects a non-negative integer" "$EXIT_USAGE"
                shift
                ;;
            -s|--sort)
                [[ $# -ge 2 ]] || die "'$1' requires a value" "$EXIT_USAGE"
                [[ "$2" == "size" || "$2" == "name" ]] || die "--sort must be 'size' or 'name', got '$2'" "$EXIT_USAGE"
                SORT_KEY="$2"
                shift 2
                ;;
            --sort=*)
                SORT_KEY="${1#*=}"
                [[ "$SORT_KEY" == "size" || "$SORT_KEY" == "name" ]] || die "--sort must be 'size' or 'name'" "$EXIT_USAGE"
                shift
                ;;
            -r|--reverse)
                REVERSE=true
                shift
                ;;
            -f|--filter)
                [[ $# -ge 2 ]] || die "'$1' requires a value" "$EXIT_USAGE"
                FILTER="$2"
                shift 2
                ;;
            --filter=*)
                FILTER="${1#*=}"
                shift
                ;;
            --dirs-only)
                DIRS_ONLY=true
                shift
                ;;
            --bytes)
                SHOW_BYTES=true
                shift
                ;;
            --no-color)
                USE_COLOR=false
                shift
                ;;
            --debug)
                DEBUG=true
                shift
                ;;
            -h|--help)
                usage
                exit "$EXIT_OK"
                ;;
            -v|--version)
                printf '%s\n' "$VERSION"
                exit "$EXIT_OK"
                ;;
            --)
                shift
                # Everything after -- is positional (the path)
                if [[ $# -gt 0 ]]; then
                    TARGET_PATH="$1"
                    shift
                fi
                break
                ;;
            -*)
                die "unknown option '$1' (see --help)" "$EXIT_USAGE"
                ;;
            *)
                TARGET_PATH="$1"
                shift
                ;;
        esac
    done

    [[ $# -eq 0 ]] || die "unexpected extra arguments: $*" "$EXIT_USAGE"
}

# --- Core logic -------------------------------------------------------------

# strip_trailing_slash PATH
# Normalizes a path so string comparisons against du's output are reliable.
# ("/var/log/" and "/var/log" must be treated as the same path.)
strip_trailing_slash() {
    local p="$1"
    if [[ "$p" != "/" ]]; then
        p="${p%/}"
    fi
    printf '%s' "$p"
}

# get_children DIR
# Prints "bytes<TAB>path" for each immediate child of DIR, already:
#   - stripped of the DIR-itself total line
#   - filtered by DIRS_ONLY / FILTER
#   - sorted per SORT_KEY / REVERSE
#   - limited to TOP_N lines
#
# Filtering note: --filter only hides *files* that don't match. Directories
# are always kept (even when non-matching) so that a matching file several
# levels deep stays reachable -- otherwise a non-matching parent directory
# would get filtered out before we ever had a chance to recurse into it.
#
# `du`'s exit status is deliberately ignored (`|| true`): scanning a large
# tree almost always hits at least one permission-denied subdirectory, and
# we don't want that to abort the whole run under `set -e`/`pipefail`.
get_children() {
    local dir="$1"
    local du_output

    du_output="$(du -ab --max-depth=1 -- "$dir" 2>/dev/null || true)"
    [[ -n "$du_output" ]] || return 0

    local -a raw=() filtered=()
    local line bytes path
    while IFS= read -r line; do
        raw+=("$line")
    done <<< "$du_output"

    for line in "${raw[@]}"; do
        bytes="${line%%$'\t'*}"
        path="${line#*$'\t'}"

        [[ "$path" == "$dir" ]] && continue   # skip the self-total line

        if [[ "$DIRS_ONLY" == true && ! -d "$path" ]]; then
            continue
        fi

        if [[ -n "$FILTER" && ! -d "$path" ]]; then
            [[ "${path##*/}" =~ $FILTER ]] || continue
        fi

        filtered+=("$line")
    done

    (( ${#filtered[@]} == 0 )) && return 0
    printf '%s\n' "${filtered[@]}" | sort_entries | limit_top
}

# sort_entries
# Reads "bytes<TAB>path" from stdin and sorts it according to SORT_KEY/REVERSE.
sort_entries() {
    local field sort_flags
    if [[ "$SORT_KEY" == "size" ]]; then
        field=1
        sort_flags=(-n)
    else
        field=2
        sort_flags=()
    fi
    if [[ "$REVERSE" == false && "$SORT_KEY" == "size" ]]; then
        # Default for size is biggest-first, i.e. numerically reversed.
        sort_flags+=(-r)
    elif [[ "$REVERSE" == true && "$SORT_KEY" == "name" ]]; then
        sort_flags+=(-r)
    fi
    sort -t $'\t' -k"${field},${field}" "${sort_flags[@]}"
}

# limit_top
# Reads lines from stdin, prints only the first TOP_N (0 = unlimited).
limit_top() {
    if [[ "$TOP_N" -eq 0 ]]; then
        cat
    else
        head -n "$TOP_N"
    fi
}

# analyze_dir DIR DEPTH PREFIX
# Recursively prints a tree of DIR's contents.
#   DEPTH  - current recursion depth (root's children are depth 1)
#   PREFIX - the tree-drawing prefix ("│   " / "    ") accumulated so far
analyze_dir() {
    local dir="$1" depth="$2" prefix="$3"
    dir="$(strip_trailing_slash "$dir")"

    local -a entries=()
    local line
    while IFS= read -r line; do
        [[ -n "$line" ]] && entries+=("$line")
    done < <(get_children "$dir")

    local count="${#entries[@]}"
    local i
    for (( i = 0; i < count; i++ )); do
        local bytes name connector child_prefix color reset
        bytes="${entries[$i]%%$'\t'*}"
        name="${entries[$i]#*$'\t'}"

        if (( i == count - 1 )); then
            connector="└── "
            child_prefix="${prefix}    "
        else
            connector="├── "
            child_prefix="${prefix}│   "
        fi

        color=""
        reset=""
        if [[ "$USE_COLOR" == true ]]; then
            if [[ -d "$name" ]]; then
                color=$'\033[1;34m'   # bold blue for directories
            fi
            reset=$'\033[0m'
        fi

        printf '%s%s%-8s %s%s%s\n' \
            "$prefix" "$connector" "$(format_size "$bytes")" "$color" "${name##*/}" "$reset"

        if [[ -d "$name" && "$depth" -lt "$MAX_DEPTH" ]]; then
            analyze_dir "$name" "$((depth + 1))" "$child_prefix"
        fi
    done
}

# --- Main --------------------------------------------------------------------

main() {
    parse_args "$@"
    [[ "$DEBUG" == true ]] && set -x

    check_dependencies

    [[ -e "$TARGET_PATH" ]] || die "path does not exist: $TARGET_PATH" "$EXIT_ERROR"
    [[ -d "$TARGET_PATH" ]] || die "path is not a directory: $TARGET_PATH" "$EXIT_ERROR"
    [[ -r "$TARGET_PATH" ]] || die "path is not readable: $TARGET_PATH" "$EXIT_ERROR"

    TARGET_PATH="$(strip_trailing_slash "$TARGET_PATH")"

    local root_bytes
    root_bytes="$(du -sb -- "$TARGET_PATH" 2>/dev/null | cut -f1)"
    [[ -n "$root_bytes" ]] || root_bytes=0

    printf '%s %s\n' "$(format_size "$root_bytes")" "$TARGET_PATH"

    if [[ "$MAX_DEPTH" -gt 0 ]]; then
        analyze_dir "$TARGET_PATH" 1 ""
    fi
}

main "$@"
```
