---
name: session-reflection-analysis
description: Use when asked to reflect on how the session went
---

# Session Reflection Analysis

Analyze recent chat history to identify improvement opportunities and reduce token waste in future sessions.

## Overview

This skill helps identify patterns of inefficiency in Claude Code sessions by analyzing session history. The analysis focuses on actionable improvements to documentation, automation, and workflows.

## Step 1: Locate Project Sessions

First, find the correct project folder. Claude Code stores sessions at `%USERPROFILE%.claude\projects\` with Folder names are derived from the project path:
- slashes → dashes  
- colons → double dashes  

Example:

C:\Users\nguyen.bui\workspace\project
→

Users-nguyen.bui-workspace-project



---

### List available projects

```cmd
dir "%USERPROFILE%\.claude\projects\"

REM Convert current path to project folder name
set CURRENT_PATH=%cd%
set CURRENT_PATH=%CURRENT_PATH:\=-%
set CURRENT_PATH=%CURRENT_PATH::=--%

set PROJECT_DIR=%USERPROFILE%\.claude\projects\%CURRENT_PATH%

if exist "%PROJECT_DIR%" (
    echo Found project dir: %PROJECT_DIR%
    dir "%PROJECT_DIR%\*.jsonl"
) else (
    echo No session folder found for current project.
    echo Available projects:
    dir "%USERPROFILE%\.claude\projects\"
)
```

If detection fails, copy the correct path manually.

## Step 2: Generate Session Summary

**CRITICAL**: Do NOT read raw session files directly. They are massive and will consume your entire token budget.

First, verify `jq` is available:

```cmd
where jq
if %errorlevel% neq 0 (
    echo ERROR: jq is required but not installed.
    echo Download from: https://jqlang.github.io/jq/download/
    exit /b 1
)
```

Then use jq to generate a 98% token-reduced summary:

```cmd
REM SET THIS if auto-detect failed
REM set PROJECT_DIR=%USERPROFILE%\.claude\projects\--Users-harper-src-myproject

set OUTPUT=%TEMP%\session-summary.jsonl

type "%PROJECT_DIR%\*.jsonl" | jq -c ^
"select(.type == \"user\" or .type == \"assistant\") | {
  type,
  ts: .timestamp,
  content: (
    if .message.content | type == \"string\" then
      .message.content[0:300]
    elif .message.content | type == \"array\" then
      [.message.content[] |
        if .type == \"text\" then {t: \"text\", v: .text[0:300]}
        elif .type == \"tool_use\" then {t: \"tool\", v: .name}
        elif .type == \"tool_result\" then {t: \"result\", len: (.content | length)}
        elif .type == \"thinking\" then empty
        else {t: .type}
        end
      ]
    else null
    end
  )
}" > "%OUTPUT%"

echo Summary created at: %OUTPUT%
```

This extracts:
- User message text (first 300 chars)
- Tool calls (name only, not full output)
- Tool result sizes (not content)
- Timestamps for sequence analysis

## Step 3: Launch Subagent for Analysis

Use the Task tool to spawn an analysis subagent:

```
Task tool parameters:
- subagent_type: "Explore"
- prompt: |
    Analyze the session summary at /tmp/session-summary.jsonl for inefficiency patterns.

    Read the file and look for these patterns:

    | Pattern | Example | Impact |
    |---------|---------|--------|
    | **Wasted tokens** | Re-reading same file 5+ times | High |
    | **Wrong paths taken** | Implemented feature, then discovered existing code | Medium-High |
    | **Repeated mistakes** | Same error type in 3+ instances | Medium |
    | **Missing automation** | Manual steps repeated across sessions | Medium |
    | **Missing documentation** | Had to figure out what should be in CLAUDE.md | Medium |
    | **Unnecessary tool calls** | Called multiple tools when one would work | Low-Medium |
    | **Context loss after compaction** | Info that should survive in persistent docs | High |
    | **Assumption without verification** | Decisions made without checking existing code | High |

    For each pattern found, propose improvements as:
    1. CLAUDE.md updates
    2. New skills for .claude/skills/
    3. New slash commands for .claude/commands/
    4. Script automation
    5. Git hooks
    6. Workflow changes

    Make all proposals copy-paste ready with complete text/code.
    Quantify impact where possible (tokens saved, time saved).
    Be specific with examples from the actual session data.
```

## Step 4: Create Reflection Document

The subagent should generate a document with this structure:

```markdown
# Session Reflection: YYYY-MM-DD

## Summary Statistics
- Session date range: [start] to [end]
- Messages analyzed: [count]
- Major inefficiencies found: [count]

## Proposed Improvements

### High Priority

#### 1. [Problem Title]

**Problem observed**: [What went wrong - be specific with examples]

**Proposed solution**:
```
[Complete text/code to add or change - copy-paste ready]
```

**File to modify**: `path/to/file.md`

**Cost**: [Effort to implement, any downsides]

**Benefit**: [Time/tokens saved - quantify if possible]

---

### Medium Priority
...

### Low Priority
...

## Implementation Notes

[Cross-cutting concerns, dependencies, warnings]
```

## Step 5: Present Findings

After analysis completes:

1. **Summarize key findings** - 3-5 bullet points of major inefficiencies
2. **Show the reflection document path** - Where to find full details
3. **Ask which changes to implement** - Don't auto-implement

## Critical Rules

1. **NEVER read raw session files directly** - Always use the jq summary
2. **NEVER implement changes automatically** - Only propose them
3. **Make proposals copy-paste ready** - Complete text, not descriptions
4. **Quantify impact when possible** - "Saved 10K tokens" not "saves tokens"
5. **Be specific with examples** - "Re-read config.py 7 times" not "read files multiple times"

## Example Analysis Output

```
High Priority Issues Found:

1. **Re-reading the same file repeatedly** (7 times in one session)
   - Propose: Add key architecture summaries to CLAUDE.md
   - Benefit: Save ~15K tokens per session

2. **Implemented feature without checking existing code first**
   - Took 3 iterations to match actual behavior
   - Propose: Add "Check existing code FIRST" rule to CLAUDE.md
   - Benefit: Prevent wrong-path implementations

3. **Manual repetitive commands**
   - Same shell commands typed each session
   - Propose: Create slash commands for common operations
   - Benefit: Save 2-3 minutes per session, prevent typos
```

## Troubleshooting

### Can't find project folder
```bash
# List all projects with recent activity
ls -lt ~/.claude/projects/ | head -10

# Search for a keyword in project names
ls ~/.claude/projects/ | grep -i "keyword"
```

### No jsonl files found
Sessions may not have been saved yet, or the project path detection failed. Check the folder manually.

### jq not installed
```bash
# macOS
brew install jq

# Ubuntu/Debian
sudo apt install jq

# Or use the standalone binary
curl -Lo /usr/local/bin/jq https://github.com/jqlang/jq/releases/download/jq-1.7.1/jq-macos-arm64
chmod +x /usr/local/bin/jq
```
