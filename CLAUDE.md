# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Superpowers is a software development workflow system for coding agents, built on composable "skills" that trigger automatically based on task context. It's not a traditional software project—it's a collection of process documentation, testing infrastructure, and integration hooks for AI coding agents.

## Architecture

### Skills System

The core is a flat namespace of skills in `skills/`. Each skill is a self-contained markdown file with YAML frontmatter:

```markdown
---
name: skill-name
description: Use when [specific triggering conditions]
---

# Skill Name

[Content follows TDD-style documentation patterns]
```

**Critical principle:** Skills are discovered via semantic search. The `description` field MUST describe triggering conditions/symptoms, NOT the skill's workflow or process.

**Skill types:**
- **Discipline-enforcing:** TDD, verification-before-completion (require following rules)
- **Technique:** condition-based-waiting, root-cause-tracing (how-to guides)
- **Pattern:** Mental models for approaching problems
- **Reference:** API docs, command references

### Platform Integration

**Claude Code:** Plugin marketplace installs skills as a plugin. A session-start hook injects the `using-superpowers` skill content.

**Codex:** Native skill discovery via symlink `~/.agents/skills/superpowers` → `~/.codex/superpowers/skills`

**OpenCode:** Plugin (`superpowers.js`) injects bootstrap content via system prompt transform. Skills discovered via symlink to `~/.config/opencode/skills/superpowers`

### Testing Infrastructure

**Integration tests:** `tests/claude-code/` runs real Claude Code sessions in headless mode and parses `.jsonl` session transcripts to verify behavior.

**Token analysis:** `analyze-token-usage.py` breaks down usage by main session and subagents from session transcripts.

**OpenCode tests:** `tests/opencode/` verifies plugin loading, skill priority, and tool mapping.

## Key Commands

### Running Tests

```bash
# Integration test for subagent-driven-development (10-30 min)
cd tests/claude-code
./test-subagent-driven-development-integration.sh

# Analyze token usage from a session
python3 tests/claude-code/analyze-token-usage.py ~/.claude/projects/<project-dir>/<session-id>.jsonl
```

### Finding Sessions

Session transcripts are in `~/.claude/projects/` with working directory path encoded (slashes become hyphens):

```bash
# Find recent sessions
find ~/.claude/projects -name "*.jsonl" -mmin -60
```

## Core Workflow

The superpowers workflow agents follow:

1. **brainstorming** - Refines rough ideas through questions, validates design in chunks
2. **using-git-worktrees** - Creates isolated workspace on new branch
3. **writing-plans** - Breaks work into 2-5 minute tasks with exact file paths
4. **subagent-driven-development** OR **executing-plans** - Executes with review checkpoints
5. **test-driven-development** - Enforces RED-GREEN-REFACTOR cycle
6. **requesting-code-review** - Pre-review between tasks
7. **finishing-a-development-branch** - Merge/PR decision workflow

## Skill Creation Rules

**The Iron Law:** No skill without a failing test first. This applies to NEW skills AND EDITS to existing skills.

Skills follow TDD methodology applied to documentation:
1. **RED:** Run pressure scenarios WITHOUT skill → document baseline violations
2. **GREEN:** Write skill addressing those specific violations
3. **REFACTOR:** Close loopholes found during testing

See `skills/writing-skills/SKILL.md` for complete methodology.

## Important Files

- `skills/using-superpowers/SKILL.md` - Entry point: how agents find and use skills
- `skills/subagent-driven-development/SKILL.md` - Core execution workflow with two-stage review
- `skills/writing-skills/SKILL.md` - Complete guide to creating and testing skills
- `hooks/session-start.sh` - Injects bootstrap context for Claude Code
- `.opencode/plugins/superpowers.js` - OpenCode integration
- `agents/code-reviewer.md` - Template for code review subagents

## Platform-Specific Notes

**Codex:** Tool mappings differ - subagent dispatch uses `@mention` syntax, not `Task` tool. `TodoWrite` becomes `update_plan`.

**OpenCode:** Uses native `skill` tool for discovery. Plugin handles bootstrap injection via system prompt transform (fixes agent reset bug).
