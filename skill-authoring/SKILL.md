---
name: skill-authoring
description: Guides creation of Claude Agent Skills following Anthropic best practices. Covers SKILL.md structure, YAML frontmatter, progressive disclosure, and content organization. Use when creating new skills, reviewing skill files, or helping users write custom skills.
---

# Skill Authoring Guide

## Required Structure

```
skill-name/
└── SKILL.md
```

For complex skills with additional resources:

```
skill-name/
├── SKILL.md           # Main instructions (<500 lines)
├── reference.md       # Detailed docs (loaded as needed)
└── scripts/
    └── utility.py     # Executed, not loaded into context
```

## SKILL.md Format

```yaml
---
name: skill-name
description: What it does and when to use it. Third person. Max 1024 chars.
---

# Skill Title

[Instructions and examples]
```

### Frontmatter Rules

**name** (required):
- Max 64 characters
- Lowercase letters, numbers, hyphens only
- No "anthropic" or "claude"

**description** (required):
- Max 1024 characters
- Third person ("Processes files..." not "I process...")
- Include WHAT it does AND WHEN to trigger it

## Writing Principles

### Be Concise

Claude is already knowledgeable. Only add context it doesn't have.

```markdown
// 🔴 Too verbose
PDF (Portable Document Format) files are a common format...
There are many libraries available...

// ✅ Concise
Use pdfplumber for text extraction:
```python
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
```

### Set Appropriate Freedom

| Task Type | Freedom Level | Approach |
|-----------|---------------|----------|
| Fragile/critical operations | Low | Exact scripts, no variation |
| Preferred patterns exist | Medium | Templates with parameters |
| Multiple valid approaches | High | Guidelines, let Claude decide |

### Progressive Disclosure

Keep SKILL.md as overview. Reference detailed content:

```markdown
## Quick start
[Essential instructions here]

## Advanced features
See [ADVANCED.md](ADVANCED.md) for complex workflows.
```

**Keep references one level deep** - don't nest references in referenced files.

## Description Examples

```yaml
# Good - specific, includes triggers
description: Extracts text and tables from PDF files, fills forms, merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.

# Good - clear capability and trigger
description: Generates commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.

# Bad - too vague
description: Helps with documents.
```

## Content Patterns

### Decision Tables

```markdown
| Situation | Solution |
|-----------|----------|
| Need X | Do Y |
| Need Z | Do W |
```

### Anti-Pattern Format

```markdown
### Problem Name

```language
// 🔴 Avoid
[bad code]

// ✅ Better
[good code]
```
```

### Workflows with Checklists

For complex multi-step tasks:

````markdown
## Workflow

Copy and track progress:

```
- [ ] Step 1: Analyze input
- [ ] Step 2: Validate
- [ ] Step 3: Execute
- [ ] Step 4: Verify
```

**Step 1: Analyze input**
[details]
````

### Feedback Loops

```markdown
1. Make changes
2. Run validation: `python validate.py`
3. If errors: fix and return to step 2
4. Only proceed when validation passes
```

## Utility Scripts

Scripts execute without loading into context - more efficient than generated code.

```markdown
**analyze.py**: Extract form fields

```bash
python scripts/analyze.py input.pdf > fields.json
```
```

Make clear whether Claude should **execute** (common) or **read as reference** (rare).

## Avoid

- Time-sensitive information (use "old patterns" section if needed)
- Windows-style paths (`\`) - always use forward slashes
- Multiple equivalent options without a clear default
- Vague descriptions ("helps with things")
- Deeply nested file references
- Magic numbers without explanation

## Checklist

Before publishing:

- [ ] `name`: lowercase, hyphens, ≤64 chars
- [ ] `description`: third person, what + when, ≤1024 chars
- [ ] SKILL.md body <500 lines
- [ ] No unnecessary explanations (Claude knows basics)
- [ ] References one level deep
- [ ] Tested with actual usage scenarios
