---
description: Audit and standardize slash command frontmatter and structure
allowed-tools: [Read, Glob, Grep]
argument-hint: "[audit|apply] [command-name] (use --help for details)"
---

# Slash Command Audit & Standardization

Audit all slash commands in `.claude/commands/` and propose frontmatter corrections to increase execution consistency while preserving the original prompt intent.

All frontmatter and configuration decisions must align with [Anthropic's documented standards](https://code.claude.com/docs/en/slash-commands).

## Quick Reference

aFor detailed usage: run with `--help`

---

**When `--help` or `-h` is in arguments:**

Do NOT execute. Show ONLY this help block with NO additional commentary:

```
Usage: /claude-slash-command [audit|apply] [command-name] [OPTIONS]

Audit and standardize slash command frontmatter and structure.

Commands:
  audit                    Analyze all commands in .claude/commands/ for issues
  apply [command-name]     Apply proposed frontmatter to a specific command

Options:
  --help, -h              Show this help message

Examples:
  /claude-slash-command audit
  /claude-slash-command apply playbook-generator
  /claude-slash-command audit --help
```

---

## Task

**DO**: Fix/add frontmatter configuration, standardize structure
**DO NOT**: Rewrite prompt content or change intended behavior

## Step 1: Inventory

**When analyzing target command(s):**

Run this to list all command files:
```bash
find .claude/commands -name "*.md" -type f | sort
```

## Step 2: For Each Command, Extract Current State
For each file, output:
- Filename
- Has frontmatter? (yes/no)
- Current frontmatter fields (if any)
- First executable action in the prompt body (what should it DO first?)
- Post-execution behavior (what happens after?)

## Step 3: Propose Standardized Frontmatter

### Official Frontmatter Fields (per Anthropic docs)

```yaml
---
allowed-tools: [tools Claude can use during execution]
argument-hint: [displayed in /help to show expected arguments]
description: [shown in /help output, enables SlashCommand tool discovery]
model: [optional - override model, e.g., claude-3-5-haiku-20241022]
disable-model-invocation: [optional - true prevents Claude from auto-invoking]
---
```

**Note**: The `description` field is REQUIRED for commands to be discoverable by Claude's SlashCommand tool. Without it, Claude cannot programmatically invoke the command.

Each command should have at minimum:

```yaml
---
allowed-tools: [list ONLY the tools needed for execution]
description: [one-line description for /help output]
argument-hint: [optional - show expected args]
---
```

### Frontmatter Rules

**allowed-tools**: Constrain to ONLY what's needed
- If command runs a script: `Bash(.claude/commands/scriptname.sh:*)`
- If command reads files: `Read, Glob`
- If command runs specific CLI: `Bash(npm test:*), Bash(git:*)`
- Wildcard patterns: `Bash(git add:*), Bash(git status:*), Bash(git commit:*)`
- NEVER leave blank if the command has an executable step

**description**: Required
- Used in `/help` output
- Keep under 60 chars
- Format: "Verb + object" (e.g., "Search KB via Typesense")

**argument-hint**: If command uses $ARGUMENTS
- Show expected format: `[query]`, `[file] [options]`, `[issue-number]`

## Step 4: Propose Body Structure (if needed)

### Official Body Syntax (per Anthropic docs)

**Arguments**:
- `$ARGUMENTS` - all arguments as single string
- `$1`, `$2`, `$3` - positional arguments

**Dynamic Content** (executed when command loads):
- `!`\`command\` - runs bash command, injects output into prompt
- Example: `!`\`git status\` injects current git status

**File References**:
- Standard markdown file references work

If the command body is unclear about execution vs. collaboration, propose restructuring to:

```markdown
## EXECUTE
[The non-negotiable first action - run this immediately]

## THEN
[What to do with output - present, summarize, wait for input]

## DO NOT
[Prevent eager-helper improvisation if needed]
```

Only propose body restructuring if:
- The first action is ambiguous
- There's no clear separation between "execute" and "collaborate"
- The command has history of being ignored/improvised around

## Output Format

For each command, output:

```
### /command-name

**File**: .claude/commands/command-name.md

**Current frontmatter**:
[show existing or "NONE"]

**Proposed frontmatter**:
---
allowed-tools: [specific tools]
description: [description]
argument-hint: [if applicable]
---

**Body changes**: None | [describe minimal restructuring]

**Rationale**: [why these changes improve consistency]
```

## Step 5: Summary Report

After all commands audited:

| Command | Had Frontmatter | allowed-tools Added | Body Restructured |
|---------|-----------------|---------------------|-------------------|
| /name   | yes/no          | yes/no              | yes/no            |

## Step 6: Help Flag Standard

All executable commands (those that invoke shell scripts) MUST support `--help` flag.

### Implementation Requirements

**1. Script must handle --help as first check:**

```bash
#!/usr/bin/env bash

# Help flag handler (MUST be first, before any other logic)
while [[ $# -gt 0 ]]; do
  case "$1" in
    --help|-h)
      cat << 'EOF'
Usage: /command-name <args> [OPTIONS]

<One-line description from frontmatter>

Options:
  --option1 VALUE          Description
  --flag2                  Description
  --help, -h              Show this help message

Examples:
  /command-name arg1
  /command-name arg1 --option1 value
EOF
      exit 0
      ;;
    # ... other argument parsing
  esac
done
```

**2. Frontmatter must indicate help availability:**

```yaml
argument-hint: "[args] [--options] (use --help for details)"
```

**3. Command body must include help directive:**

Add FIRST, immediately after the heading (BEFORE any execution warnings):
```markdown
**When `--help` or `-h` is in $ARGUMENTS:**

Do NOT execute the script. Show ONLY this help block with NO additional commentary, explanation, or follow-up text:

```
Usage: /command-name <args> [OPTIONS]

<One-line description>

Options:
  --option1 VALUE          Description
  --option2                Description
  --help, -h               Show this help message

Examples:
  /command-name arg1
  /command-name arg1 --option1 value
```

**⚠️ EXECUTION MODE: For all other arguments, this is a DIRECT COMMAND INVOCATION. You MUST immediately execute `.claude/commands/<script>.sh` with the provided arguments.**
```

Format requirements:
- Use triple backticks with no language identifier
- Match exact formatting from script's --help output
- Include: Usage line, description, Options section, Examples section
- Maintain alignment spacing from script

**If help section does not exist:**
- Read the command's referenced playbook file (if mentioned in the command body)
- Read the script's --help handler output
- Build the help block using the script's exact output
- Copy formatting, spacing, and content verbatim from script

**Example (from /search):**
```markdown
**When `--help` or `-h` is in $ARGUMENTS:**

Do NOT execute the script. Show ONLY this help block with NO additional commentary, explanation, or follow-up text:

```
Usage: /search <query> [OPTIONS]

Query the Typesense KB search engine using navigation-first retrieval.

Options:
  --limit N                    Limit results (1-100, default: 10)
  --intent TYPE                Filter by intent: concept|procedure|reference|troubleshooting
  --complexity LEVEL           Filter by level: beginner|intermediate|advanced
  --content-type TYPE          Filter by type: prose|code-heavy|mixed
  --authoritative-only         Only production-ready code
  --help, -h                   Show this help message

Examples:
  /search "aws lambda"
  /search "error handling" --intent troubleshooting --complexity intermediate
  /search "async patterns" --authoritative-only --limit 20
```

**⚠️ EXECUTION MODE: For all other arguments, this is a DIRECT COMMAND INVOCATION. You MUST immediately execute `.claude/commands/search.sh` with the provided arguments.**
```

Also mention in Quick Start section:
```markdown
## Quick Start

For detailed usage: `/command --help`
```

### Validation Checklist

For each command with a shell script:
- [ ] Script includes `--help` and `-h` flag handling
- [ ] Help handler is FIRST in argument parsing (before other logic)
- [ ] Help output includes: Usage line, description, options, examples
- [ ] Script exits with code 0 after showing help
- [ ] Frontmatter `argument-hint` mentions `--help`
- [ ] Command body includes CRITICAL help directive (show ONLY script output)
- [ ] Command body references `--help` in Quick Start section

### Benefits

- **For users**: Quick reference without reading full docs
- **For Claude**: Exact syntax reminder before execution
- **Consistency**: Standard CLI convention users expect

### Terminal Output Standards

When displaying help text in terminal (outside VS Code IDE):

**MUST provide this level of detail:**
- Command usage syntax with all options
- Description of what the command does
- All available options with descriptions
- Concrete examples showing real usage patterns
- Proper formatting (spacing, alignment, sections)

**Format requirements:**
- Use the embedded help block verbatim (from command markdown)
- Maintain all spacing and alignment from the original
- Show complete help text, not summaries
- Include all sections: Usage, Description, Options, Examples
- **NO additional commentary** (no "I see you've invoked...", no "Ready to search...", no follow-up prompts)
- Output ONLY the help block, nothing before or after

**Why this matters:**
- Terminal users need complete reference without opening files
- Detailed help enables self-service without trial-and-error
- Consistent formatting builds user confidence
- Examples show real usage patterns, not placeholders

## Implementation

When applying this audit:
1. Analyze the target command(s) for frontmatter completeness
2. Propose standardized frontmatter
3. Validate help flag implementation (if applicable)
4. Ask: "Apply these changes?" before modifying any files
5. Do NOT rewrite content - only standardize structure
