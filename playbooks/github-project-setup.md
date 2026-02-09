# GitHub Project Setup Playbook

> **Purpose**: Initialize a new GitHub project with standardized directory structure, configuration files, and essential tooling setup
> **Version**: 1.0
> **Last Updated**: 2026-02-08

---

## 1. Purpose

This playbook defines the canonical process for setting up a new GitHub project repository with:

- Consistent directory structure
- Essential configuration files (`.gitignore`, `.editorconfig`, etc.)
- Claude Code integration (`.claude/` directory)
- Development environment setup
- GitHub-specific configurations (`.github/` directory)
- Documentation scaffolding

---

## 2. Core Principles

All project setups MUST:

1. Follow lowercase filename conventions (kebab-case)
2. Include comprehensive `.gitignore` from start
3. Set up `.claude/` directory for AI-assisted development
4. Configure `.github/` for workflows and templates
5. Initialize git repository before creating files
6. Create essential documentation files
7. Follow Bryan Developer Profile standards
8. Avoid creating unnecessary files

---

## 3. Prerequisites

Before running this playbook:

- [ ] Project name decided (lowercase, kebab-case recommended)
- [ ] Project purpose defined (one-line description)
- [ ] Primary language/framework identified
- [ ] Target deployment environment known (if applicable)
- [ ] License type decided (MIT, Apache, proprietary, etc.)

---

## 4. Directory Structure (Canonical Layout)

```
<project-root>/
├── .git/                           # Git repository (initialized first)
├── .gitignore                      # Git ignore rules
├── .editorconfig                   # Editor configuration
├── .env.example                    # Environment variables template
│
├── _build/                         # Build artifacts (gitignored except README)
│   └── README.md                   # Build directory documentation
├── _private/                       # Private/local files (gitignored except README)
│   └── README.md                   # Private directory documentation
│
├── .claude/                        # Claude Code integration
│   ├── CLAUDE.md                   # Project-specific Claude instructions
│   ├── commands/                   # Slash commands
│   │   ├── *.md                    # Command files
│   │   └── playbooks/              # Implementation playbooks
│   │       ├── .claudeignore       # Ignore playbooks by default
│   │       └── *-playbook.md       # Playbook files
│   ├── docs/                       # Claude-specific documentation
│   │   └── README-playbooks-and-slash-commands.md
│   └── agents/                     # Agent configurations (optional)
│
├── .github/                        # GitHub-specific configurations
│   ├── workflows/                  # GitHub Actions workflows
│   │   ├── ci.yml                  # Continuous integration
│   │   └── deploy.yml              # Deployment workflow (if applicable)
│   ├── ISSUE_TEMPLATE/             # Issue templates
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   ├── PULL_REQUEST_TEMPLATE.md    # PR template
│   └── CODEOWNERS                  # Code ownership (if applicable)
│
├── docs/                           # Project documentation
│   ├── README.md                   # Documentation index
│   ├── architecture.md             # Architecture documentation (if needed)
│   └── setup.md                    # Setup instructions
│
├── src/                            # Source code (language-specific)
├── tests/                          # Test files
├── scripts/                        # Utility scripts
│
├── README.md                       # Project README
├── LICENSE                         # License file
├── CHANGELOG.md                    # Change log
└── package.json                    # Or equivalent (requirements.txt, Cargo.toml, etc.)
```

---

## 5. PLAY: Initialize Repository

**Goal**: Create git repository and establish baseline structure.

**Steps**:

1. Navigate to project directory
2. Initialize git repository: `git init`
3. Rename default branch if needed: `git branch -M main`
4. Verify initialization: `git status`

**Validation**:
- [ ] `.git/` directory exists
- [ ] Branch is `main` (or configured default)
- [ ] No errors from git commands

---

## 6. PLAY: Create .gitignore

**Goal**: Establish comprehensive ignore rules from the start.

**Algorithm**:

1. Determine primary language/framework
2. Include language-specific ignore patterns
3. Add common IDE/editor patterns
4. Add OS-specific patterns
5. Include environment and secret files
6. Add build output and dependency directories

**Required Sections**:

```gitignore
# Environment variables (local only)
.env
.env.local
.env.*.local
*.env
!.env.example

# Dependencies
node_modules/
venv/
.venv/
__pycache__/

# Build outputs
dist/
build/
target/
*.log
_build/
!_build/README.md

# Private/local files
_private/
!_private/README.md

# Claude/Codex directories (completely ignored for local use)
.claude/*
.codex/*

# IDE / Editor
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store

# OS
Thumbs.db
.DS_Store

# Temporary files
temp/
tmp/
*.tmp

# Secrets (extra safety)
*.pem
*.key
*.cert
credentials.json
secrets.yaml
```

**Language-Specific Additions**:

- **Node.js**: `node_modules/`, `npm-debug.log`, `yarn-error.log`, `.npm`, `.yarn`
- **Python**: `__pycache__/`, `*.py[cod]`, `.pytest_cache/`, `.mypy_cache/`, `venv/`, `.venv/`
- **Rust**: `target/`, `Cargo.lock` (for binaries)
- **Go**: `vendor/`, `*.exe`, `*.test`
- **Java**: `*.class`, `target/`, `.gradle/`, `build/`

**Validation**:
- [ ] `.gitignore` exists in project root
- [ ] Language-specific patterns included
- [ ] Environment files ignored
- [ ] Secrets patterns included

---

## 7. PLAY: Create .editorconfig

**Goal**: Ensure consistent code formatting across editors.

**Template**:

```ini
# EditorConfig: https://EditorConfig.org

root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.{js,ts,jsx,tsx,json,yml,yaml,md}]
indent_style = space
indent_size = 2

[*.{py,rs,go}]
indent_style = space
indent_size = 4

[Makefile]
indent_style = tab

[*.md]
trim_trailing_whitespace = false
```

**Validation**:
- [ ] `.editorconfig` exists in project root
- [ ] Language-specific rules included
- [ ] Line endings set to LF

---

## 8. PLAY: Setup .claude Directory (OPTIONAL - Local Only)

**Goal**: Initialize Claude Code integration with proper structure.

**IMPORTANT**: The `.claude/` and `.codex/` directories are for **local development only** and are completely gitignored. This play is optional and only needed if you're using Claude Code or Codex locally.

**Steps**:

1. Create `.claude/` directory structure
2. Create `.claude/CLAUDE.md` with project instructions
3. Create `.claude/commands/` directory
4. Create `.claude/commands/playbooks/` directory
5. Create `.claude/commands/playbooks/.claudeignore` with content: `*`
6. Create `.claude/docs/` directory
7. Create master index file

**CLAUDE.md Template**:

```markdown
# CLAUDE.md - Project Instructions

> **Project**: <project-name>
> **Version**: 1.0
> **Last Updated**: <date>

---

## Project Overview

<One-paragraph description of the project>

---

## Tech Stack

- **Language**: <primary-language>
- **Framework**: <framework-if-applicable>
- **Runtime**: <version-requirements>
- **Package Manager**: <npm/pip/cargo/etc>

---

## Project Structure

<Brief description of key directories>

---

## Development Workflow

### Setup
```bash
# Installation steps
npm install  # or equivalent
```

### Running Locally
```bash
# Local development command
npm run dev  # or equivalent
```

### Testing
```bash
# Test command
npm test  # or equivalent
```

---

## Slash Commands and Playbooks

For available slash commands and playbooks, see:
[.claude/docs/README-playbooks-and-slash-commands.md](.claude/docs/README-playbooks-and-slash-commands.md)

---

## Code Standards

- Follow lowercase filename convention (kebab-case)
- Write tests for new features
- Update documentation for API changes
- Use meaningful commit messages

---

## Environment Variables

See `.env.example` for required environment variables.

Never commit `.env` files with secrets.

---

## Deployment

<Deployment process if applicable>

---

**Questions**: File issue or contact <maintainer>
```

**Index File Template** (`.claude/docs/README-playbooks-and-slash-commands.md`):

```markdown
# Slash Commands and Playbooks

## Overview

This directory contains slash commands and their implementation playbooks.

## Available Commands

(To be populated as commands are added)

---

## Pattern

All commands follow the Thin Command Surface Pattern:
- Command files (`.claude/commands/*.md`) are lightweight dispatch layers
- Playbooks (`.claude/commands/playbooks/*-playbook.md`) contain implementation details
- Playbooks are excluded from context via `.claudeignore`

---

## Adding New Commands

See [playbook-generator-playbook.md](../commands/playbooks/playbook-generator-playbook.md)
```

**Validation**:
- [ ] `.claude/` directory exists
- [ ] `.claude/CLAUDE.md` created and customized
- [ ] `.claude/commands/` directory exists
- [ ] `.claude/commands/playbooks/.claudeignore` contains `*`
- [ ] `.claude/docs/README-playbooks-and-slash-commands.md` created

---

## 9. PLAY: Setup .github Directory

**Goal**: Configure GitHub-specific files for issues, PRs, and workflows.

**Steps**:

1. Create `.github/` directory structure
2. Create issue templates
3. Create pull request template
4. Create basic CI workflow (if applicable)

**Bug Report Template** (`.github/ISSUE_TEMPLATE/bug_report.md`):

```markdown
---
name: Bug Report
about: Report a bug or unexpected behavior
title: '[BUG] '
labels: bug
assignees: ''
---

## Description

A clear description of the bug.

## Steps to Reproduce

1. Step 1
2. Step 2
3. ...

## Expected Behavior

What you expected to happen.

## Actual Behavior

What actually happened.

## Environment

- OS: [e.g., Ubuntu 22.04, macOS 14, Windows 11]
- Version: [e.g., v1.2.3]
- Runtime: [e.g., Node 20.x, Python 3.11]

## Additional Context

Any other relevant information.
```

**Feature Request Template** (`.github/ISSUE_TEMPLATE/feature_request.md`):

```markdown
---
name: Feature Request
about: Suggest a new feature or enhancement
title: '[FEATURE] '
labels: enhancement
assignees: ''
---

## Problem Statement

What problem does this feature solve?

## Proposed Solution

Describe your proposed solution.

## Alternatives Considered

Other approaches you've considered.

## Additional Context

Any other relevant information.
```

**Pull Request Template** (`.github/PULL_REQUEST_TEMPLATE.md`):

```markdown
## Description

Brief description of changes.

## Type of Change

- [ ] Bug fix
- [ ] New feature
- [ ] Breaking change
- [ ] Documentation update
- [ ] Refactoring

## Checklist

- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] Code follows project standards
- [ ] All tests passing
- [ ] No new warnings

## Related Issues

Closes #<issue-number>
```

**Basic CI Workflow** (`.github/workflows/ci.yml`) - **Node.js Example**:

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    steps:
    - uses: actions/checkout@v3

    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}

    - name: Install dependencies
      run: npm ci

    - name: Run tests
      run: npm test

    - name: Run linter
      run: npm run lint
```

**Validation**:
- [ ] `.github/` directory exists
- [ ] Issue templates created
- [ ] PR template created
- [ ] CI workflow created (if applicable)

---

## 10. PLAY: Create Essential Documentation

**Goal**: Establish baseline documentation structure.

**README.md Template**:

```markdown
# <Project Name>

<One-line description>

## Overview

<Brief overview of what this project does and why>

## Features

- Feature 1
- Feature 2
- Feature 3

## Installation

```bash
# Installation steps
npm install  # or equivalent
```

## Usage

```bash
# Basic usage example
npm start  # or equivalent
```

## Documentation

See [docs/](docs/) for detailed documentation.

## Development

### Prerequisites

- <Prerequisite 1>
- <Prerequisite 2>

### Setup

```bash
# Setup steps
git clone <repo-url>
cd <project-name>
npm install
```

### Running Tests

```bash
npm test
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) (if applicable)

## License

This project is licensed under the <LICENSE-TYPE> License - see [LICENSE](LICENSE) file.

## Contact

<Maintainer contact info>
```

**CHANGELOG.md Template**:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial project setup

## [0.1.0] - <date>

### Added
- Project structure
- Basic documentation
- Development environment
```

**.env.example Template**:

```bash
# Environment Variables Template
# Copy to .env and fill in actual values

# Application
APP_NAME=<project-name>
APP_ENV=development
APP_PORT=3000

# Database (if applicable)
# DATABASE_URL=

# API Keys (if applicable)
# API_KEY=

# Secrets (never commit actual values)
# SECRET_KEY=
```

**Validation**:
- [ ] `README.md` created and customized
- [ ] `CHANGELOG.md` created
- [ ] `.env.example` created
- [ ] `docs/` directory created

---

## 11. PLAY: Scan for Secrets and Generate .env

**Goal**: Scan codebase for hardcoded secrets and generate `.env` file with detected variables.

**Algorithm**:

1. Scan all source files for potential secrets
2. Identify environment variable patterns
3. Extract unique variable names
4. Generate `.env.example` with placeholders
5. Warn about any hardcoded secrets found

**Secret Detection Patterns**:

```regex
# API Keys
(api[_-]?key|apikey|access[_-]?key)\s*[=:]\s*['\"]?([a-zA-Z0-9_-]{20,})['\"]?

# Tokens
(token|auth[_-]?token|bearer[_-]?token)\s*[=:]\s*['\"]?([a-zA-Z0-9_-]{20,})['\"]?

# Passwords
(password|passwd|pwd)\s*[=:]\s*['\"]?([^\s'\";]{8,})['\"]?

# Database URLs
(database[_-]?url|db[_-]?url|connection[_-]?string)\s*[=:]\s*['\"]?([^\s'\";]+)['\"]?

# AWS Credentials
(aws[_-]?access[_-]?key|aws[_-]?secret)\s*[=:]\s*['\"]?([A-Z0-9]{20,})['\"]?

# Private Keys
-----BEGIN (RSA |EC |OPENSSH )?PRIVATE KEY-----

# Environment Variable References
process\.env\.([A-Z_][A-Z0-9_]*)
os\.getenv\(['\"]([A-Z_][A-Z0-9_]*)['\"]
ENV\[['"]([A-Z_][A-Z0-9_]*)['\"]
\$\{?([A-Z_][A-Z0-9_]*)\}?
```

**Steps**:

1. **Scan Source Files**:
   ```bash
   # Find all source files
   find src/ -type f \( -name "*.js" -o -name "*.ts" -o -name "*.py" -o -name "*.rs" -o -name "*.go" \) 2>/dev/null
   ```

2. **Detect Hardcoded Secrets**:
   - Search for patterns above
   - Flag any matches as security issues
   - Create list of violations

3. **Extract Environment Variables**:
   - Find `process.env.VAR_NAME` patterns
   - Find `os.getenv('VAR_NAME')` patterns
   - Find `ENV['VAR_NAME']` patterns
   - Find `${VAR_NAME}` patterns
   - Collect unique variable names

4. **Generate .env.example**:
   ```bash
   # Example generated content
   # Environment Variables (detected from codebase)
   # Copy to .env and fill in actual values

   # Application
   APP_NAME=<project-name>
   APP_ENV=development
   APP_PORT=3000

   # Detected Variables
   DATABASE_URL=
   API_KEY=
   JWT_SECRET=
   REDIS_URL=
   ```

5. **Create .gitignore Entries**:
   Ensure `.env` is in `.gitignore` but `.env.example` is not.

**Security Report Output**:

```markdown
## Secret Scan Results

### ⚠️ Hardcoded Secrets Found: X

| File | Line | Type | Severity |
|------|------|------|----------|
| src/config.js | 12 | API Key | HIGH |
| src/auth.js | 45 | Password | HIGH |

**Action Required**: Move these to environment variables immediately.

### ✅ Environment Variables Detected: Y

- DATABASE_URL
- API_KEY
- JWT_SECRET
- REDIS_URL
- AWS_ACCESS_KEY_ID
- AWS_SECRET_ACCESS_KEY

**Files Created**:
- `.env.example` (template with all detected variables)
- `.env` (local file, gitignored)
```

**Validation**:
- [ ] Source files scanned
- [ ] Hardcoded secrets identified (if any)
- [ ] Environment variables extracted
- [ ] `.env.example` generated
- [ ] `.env` added to `.gitignore`
- [ ] Security warnings issued for hardcoded secrets

---

## 12. PLAY: Identify and Document Dependencies

**Goal**: Scan codebase for dependencies and create/update package manager files.

**Algorithm**:

1. Detect project type (Node.js, Python, Rust, Go, etc.)
2. Scan import statements
3. Identify external dependencies
4. Generate/update package manager file
5. Document version constraints

**Detection Strategies**:

**Node.js/TypeScript**:
```bash
# Scan for imports
grep -rh "^\s*import\s.*from\s*['\"]" src/ --include="*.js" --include="*.ts" | \
  sed -E "s/.*from\s*['\"]([^'\"]+)['\"].*/\1/" | \
  grep -v "^\." | sort -u

# Scan for requires
grep -rh "require\s*(['\"]" src/ --include="*.js" | \
  sed -E "s/.*require\s*\(['\"]([^'\"]+)['\"].*/\1/" | \
  grep -v "^\." | sort -u
```

**Python**:
```bash
# Scan for imports
grep -rh "^\s*import\s\|^\s*from\s.*import" src/ --include="*.py" | \
  sed -E 's/^(from|import)\s+([a-zA-Z0-9_]+).*/\2/' | \
  sort -u
```

**Rust**:
```bash
# Scan Cargo.toml if exists, or scan for use statements
grep -rh "^\s*use\s" src/ --include="*.rs" | \
  sed -E 's/use\s+([a-zA-Z0-9_]+)::.*/\1/' | \
  grep -v "^(std|crate|self|super)" | \
  sort -u
```

**Go**:
```bash
# Scan for imports
grep -rh "^\s*import\s" src/ --include="*.go" | \
  sed -E 's/.*"([^"]+)".*/\1/' | \
  grep "/" | \
  sort -u
```

**Output**:

```markdown
## Dependency Analysis

### Detected Language: <language>

### External Dependencies Found: X

**Production Dependencies**:
- package-name-1 (detected in: file1.js, file2.js)
- package-name-2 (detected in: file3.js)

**Development Dependencies** (inferred):
- jest (test files)
- eslint (config file detected)

### Package Manager File Status

- [x] File exists: package.json
- [x] Dependencies documented: Yes
- [ ] Lock file exists: package-lock.json

### Recommended Actions

1. Run `npm install <package>` for each detected dependency
2. Verify versions are appropriate
3. Update package.json with version constraints
4. Generate lock file: `npm install`
```

**Package Manager File Generation**:

**Node.js (package.json)**:
```json
{
  "name": "<project-name>",
  "version": "0.1.0",
  "description": "<project-description>",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "test": "jest",
    "lint": "eslint src/",
    "format": "prettier --write \"src/**/*.{js,ts,jsx,tsx}\""
  },
  "dependencies": {
    "<detected-dep-1>": "^x.y.z",
    "<detected-dep-2>": "^x.y.z"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "eslint": "^8.0.0",
    "nodemon": "^3.0.0"
  },
  "keywords": [],
  "author": "<author>",
  "license": "<license-type>"
}
```

**Python (requirements.txt)**:
```txt
# Production dependencies (detected from imports)
<detected-package-1>>=x.y.z
<detected-package-2>>=x.y.z

# Development dependencies
pytest>=7.0.0
black>=23.0.0
pylint>=2.0.0
```

**Validation**:
- [ ] Project language detected
- [ ] Import statements scanned
- [ ] Dependencies identified
- [ ] Package manager file created/updated
- [ ] Development dependencies inferred
- [ ] Lock file generation recommended

---

## 13. PLAY: Create Build and Private Directories

**Goal**: Create standard directories for build artifacts and private/local files.

**Directory Purpose**:

- `_build/` - Build outputs, generated artifacts, reports, compiled assets
- `_private/` - Local-only files, secrets, personal notes, temporary work

**Algorithm**:

1. Create `_build/` directory in project root
2. Create `_private/` directory in project root
3. Add both to `.gitignore`
4. Create README files in each explaining purpose

**Steps**:

```bash
# Create directories
mkdir -p _build
mkdir -p _private

# Create README files
echo "# Build Artifacts

This directory contains generated build outputs and artifacts.

**Contents (examples)**:
- Compiled code
- Build reports
- Generated documentation
- Test coverage reports
- Performance benchmarks

**Important**: All contents are gitignored. Do not commit build artifacts.
" > _build/README.md

echo "# Private Files

This directory contains local-only files that should never be committed.

**Contents (examples)**:
- Local development notes
- Temporary scripts
- Personal configuration
- Local credentials (use .env instead when possible)
- Work-in-progress files

**Important**: All contents are gitignored.
" > _private/README.md
```

**Update .gitignore**:

Add these entries to `.gitignore`:

```gitignore
# Build artifacts
_build/
!_build/README.md

# Private/local files
_private/
!_private/README.md

# Alternative: if you want to ignore everything including README
# _build/
# _private/
```

**Directory Structure Result**:

```
<project-root>/
├── _build/
│   └── README.md (tracked)
├── _private/
│   └── README.md (tracked)
├── .gitignore (updated)
└── ...
```

**Validation**:
- [ ] `_build/` directory created
- [ ] `_private/` directory created
- [ ] README.md created in both directories
- [ ] Both directories added to `.gitignore`
- [ ] README files optionally tracked (pattern: `!_build/README.md`)

---

## 14. PLAY: Initialize Package Manager

**Goal**: Create package manager configuration file (after dependencies are identified).

**Node.js (package.json)**:

```json
{
  "name": "<project-name>",
  "version": "0.1.0",
  "description": "<project-description>",
  "main": "src/index.js",
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "test": "jest",
    "lint": "eslint src/",
    "format": "prettier --write \"src/**/*.{js,ts,jsx,tsx}\"",
    "build": "# Add build command",
    "clean": "rm -rf _build/*"
  },
  "keywords": [],
  "author": "<author>",
  "license": "<license-type>",
  "devDependencies": {},
  "dependencies": {}
}
```

**Python (requirements.txt + setup.py)**:

```python
# requirements.txt
# Add dependencies here

# setup.py
from setuptools import setup, find_packages

setup(
    name="<project-name>",
    version="0.1.0",
    description="<project-description>",
    author="<author>",
    packages=find_packages(),
    install_requires=[],
    python_requires=">=3.8",
)
```

**Rust (Cargo.toml)**:

```toml
[package]
name = "<project-name>"
version = "0.1.0"
edition = "2021"
authors = ["<author>"]
description = "<project-description>"

[dependencies]
```

**Validation**:
- [ ] Package manager file created
- [ ] Basic scripts/tasks defined
- [ ] Build and clean scripts reference `_build/`
- [ ] Version set to 0.1.0

---

## 15. PLAY: Create Initial Commit

**Goal**: Commit initial project structure to git.

**Algorithm**:

1. Stage all created files
2. Create initial commit with descriptive message
3. Optionally create develop branch
4. Optionally add remote and push

**Steps**:

```bash
# Stage all files
git add .

# Create initial commit
git commit -m "feat: initial project setup

- Initialize git repository
- Add .gitignore and .editorconfig
- Setup .claude/ directory for Claude Code integration
- Configure .github/ for workflows and templates
- Create essential documentation (README, CHANGELOG)
- Initialize package manager configuration

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"

# Optional: Create develop branch
git checkout -b develop

# Optional: Add remote and push
# git remote add origin <repo-url>
# git push -u origin main
# git push -u origin develop
```

**Validation**:
- [ ] All files committed
- [ ] Commit message follows conventional commits format
- [ ] Remote added (if applicable)
- [ ] Pushed to GitHub (if applicable)

---

## 16. Invariants (MUST / NEVER)

**MUST**:
- Use lowercase filenames (kebab-case)
- Initialize git before creating files
- Create `.gitignore` before committing
- Include `.env.example`, never commit `.env`
- Setup `.claude/` directory structure
- Create `.claudeignore` in playbooks directory
- Create `_build/` and `_private/` directories
- Add `_build/` and `_private/` to `.gitignore`
- Scan for hardcoded secrets
- Identify and document dependencies
- Follow directory structure pattern

**NEVER**:
- Commit secrets or credentials
- Commit contents of `_build/` (except README.md)
- Commit contents of `_private/` (except README.md)
- Use UPPERCASE filenames (except pre-existing conventions like LICENSE)
- Create files in root without purpose
- Skip `.gitignore` creation
- Initialize without documentation
- Leave hardcoded secrets in code

---

## 17. Error Handling

**Missing Prerequisites**:
- If project name not decided → STOP, ask user
- If language/framework unknown → STOP, ask user
- If git not installed → FAIL with installation instructions

**File Conflicts**:
- If directory already exists → Ask user to confirm overwrite or merge
- If `.git/` already exists → Ask user if re-initialization is intended

**Permission Errors**:
- If cannot create directories → Check filesystem permissions
- If cannot initialize git → Check git installation and PATH

---

## 18. Testing Checklist

- [ ] Git repository initialized successfully
- [ ] `.gitignore` exists and covers common patterns
- [ ] `.editorconfig` exists
- [ ] `_build/` directory created with README.md
- [ ] `_private/` directory created with README.md
- [ ] Both `_build/` and `_private/` in `.gitignore`
- [ ] Secret scan completed
- [ ] No hardcoded secrets found (or moved to .env)
- [ ] Dependencies identified and documented
- [ ] `.env.example` generated from detected variables
- [ ] `.claude/` directory structure complete
- [ ] `.github/` directory structure complete
- [ ] `README.md` created and contains project info
- [ ] `CHANGELOG.md` created
- [ ] Package manager file created with dependencies
- [ ] Initial commit created
- [ ] No secrets committed
- [ ] All files use lowercase naming (except allowed exceptions)

---

## 19. Performance Considerations

**Typical Setup Time**: 2-5 minutes for basic setup

**Scaling**:
- Simple projects: Use minimal structure
- Complex projects: Add additional directories as needed
- Monorepos: Consider additional tooling (lerna, nx, turborepo)

---

## 20. Explicit Non-Goals

- This playbook does NOT set up specific frameworks (React, Django, etc.)
- This playbook does NOT configure deployment pipelines beyond basic CI
- This playbook does NOT create language-specific linting configs (ESLint, Pylint, etc.)
- This playbook does NOT set up Docker or containerization
- This playbook does NOT configure databases or infrastructure
- This playbook does NOT automatically install dependencies (only identifies them)
- This playbook does NOT fix hardcoded secrets automatically (only detects and warns)

Those should be handled by framework-specific setup playbooks or manual intervention.

---

## 21. Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-02-08 | Initial playbook creation for standardized GitHub project setup with secret scanning, dependency detection, and `_build/`/`_private/` directory creation |

---

**Maintained by**: Project maintainers
**Questions**: File issue or contact maintainer
**Status**: Production standard
