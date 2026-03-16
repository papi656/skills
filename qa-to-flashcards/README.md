# qa-to-flashcards

Convert question and answer pairs to JSON format for flashcard app import. This is a custom app that I build for myself

## Description

This skill converts question and answer pairs into a JSON file that can be imported into a flashcard application. The flashcard app requires each card to have at least one tag.

## Features

- **Multiple Input Formats**: Supports CSV, freeform text (Q:/A: markers), and line-by-line formats
- **Code Preservation**: Keeps code examples intact with proper formatting
- **Automatic Tag Extraction**: Extracts tags from input context or prompts user for tags
- **JSON Output**: Generates properly formatted JSON for flashcard app import

## Usage

Load this skill when the user wants to create flashcards from Q&A content. The skill will:
1. Detect the format of the user's input
2. Parse questions and answers
3. Extract or ask for tags
4. Generate JSON output
5. Save to `flashcards.json`

## Input Formats

### CSV
```
question,answer,tags
```

### Freeform
```
Q: What is X? A: X is Y
```

### Line-by-line
```
Question 1
Answer 1
Question 2
Answer 2
```

## Requirements

- Each card must have at least one tag
- Tags should be lowercase, trimmed

## Files

- `SKILL.md` - Main skill definition
- `evals/evals.json` - Evaluation test cases

## License

MIT
