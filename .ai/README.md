# AI Repository Memory System

This directory contains persistent architectural memory for the AI Trading Agent project.

## Purpose

The `.ai/` directory serves as a structured knowledge base that AI coding assistants use to maintain context, understand architecture, and make informed decisions when generating code or suggestions.

## Files

### 1. `session-context.md` - Current Session Memory
**What it tracks:** Active development session
- Current objectives
- Tasks in progress
- Files being modified
- Decisions made during this session
- Important observations

**Update frequency:** Throughout active development session
**Usage:** Reference before making changes; update after significant steps

---

### 2. `repo-architecture.md` - System Architecture
**What it tracks:** High-level system design
- Core components
- Data flow diagrams
- Key interfaces
- External integrations
- Technology stack
- Architectural principles

**Update frequency:** When architecture changes
**Usage:** Reference before implementing new features; ensure consistency with existing design

---

### 3. `git-history-context.md` - Git Intelligence
**What it tracks:** Commit history analysis
- Recent commits with inferred meaning
- Impact analysis of changes
- Evolution patterns

**Update frequency:** Periodically or before major work
**Usage:** Understand recent changes; identify patterns; avoid regressions

**Note:** Currently a placeholder as git is not yet initialized

---

### 4. `design-decisions.md` - Architectural Decision Records (ADR)
**What it tracks:** Major architectural choices
- Context: What problem existed
- Decision: What solution was chosen
- Reasoning: Why this approach
- Consequences: Tradeoffs and implications

**Update frequency:** When making significant architectural decisions
**Usage:** Understand why things are built a certain way; maintain consistency

---

### 5. `task-tracker.md` - Development Tasks
**What it tracks:** Project task management
- Tasks in progress
- Planned work
- Completed tasks
- Blocked items

**Update frequency:** As tasks change status
**Usage:** Track progress; prioritize work; see what's done

---

### 6. `code-map.md` - Codebase Navigation
**What it tracks:** Directory structure and responsibilities
- Purpose of each directory
- Key files and their roles
- Patterns and conventions
- Navigation tips

**Update frequency:** When structure changes significantly
**Usage:** Find where things belong; understand organization

---

## Workflow for AI Assistants

### Before Generating Code or Making Changes

1. **Load context files in this order:**
   ```
   .ai/session-context.md       (what's happening now)
   .ai/repo-architecture.md     (how things work)
   .ai/git-history-context.md   (what changed recently)
   .ai/design-decisions.md      (why things are this way)
   .ai/code-map.md              (where things are)
   .ai/task-tracker.md          (what needs to be done)
   ```

2. **Verify understanding:**
   - Does the proposed change align with architecture?
   - Are there existing patterns to follow?
   - What recent changes might affect this?
   - What decisions have been made about this area?

3. **Proceed with implementation**
   - Respect existing patterns
   - Maintain architectural consistency
   - Reference decisions when relevant

### After Making Changes

1. **Update session-context.md**
   - Add to "Files Modified"
   - Document decisions taken
   - Note any important observations

2. **Update other files if needed:**
   - `repo-architecture.md` - if architecture changed
   - `design-decisions.md` - if architectural decision made
   - `task-tracker.md` - if task status changed
   - `code-map.md` - if structure changed

### Periodically

1. **Update git-history-context.md**
   - Run: `git log master --pretty=format:"%h %ad %s" --date=short -n 100`
   - Analyze recent commits
   - Document impact and patterns

2. **Review and clean up**
   - Archive old session contexts
   - Remove outdated decisions
   - Update architecture docs

---

## Maintenance Guidelines

### Do's ✅
- Keep files up to date
- Be concise and clear
- Focus on "why" not just "what"
- Reference these files when making decisions
- Update after every significant change

### Don'ts ❌
- Don't let files become stale
- Don't document trivial changes
- Don't contradict existing decisions without updating them
- Don't ignore the memory system

---

## Integration with Development

This system is designed to work alongside:
- `.github/copilot-instructions.md` - Coding style and conventions
- Project README - User-facing documentation
- Code comments - Implementation details
- Commit messages - Change history

Each serves a different purpose. The `.ai/` directory focuses on architectural context and decision tracking.

---

## Benefits

1. **Consistency** - New code follows established patterns
2. **Context Preservation** - Understanding survives across sessions
3. **Decision Tracking** - Know why things are built a certain way
4. **Onboarding** - New AI sessions start with full context
5. **Architecture Enforcement** - Changes align with design
6. **Reduced Errors** - Avoid conflicting with existing decisions

---

## Example Usage

### Scenario: Adding a New Data Source

**Step 1 - Before coding:**
```
Read .ai/repo-architecture.md → Understand data source patterns
Read .ai/code-map.md → Find where data sources live
Read .ai/design-decisions.md → Check async patterns, error handling
```

**Step 2 - During coding:**
```
Follow established patterns from architecture
Use async/await per architectural principles
Implement in data_sources/ per code-map
```

**Step 3 - After coding:**
```
Update .ai/session-context.md → Add files modified
Update .ai/task-tracker.md → Mark task complete
If significant: Update .ai/design-decisions.md
```

---

## Version Control

- ✅ **DO commit** `.ai/` directory to version control
- ✅ All files should be tracked in git
- ✅ Changes to architecture docs are part of PRs
- ✅ Session context can be cleaned/archived periodically

---

## Questions?

This system is designed to be self-documenting. If you need clarification:
1. Check the specific memory file
2. Look at the patterns used
3. Reference `.github/copilot-instructions.md` for related guidance

---

**Last Updated:** 2026-04-25
**System Version:** 1.0
**Status:** Initialized
