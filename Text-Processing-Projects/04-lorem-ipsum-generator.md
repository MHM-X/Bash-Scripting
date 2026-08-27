# 📂 Lorem IPSUM Generator
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/bin/bash

set -euo pipefail

LANGUAGE="la"
LENGTH=20

ENGLISH=(
    "the" "quick" "brown" "fox" "jumps"
    "over" "the" "lazy" "dog" "while"
    "designers" "create" "beautiful" "modern"
    "websites" "using" "simple" "creative"
    "ideas" "for" "users" "around" "the" "world"
)

LATIN=(
    "lorem" "ipsum" "dolor" "sit" "amet"
    "consectetur" "adipiscing" "elit" "sed"
    "do" "eiusmod" "tempor" "incididunt"
    "ut" "labore" "et" "dolore" "magna"
    "aliqua" "enim" "ad" "minim" "veniam"
)

SPANISH=(
    "el" "rápido" "zorro" "marrón" "salta"
    "sobre" "el" "perro" "perezoso" "mientras"
    "los" "diseñadores" "crean" "sitios"
    "web" "modernos" "y" "hermosos"
    "para" "los" "usuarios"
)

FRENCH=(
    "le" "renard" "brun" "rapide" "saute"
    "par-dessus" "le" "chien" "paresseux"
    "pendant" "que" "les" "designers"
    "créent" "des" "sites" "modernes"
    "et" "élégants" "pour" "les" "utilisateurs"
)

ARABIC=(
    "هذا" "نص" "تجريبي" "يستخدم" "لملء"
    "التصميم" "بشكل" "مؤقت" "حتى" "يتم"
    "إضافة" "المحتوى" "الحقيقي" "إلى"
    "المشروع" "بطريقة" "بسيطة" "وواضحة"
)

usage() {
    echo "Usage: $0 [-n length] [-l language] [-h]"
    echo
    echo "Options:"
    echo "  -n LENGTH    Number of words"
    echo "  -l LANGUAGE  Language: la, en, es, fr, ar"
    echo "  -h           Show help"
}

while getopts "n:l:h" opt; do
    case "$opt" in
        n)
            LENGTH="$OPTARG"
            ;;
        l)
            LANGUAGE="$OPTARG"
            ;;
        h)
            usage
            exit 0
            ;;
        \?)
            usage
            exit 1
            ;;
    esac
done

if ! [[ "$LENGTH" =~ ^[0-9]+$ ]] || [[ "$LENGTH" -le 0 ]]; then
    echo "Error: length must be a positive number."
    exit 1
fi

case "$LANGUAGE" in

    la)
        WORDS=("${LATIN[@]}")
        ;;

    en)
        WORDS=("${ENGLISH[@]}")
        ;;

    es)
        WORDS=("${SPANISH[@]}")
        ;;

    fr)
        WORDS=("${FRENCH[@]}")
        ;;

    ar)
        WORDS=("${ARABIC[@]}")
        ;;

    *)
        echo "Error: unsupported language '$LANGUAGE'."
        echo "Supported languages: la, en, es, fr, ar"
        exit 1
        ;;
esac

for ((i = 0; i < LENGTH; i++)); do

    index=$((RANDOM % ${#WORDS[@]}))

    printf "%s" "${WORDS[$index]}"

    if [[ "$i" -lt $((LENGTH - 1)) ]]; then
        printf " "
    fi

done

echo
```
