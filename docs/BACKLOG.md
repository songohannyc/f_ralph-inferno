# Ralph Inferno - Backlog

## Potential Extensions

---

### Runtime Discovery Loop

**Problem:** Ralph may discover things during implementation that should affect scope.

**Idea:** Graduated response based on impact:

| Impact | Example | Action |
|--------|---------|--------|
| **Small** | Edge case, minor refactor | Expand in-place, continue |
| **Medium** | New API call, extra component | Create follow-up spec (`02a-discovery.md`) |
| **Large** | Architecture change, new dependency | Pause & notify user |

**Always:** Log discoveries to `DISCOVERIES.md`

---

### Issue-Based Dynamic Model

**Problem:** Current spec model is static - all specs generated in `/ralph:plan`. Ralph cannot dynamically create new tasks during execution.

**Solution:** Switch from `specs/` to `issues/` with dynamic creation.

```
PRD.md = The Fence (guardrails)
issues/ = Dynamic, Ralph creates as needed
```

**Rules:**
- Ralph MAY create issues that contribute to PRD
- Ralph MAY NOT build outside PRD without approval
- Scope changes → CR → `/ralph:review` → Approve/Reject

---

### Blind Validation

**Problem:** When the same agent implements and validates, we get confirmation bias.

**Solution:** Separate validator agent that does NOT see implementation context.

```
Implementer Agent: Writes code based on spec
        ↓
Blind Validator Agent: Reviews CODE ONLY
        ↓
"Code has bug on line 45" (honest feedback)
```

**Implementation:**
```bash
# After implementation (Codex or Claude)
codex exec --dangerously-bypass-approvals-and-sandbox - <<'EOF'
Review this code for bugs. You have NOT seen the spec.
Only review the code itself.
$(cat src/checkout.ts)
EOF
# Alternative (Claude):
# claude --dangerously-skip-permissions -p "
#   Review this code for bugs. You have NOT seen the spec.
#   Only review the code itself.
#   $(cat src/checkout.ts)
# "
```
**Benefits:**
- Finds more bugs
- No "I know what I meant" bias
- More like real code review

---

### SQLite for State

**Problem:** Checksum files and logs are fragile. Hard to query history.

**Solution:** SQLite database for all state.

```sql
CREATE TABLE specs (
  id TEXT PRIMARY KEY,
  name TEXT,
  status TEXT,  -- pending, running, done, failed
  attempts INTEGER,
  tokens_used INTEGER,
  cost_usd REAL,
  started_at TIMESTAMP,
  completed_at TIMESTAMP,
  error_log TEXT
);
```

**Benefits:**
- Crash recovery (resume from exact point)
- Cost tracking per spec
- Query history ("which specs fail most?")

---

### Complexity Classification

**Problem:** Ralph runs all specs the same way, regardless of complexity.

**Solution:** Classify specs and adapt approach.

```
SIMPLE (< 100 tokens)     → Fast, no validation
MEDIUM (100-500 tokens)   → Standard flow
COMPLEX (> 500 tokens)    → Extra validation, more retries
RISKY (modifies auth/db)  → Blind validation + manual review
```

---

### Domain Modules

**Problem:** Ralph is optimized for code. What if you want to create other things?

**Idea:** Domain-specific modules with adapted workflows.

```
/ralph:idea "shark book" --module book
/ralph:idea "todo app" --module app
/ralph:idea "synthwave album" --module music
```

| Module | Output | "Deploy" does | "Test" does |
|--------|--------|--------------|-------------|
| **app** | Code + tests | `npm run build` | Playwright E2E |
| **book** | Chapters (markdown) | Compile PDF/ePub | Spell-check, consistency |
| **music** | Prompts + metadata | Suno/Udio API | Preview + quality check |
| **video** | Script + shot list | Storyboard render | ? |
| **course** | Lessons + quizzes | LMS export | Quiz validation |

**Questions to solve:**
- What is "spec" for a book? (Chapter outline?)
- What is "build pass" for music? (API returns audio?)
- How do we verify quality on creative content?

---

### Design System Validation

**Problem:** Inferno mode has design review but it's basic.

**Idea:** Integrate with design system for automatic validation.

- Read design tokens from Figma/Storybook
- Verify implementation matches
- Screenshot diff against design mockups

---

### Multi-VM Parallel Execution

**Problem:** One VM = one spec at a time. Slow for large projects.

**Idea:** Spin up multiple VMs and run independent specs in parallel.

```
VM-1: 01-setup.md → 02-auth.md
VM-2: 03-feature-a.md (after setup done)
VM-3: 04-feature-b.md (after setup done)
```

**Requires:** Dependency graph, merge strategy, conflict resolution.

---

### Cost Budgets & Alerts

**Problem:** Ralph can burn tokens without control.

**Idea:** Set budget per session/spec.

```json
{
  "cost_budget": {
    "per_spec_max_usd": 5.00,
    "session_max_usd": 50.00,
    "alert_at_percent": 80
  }
}
```

**On budget hit:** Pause, notify, wait for approval.

---

### Git Branch Strategy

**Problem:** Everything goes to main/current branch.

**Idea:** Feature branches per epic/CR.

```
main
├── ralph/epic-1-auth
├── ralph/epic-2-todos
└── ralph/cr-dark-mode
```

**Auto-PR:** When epic done → create PR for review.

---

## Priority

| Feature | Value | Complexity | Notes |
|---------|-------|------------|-------|
| Runtime discovery loop | High | Medium | Builds on existing CR logic |
| Blind validation | High | Low | Easy to implement |
| SQLite state | Medium | Medium | Good for debugging |
| Complexity classification | Medium | Low | Quick win |
| Domain modules | High | High | Major rewrite |
| Design system validation | Medium | High | Requires integrations |
| Multi-VM parallel | High | High | Complex orchestration |
| Cost budgets | Medium | Low | Quick win |
| Git branch strategy | Medium | Medium | Nice to have |
| Issue-based model | High | Medium | Fundamental change |

---

## Inspiration

- [Gas Town](https://github.com/steveyegge/gastown) - Multi-agent orchestration, git worktree persistence
- [Zeroshot](https://github.com/covibes/zeroshot) - Blind validation, SQLite state, complexity classification
- [BMAD Method](https://github.com/bmadcode/BMAD-METHOD) - Brainstorm/Analyst personas

---

*Last updated: 2026-01-19*
