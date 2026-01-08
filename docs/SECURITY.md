# Security Analysis: get-shit-done

**Analysis Date:** 2026-01-06
**Version Analyzed:** 1.3.26
**Analyst:** Claude Code (automated analysis)

## Executive Summary

**Verdict: SAFE** - This package is not malware. It is a legitimate meta-prompting system for Claude Code consisting primarily of markdown instruction files and a simple file-copying installer.

---

## What This Package Actually Does

GSD (Get Shit Done) is a **prompt engineering framework** for Claude Code. It provides:

1. **Slash commands** (`.md` files) that Claude Code interprets as instructions
2. **Templates** for project planning documents
3. **Workflows** that guide Claude through multi-step operations

When you run `npx get-shit-done-cc`, it copies markdown files to your Claude Code configuration directory. That's it.

---

## Code Inventory

### Executable Code

| File | Purpose | Risk |
|------|---------|------|
| `bin/install.js` | Copies files to `~/.claude/` or `./.claude/` | **None** - uses only Node.js built-ins |

**Total executable code:** 1 JavaScript file (154 lines)

### Non-Executable Content

| Directory | Contents | Count |
|-----------|----------|-------|
| `commands/gsd/` | Slash command definitions | 19 `.md` files |
| `get-shit-done/references/` | Guideline documents | 9 `.md` files |
| `get-shit-done/templates/` | Document templates | 18 `.md` files |
| `get-shit-done/workflows/` | Multi-step procedures | 14 `.md` files |

---

## Installer Analysis (`bin/install.js`)

### Dependencies

```javascript
const fs = require('fs');        // Built-in: file system
const path = require('path');    // Built-in: path manipulation
const os = require('os');        // Built-in: OS info (home directory)
const readline = require('readline'); // Built-in: user input
```

**Zero external dependencies.** Only uses Node.js standard library modules.

### What It Does

1. Displays an ASCII banner
2. Asks where to install (global `~/.claude/` or local `./.claude/`)
3. Copies `commands/gsd/` and `get-shit-done/` directories to chosen location
4. Replaces path references in `.md` files (`~/.claude/` → appropriate prefix)

### What It Does NOT Do

- No network requests (no `http`, `https`, `fetch`, `axios`)
- No code execution (`eval`, `exec`, `spawn`, `child_process`)
- No data exfiltration
- No postinstall scripts in `package.json`
- No obfuscated code
- No minified code
- No encoded strings (base64, etc.)
- No credential access
- No system modification outside target directory

### Environment Variables

Only reads `CLAUDE_CONFIG_DIR` - a documented Claude Code configuration variable for customizing the config directory location. This is legitimate and expected.

---

## Markdown Command Analysis

The `.md` files contain **instructions for Claude Code**, not executable code. However, they do instruct Claude to run bash commands. Analysis of these commands:

### Commands Claude Is Instructed to Run

| Type | Examples | Risk |
|------|----------|------|
| Git operations | `git init`, `git add`, `git commit`, `git status` | **Safe** - standard version control |
| File checks | `[ -f .planning/PROJECT.md ]`, `ls`, `cat` | **Safe** - read-only checks |
| Directory creation | `mkdir -p .planning/` | **Safe** - creates project folders |
| Date/time | `date -u +"%Y-%m-%dT%H:%M:%SZ"` | **Safe** - timestamp generation |

### Commands NOT Present

- No `curl`, `wget`, or network fetching
- No `rm -rf` or destructive deletions
- No `sudo` or privilege escalation
- No `chmod`, `chown`, or permission changes
- No `ssh`, `scp`, or remote access
- No package installation commands
- No system service manipulation

### Scope of Operations

All instructed operations are confined to:
- `.planning/` directory (project planning artifacts)
- `.git/` directory (version control)
- Files the user explicitly asks Claude to modify

---

## package.json Analysis

```json
{
  "name": "get-shit-done-cc",
  "version": "1.3.26",
  "bin": { "get-shit-done-cc": "bin/install.js" },
  "files": [ "bin", "commands", "get-shit-done" ],
  "engines": { "node": ">=16.7.0" }
}
```

**Notable absences:**
- No `dependencies` or `devDependencies`
- No `scripts` section (no postinstall hooks)
- No lifecycle scripts that auto-execute

---

## Trust Indicators

### Positive Signs

1. **Open source** - Full source visible on GitHub (`github.com/glittercowboy/get-shit-done`)
2. **Zero dependencies** - Nothing to hide in dependency tree
3. **No obfuscation** - All code is readable
4. **No network access** - Completely offline operation
5. **Minimal executable code** - 154 lines of JavaScript
6. **Clear purpose** - Files do exactly what documentation claims
7. **MIT licensed** - Standard open source license

### Author Verification

- npm package: `get-shit-done-cc`
- GitHub: `github.com/glittercowboy/get-shit-done`
- Author: TÂCHES

You can verify the npm package matches this repository by comparing contents.

---

## Potential Concerns (Low Risk)

### 1. Claude Code Will Execute Commands

The markdown files instruct Claude to run bash commands. This is the **intended purpose** of Claude Code. However:

- Commands are visible in the `.md` files before execution
- Claude Code has its own permission system
- User can review what Claude plans to do

**Mitigation:** Read the command files if concerned. They're plain text.

### 2. Files Written to ~/.claude/

Global install writes to your Claude Code config directory. This is:

- The documented location for Claude Code customizations
- Standard practice for Claude Code extensions
- Sandboxed to that directory only

**Mitigation:** Use `--local` flag to install to current project only.

### 3. Git Commits Are Automated

The workflows instruct Claude to make git commits automatically. This is:

- Documented behavior
- Reversible (`git reset`, `git revert`)
- Only affects the current repository

**Mitigation:** Review commits before pushing. Use interactive mode.

---

## How to Verify Yourself

```bash
# 1. Check what gets installed (before running)
npx get-shit-done-cc --help  # Won't install, just shows help

# 2. Download and inspect without installing
git clone https://github.com/glittercowboy/get-shit-done.git
cat get-shit-done/bin/install.js
grep -r "require(" get-shit-done/

# 3. Check for network calls
grep -rE "http|fetch|axios|request" get-shit-done/

# 4. Check for dangerous commands
grep -rE "rm -rf|sudo|eval|exec" get-shit-done/

# 5. Verify npm package matches GitHub
npm pack get-shit-done-cc --dry-run
```

---

## Conclusion

This package is **safe to use**. It is a well-designed prompt engineering system with:

- No malicious behavior
- No hidden functionality
- No external dependencies
- No network access
- Transparent, readable code

The author appears to be a legitimate developer creating tools for the Claude Code ecosystem. The package does exactly what it claims: provides structured workflows for project planning with Claude Code.

**Recommendation:** Safe to install and use. For maximum caution, use `--local` flag to keep files in current project only.
