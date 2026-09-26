---
name: ai-assisted-development
description: Practices for working with AI coding assistants on long projects — token waste, duplicated work, and why the same bug keeps coming back
---

# AI-assisted software development — best practices

## When this applies

- You use an AI coding assistant (Claude Code, Cursor, Codex, Gemini, …) to build a real project.
- The work spans **many sessions**, so context does not survive in one conversation.

## 1. Cutting token waste

| Practice | Why | Typical saving |
|---|---|:--:|
| **Start a fresh session before step ~900** | Context bloat means ~70% of tokens are spent re-reading old content | 70% |
| **Read files by line range**, not whole files | Full-file reads are mostly irrelevant | 50–80% |
| **Pin env vars and config into `task.md`** | Stops the agent from searching for config every time | 30% |
| **Confirm several options in one round** | Fewer back-and-forth iterations | 40% |
| **Write knowledge into skill files** | New sessions read them instead of re-explaining context | 60% |

## 2. Stopping the agent from redoing work

**2.1 Ship knowledge as skill files.** Keep `SKILL.md` files in the project (e.g. `.agents/skills/<name>/SKILL.md`). A new session picks them up instead of being re-briefed.

**2.2 Track progress in `task.md`.**

```markdown
- [x] Database schema designed
- [x] Backend CRUD done
- [/] WeChat Pay integration in progress
- [ ] Frontend integration test
```

The agent stops "helpfully" rebuilding what already exists.

**2.3 Always write a handover note at the end of a session.**

```markdown
Next session:
1. Bug X is located at YYY.java:123
2. Environment config is in task.md
3. Do NOT restore the mock payment code (see skill section 5)
```

## 3. Why the same bug survives five "fixes"

**3.1 Switching models loses context.** After a model switch, have the new model read the skill/task files first.

**3.2 The agent "helpfully" restores code you deleted.** ⭐ The most dangerous one.
Physical deletion, not commenting-out; and state "do not restore X" in the skill file. In one project, mock payment code came back three times.

**3.3 Bulk replace operations eat `import` statements.** When a search-and-replace region includes imports, they get dropped or duplicated. Keep import edits as a separate operation from logic edits.

**3.4 YAML indentation errors fail silently.** `@ConfigurationProperties` binds everything to `null` with no error. Prefer explicit `@Value` injection so a missing key fails at startup instead of at runtime.

**3.5 The same setting lives in two places.** YAML is correct but the database config table was never updated (a classic with payment keys). Keep a table in `task.md` listing every place a setting must be synced.

## 4. Where AI is safe and where it is not

| Area | Quality | Guidance |
|---|---|---|
| CRUD scaffolding | excellent | Use it freely |
| Config files | good | Review YAML indentation by hand |
| Algorithmic logic | good | Watch the boundary conditions |
| **Payments / security** | **risky** | **Human line-by-line review required** |
| UI / logo design | mediocre | Use the original brand assets |
| Deployment & ops | usable | Windows → Linux has many traps |

## 5. Costing and delivery structure

| Item | Approach |
|---|---|
| Standard quote | market rate for the scope |
| Discounted tier | AI assistance lowers the cost base, so a lower-priced tier is possible — but decide the floor deliberately, not by accident |
| Payment milestones | requirements confirmed 40% → integration accepted 30% → go-live delivered 30% |
| Maintenance | first 3 months included, then a monthly retainer |

> Pricing figures from the original project were removed before publishing this skill.

*Extracted from a production mini-program project: 7 sessions, 35 distinct pitfalls.*
