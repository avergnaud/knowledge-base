# Manage Knowledge Base Nodes Skill

## Purpose
This skill enables Claude to create, edit, and delete nodes in the knowledge base located at `C:\Users\a.vergnaud\dev\knowledge-base`.

## Node Structure

Each node follows this structure:
```
nodes/
└── <uuid>/
    ├── metadata.json
    ├── en.md
    ├── fr.md
    └── es.md
```

### Metadata Schema
```json
{
  "id": "uuid-v4",
  "title": "Node Title",
  "languages": ["en", "fr", "es"],
  "createdAt": "ISO-8601 timestamp",
  "updatedAt": "ISO-8601 timestamp",
  "tags": ["tag1", "tag2"],
  "authors": ["author-name"]
}
```

### Content Template
Each language file (en.md, fr.md, es.md) follows this structure:
```markdown
# [Title]

[Content]
```

## Supported Languages
- English (en)
- French (fr)
- Spanish (es)

## Operations

### 1. Create Node

**User Request Patterns:**
- "Create a new node about [topic]"
- "Add a node titled [title]"
- "Create a knowledge node for [concept]"

**Workflow:**
1. Ask user for:
   - Title (required)
   - Content (required)
   - Tags (optional, comma-separated)
   - Author (optional, defaults to "unknown")

2. Generate UUID v4 for the node

3. Translate content to all three languages (en, fr, es)

4. Create directory: `nodes/<uuid>/`

5. Create `metadata.json` with:
   - Generated UUID
   - User-provided title
   - All three languages: ["en", "fr", "es"]
   - Current timestamp for createdAt and updatedAt
   - User-provided tags (or empty array)
   - User-provided author (or ["unknown"])

6. Create three content files:
   - `en.md` with English title and content
   - `fr.md` with French translation
   - `es.md` with Spanish translation

7. Validate:
   - UUID in metadata.json matches directory name
   - All three language files exist
   - metadata.json is valid JSON

8. Check if git is initialized in the repository
   - If yes, suggest: `git add nodes/<uuid>/ && git commit -m "Add node: [title]"`
   - If no, inform user that git is not initialized

9. Confirm completion with node UUID and location

### 2. Edit Node

**User Request Patterns:**
- "Edit node [uuid or title]"
- "Update the content of [node]"
- "Modify node [identifier]"

**Workflow:**
1. Identify the node by UUID or title:
   - If UUID provided, use directly
   - If title provided, search nodes/ for matching title in metadata.json
   - If multiple matches or not found, ask user to clarify

2. Show current content and ask what to change:
   - Title
   - Content (specify which language or all)
   - Tags
   - Author

3. For content changes:
   - If user edits one language, offer to translate to other languages
   - If user requests all languages, translate automatically

4. Update the specified fields

5. Update `updatedAt` timestamp in metadata.json

6. Validate:
   - UUID still matches directory name
   - All referenced languages have files
   - metadata.json is valid JSON

7. Suggest git commit: `git add nodes/<uuid>/ && git commit -m "Update node: [title]"`

8. Confirm completion

### 3. Delete Node

**User Request Patterns:**
- "Delete node [uuid or title]"
- "Remove node [identifier]"

**Workflow:**
1. Identify the node by UUID or title (same as edit)

2. Show node title and ask for confirmation:
   "Are you sure you want to delete '[title]' (UUID: [uuid])?"

3. If confirmed:
   - Delete entire directory `nodes/<uuid>/`
   - Suggest git commit: `git add nodes/<uuid>/ && git commit -m "Delete node: [title]"`

4. If not confirmed, cancel operation

5. Confirm completion or cancellation

## Translation Guidelines

When translating content:
- Maintain the same structure (title + content)
- Preserve markdown formatting
- Keep technical terms consistent
- For French: use formal register unless context suggests otherwise
- For Spanish: use neutral Spanish (not regional variants)

## Validation Rules

Before completing any operation:
1. Verify UUID format is valid v4
2. Confirm UUID in metadata.json == directory name
3. Check all language files exist for languages in metadata
4. Ensure metadata.json is valid JSON
5. Verify timestamps are ISO-8601 format

## Error Handling

- If UUID already exists: inform user and suggest different action
- If node not found: list available nodes or suggest search
- If translation fails: create files with [TRANSLATION NEEDED] placeholder
- If git operations fail: complete the file operation but inform user
- If validation fails: show specific error and do not complete operation

## File Locations

- Repository root: `C:\Users\a.vergnaud\dev\knowledge-base`
- Nodes directory: `C:\Users\a.vergnaud\dev\knowledge-base\nodes`
- Individual node: `C:\Users\a.vergnaud\dev\knowledge-base\nodes/<uuid>/`

## Important Notes

- This skill does NOT handle `links.json` - that is managed by the separate "manage-links" skill
- Always validate before completing operations
- Suggest git commits but do not execute them automatically
- Use filesystem tools to interact with the actual directory structure
- When translating, be accurate and maintain the original meaning

## Example Interaction

**User:** "Create a new node about modularity in software"

**Claude:**
"I'll create a new node about modularity in software. Let me gather some information:

What content would you like for this node? Please provide the main text.

Also, please specify:
- Tags (optional, comma-separated): 
- Author (optional):"

**User:** "Content: Modularity is the principle of dividing software into separate components. Tags: software-engineering, design-principles. Author: A.Vergnaud"

**Claude:**
"Perfect! I'll create the node now with:
- Title: Modularity in Software
- Content: [as provided]
- Tags: software-engineering, design-principles
- Author: A.Vergnaud
- Languages: English, French, Spanish (automatically translated)

[Creates node with UUID 7c9e6679-7425-40de-944b-e07fc1f90ae7]

✓ Node created successfully!
- UUID: 7c9e6679-7425-40de-944b-e07fc1f90ae7
- Location: C:\Users\a.vergnaud\dev\knowledge-base\nodes\7c9e6679-7425-40de-944b-e07fc1f90ae7
- Files created: metadata.json, en.md, fr.md, es.md

Git is initialized. To commit this change, run:
`git add nodes/7c9e6679-7425-40de-944b-e07fc1f90ae7 && git commit -m "Add node: Modularity in Software"`"
