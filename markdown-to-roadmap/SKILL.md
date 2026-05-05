---
name: markdown-to-roadmap
description: |
  Convert various input formats (markdown, JSON, CSV, text) into a roadmap JSON file.
  Use when the user has a roadmap description in markdown, a CSV of roadmap items, a JSON
  file with wrong structure, or any raw text describing a learning plan. The skill will
  parse the input, validate required fields, and output properly formatted roadmap JSON
  matching the app's schema. Perfect for importing roadmaps from other sources or creating
  roadmaps from notes.
triggers:
  - "create a roadmap from"
  - "convert to roadmap"
  - "import roadmap from"
  - "make a roadmap from"
  - "roadmap from markdown"
  - "roadmap from notes"
---

# Markdown to Roadmap Converter

This skill converts various input formats into the standard roadmap JSON format for this app.

## Output Format

The output roadmap JSON must follow this schema:

```json
{
  "title": "string (required)",
  "description": "string (required)",
  "items": [
    {
      "weekNumber": "integer (required)",
      "title": "string (required)",
      "description": "string (required)",
      "resourceList": [
        {
          "type": "book|essay|video|course|article|podcast (required)",
          "subject": "string (required)",
          "title": "string (required)",
          "author": "string (required)",
          "coverUrl": "string (optional)"
        }
      ],
      "resourceListStatus": ["boolean array, defaults to all false"],
      "status": "backlog|in_progress|completed (defaults to backlog)"
    }
  ]
}
```

## Steps

### 1. Detect Input Format

Try to detect the format from the file extension:
- `.json` → JSON
- `.csv` → CSV
- `.md` or `.txt` → Markdown/text
- `.xlsx` → Excel (if openpyxl available)

If format is unclear, assume markdown/text.

### 2. Parse Input

**Markdown parsing:**
- First `# Heading` becomes `title`
- First paragraph after title becomes `description`
- `## Week N` or `### Week N` headers become roadmap items
- Under each week header:
  - First line is `title`
  - Content until next week header is `description`
  - `- [ ] Resource` or `- Resource` lines under a week are parsed as resources
  - `status` can be indicated with `- [x]` (completed), `- [-]` (in_progress), `- [ ]` (backlog)

**JSON parsing:**
- If it's an array, treat as items
- If it has `items` or `roadmapItems` key, use that
- Map common field names (e.g., `week` → `weekNumber`, `resources` → `resourceList`)

**CSV parsing:**
- Headers: weekNumber, title, description, status, resource_type, resource_subject, resource_title, resource_author
- Resources are embedded as JSON arrays in resource fields

**Text parsing:**
- Split by double newlines or numbered patterns like "1. Week 1: Title"
- Extract fields using common patterns

### 3. Validate

Check required fields:
- `title` must be non-empty string
- `description` must be non-empty string
- Each item must have `weekNumber`, `title`, `description`
- Each resource must have `type`, `subject`, `title`, `author`

### 4. Output

If validation passes:
- Write the roadmap JSON to `roadmap.json` in the current directory
- Tell user "Created roadmap.json with N items"

If validation fails:
- List all errors with line numbers or item numbers
- Ask user to fix the input

## Resource Type Mapping

When user writes resource type in plain text, map to standard values:
- "book", "reading" → "book"
- "video", "youtube", "talk" → "video"
- "course", "tutorial", "lesson" → "course"
- "article", "blog", "post" → "article"
- "essay", "write-up" → "essay"
- "podcast", "audio" → "podcast"

## Status Mapping

- "done", "completed", "x", "[x]" → "completed"
- "in progress", "current", "[-]" → "in_progress"
- "todo", "backlog", "[ ]", "pending" → "backlog"