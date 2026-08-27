# 📂 Text summarizer
## Description
Automatically organizes files into categorized subfolders based on their file extensions.

## Code
```bash
#!/bin/bash

set -euo pipefail

INPUT=""
SENTENCES=3

usage() {
    echo "Usage: $0 -i file [-n number]"
    echo
    echo "Options:"
    echo "  -i FILE    Input text file"
    echo "  -n NUM     Number of sentences in summary"
    echo "  -h         Show help"
}

while getopts "i:n:h" opt; do
    case "$opt" in
        i)
            INPUT="$OPTARG"
            ;;
        n)
            SENTENCES="$OPTARG"
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

if [[ -z "$INPUT" ]]; then
    echo "Error: input file is required."
    exit 1
fi

if [[ ! -f "$INPUT" ]]; then
    echo "Error: file '$INPUT' does not exist."
    exit 1
fi

if ! [[ "$SENTENCES" =~ ^[0-9]+$ ]] || [[ "$SENTENCES" -le 0 ]]; then
    echo "Error: number of sentences must be positive."
    exit 1
fi

awk -v limit="$SENTENCES" '
BEGIN {
    IGNORECASE = 1

    stopwords["the"] = 1
    stopwords["a"] = 1
    stopwords["an"] = 1
    stopwords["and"] = 1
    stopwords["or"] = 1
    stopwords["but"] = 1
    stopwords["is"] = 1
    stopwords["are"] = 1
    stopwords["was"] = 1
    stopwords["were"] = 1
    stopwords["to"] = 1
    stopwords["of"] = 1
    stopwords["in"] = 1
    stopwords["on"] = 1
    stopwords["for"] = 1
    stopwords["with"] = 1
    stopwords["this"] = 1
    stopwords["that"] = 1
    stopwords["it"] = 1
}

{
    text = text " " $0
}

END {

    sentence_count = split(text, sentences, /[.!?]+[[:space:]]*/)

    for (i = 1; i <= sentence_count; i++) {

        sentence = sentences[i]

        if (sentence ~ /^[[:space:]]*$/)
            continue

        word_count = split(sentence, words, /[^[:alnum:]]+/)

        score = 0

        for (j = 1; j <= word_count; j++) {

            word = tolower(words[j])

            if (word == "")
                continue

            if (!(word in stopwords))
                score++
        }

        scores[i] = score
    }

    for (x = 1; x <= limit; x++) {

        best = 0
        best_score = -1

        for (i = 1; i <= sentence_count; i++) {

            if (scores[i] > best_score && !selected[i]) {
                best = i
                best_score = scores[i]
            }
        }

        if (best == 0)
            break

        selected[best] = 1
        chosen[x] = best
    }

    for (i = 1; i <= limit; i++) {

        sentence_number = chosen[i]

        if (sentence_number == 0)
            continue

        print sentences[sentence_number] "."
    }
}
' "$INPUT"
```
