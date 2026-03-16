---
name: qa-to-flashcards
description: Convert question and answer pairs to JSON format for flashcard app import. Use this skill whenever the user wants to create flashcards from Q&A content - whether freeform text, CSV data, or any Q&A format. The flashcard app requires tags for each card, so extract tags from input or ask the user. Output is saved to a JSON file.
---

# Q&A to Flashcards JSON Converter

This skill converts question and answer pairs into a JSON file that can be imported into the flashcard application. The flashcard app requires each card to have at least one tag.

## Input Detection

Detect the format of the user's input:

1. **CSV format**: If input contains lines with commas (e.g., `question,answer,tags`), parse as CSV
2. **Freeform text**: If input contains Q:/Question:/Q: or A:/Answer:/A: markers, parse accordingly
3. **Line-by-line**: If input is multiple lines without markers, treat odd lines as questions and even lines as answers

### CSV Parsing
- Expected columns: `question`, `answer`, `tags` (tags can be comma-separated)
- If tags column is missing, flag cards that need tags

### Freeform Parsing
Look for patterns like:
- `Q: What is X? A: X is Y`
- `Question: ... Answer: ...`
- `Q: ... A: ...`
- `What is X? | X is Y` (pipe-separated)

Extract question (front) and answer (back) from each pair.

## Code Example Handling

**When answers contain code examples, preserve them exactly!** Code examples make excellent flashcards for programming topics.

### Detecting Code in Answers
- Answers that contain code snippets should preserve the code formatting
- Look for code blocks using markdown backticks (\`\`\`) or inline code (`)

### How to Handle Code
1. **Multi-line code blocks**: Preserve the entire code block with proper formatting
2. **Inline code**: Keep inline code within backticks
3. **Code with explanations**: If an answer has both explanation AND code, include both

### Example Input with Code

**Input:**
```
Q: How do you reverse a string in Python?
A: Use slicing with [::-1]:
```python
s = "hello"
reversed_s = s[::-1]
```
```

**Output JSON:**
```json
[
  {
    "front": "How do you reverse a string in Python?",
    "back": "Use slicing with [::-1]:\n```python\ns = \"hello\"\nreversed_s = s[::-1]\n```",
    "tags": ["python", "strings"]
  }
]
```

### Tips for Code in Flashcards
- Keep code examples short and focused
- Include the language hint (e.g., \`\`\`python) for syntax highlighting
- If code is too long, extract the key concept into a shorter example

## Tag Extraction

Tags are **required** for each card. Extract tags in this order:

1. **From explicit tags in input**: Look for patterns like `tags: science, math` or `[science, math]` or `tag: history`
2. **From context**: If the input mentions a topic (e.g., "Chapter 1: Biology", "French vocabulary"), use that as a tag
3. **From code content**: If the answer contains code, infer tags from the language (e.g., "python", "javascript", "sql")
4. **Ask user**: If no tags found, ask the user to provide tags for the cards

When asking user for tags:
- Ask once for all cards, or per-card? Ask: "Should these tags apply to all cards, or different tags per card?"
- Accept comma-separated or space-separated tags

## JSON Output Format

Generate JSON matching this schema:
```json
[
  {
    "front": "Question text",
    "back": "Answer text",
    "tags": ["tag1", "tag2"]
  }
]
```

**Important:** Preserve all whitespace, newlines, and code formatting in the answer text. Use `\n` for newlines in JSON strings.

Requirements:
- Each card MUST have at least one tag
- Tags should be lowercase, trimmed
- Save output to `flashcards.json` in the current directory (or user-specified filename)

## Output

Save the JSON to a file and confirm to the user:
- Number of cards created
- Tags used
- File location

Example output message:
```
Created 5 flashcards with tags: [science, history]
Saved to: flashcards.json
```

## Error Handling

- If no valid Q&A pairs found: Tell user "Could not detect any question/answer pairs. Please format as 'Q: ... A: ...' or provide CSV."
- If any card missing tags after extraction: Ask user to provide tags before generating JSON
