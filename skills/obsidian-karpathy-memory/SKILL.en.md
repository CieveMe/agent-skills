---
name: obsidian-karpathy-memory
description: Give a project long-term memory the Obsidian + Karpathy LLM-wiki way — let the AI keep maintaining a plain-Markdown knowledge base (index / hot-cache / log / topic pages / raw sources) instead of relying on chat history. Includes copy-paste bootstrap prompts and a "what to install first" checklist. For new projects, retrofitting an existing one, resuming in a new session, or handing off.
---

# Obsidian + Karpathy-style long-term memory

The claim is one sentence: **do not let a project's memory live in chat history; let it live in a set of plain
Markdown files that an AI keeps maintaining.** This follows Karpathy's framing of the LLM as a *wiki
maintainer* rather than a question-answering machine. The files read, link and search well in Obsidian, and
they are ordinary text that Git handles well.

## Why it saves tokens

Writing more documents is not the point; **reading less** is:

- `hot-cache.md` holds only the current context (what we are doing, next step, blockers) and stays short
  (roughly under 2,500 characters);
- `index.md` is the table of contents — read it first, then decide which page to open;
- stable facts sink into topic/source pages instead of piling up in the cache;
- known-good routes live in `operations/operation-memory.md` so failures are not rediscovered.

## Layout

```text
raw/                      original material, append-only
wiki/
  index.md                the page directory you navigate by
  hot-cache.md            current context (short)
  log.md                  append-only timeline
  overview.md             goal, scope, constraints, success criteria
  sources/                one summary page per source, with its path
  decisions/              decisions and why (including rejected options)
  operations/
    operation-memory.md   working commands/paths + routes not to take + validation
AGENTS.md                 rules every session must follow
```

## Three rules that keep it from rotting

1. **One topic per page.** A fact is maintained in one place; everywhere else links to it.
2. **Conclusions carry their source.** Say whether it is measured / a file path / a link / a user decision,
   or an inference. Inferences do not get filed as facts.
3. **Write back in order.** Change → update the topic page → link new pages from `index.md` → append to
   `log.md` → put only what is *current* into `hot-cache.md`.
4. **Use standard relative Markdown links** — `[title](sources/x.md)` — never Obsidian-only `[[wikilinks]]`.
   Standard links open in Obsidian, VS Code, GitHub and Typora alike; `[[...]]` becomes plain text outside
   Obsidian.

## Obsidian is optional (say this first when sharing)

**Nothing has to be installed.** The memory is a set of `.md` files that the AI reads and writes directly.
Obsidian is the nicest *reader* (backlinks, graph, tags, mobile), but the method works without it: VS Code,
Typora, or the vault pushed to GitHub and read in the browser are all fine. See the four configurations in
[`references/prerequisites.md`](references/prerequisites.md) (zero-install / VS Code / Obsidian / GitHub).

## The three ways it rots, and the fix

| Symptom | Fix |
|---|---|
| `hot-cache.md` grows until nobody reads it | Move stable facts to topic pages; keep the cache about *now*; run a quality check periodically. |
| Two sessions write conflicting versions | Append only your own titled section; do not rewrite someone else's. But if their content is **stale**, fix it on the spot rather than leaving contradictions on one page. |
| An outdated conclusion keeps being quoted | Record every correction in the relevant page ("do not use X; use Y because Z") and leave a trace in `log.md`. |

## Sensitive material

Passwords, keys, ID numbers and tokens are recorded as **where to get them**, never in plaintext. Public
contact details and public links are fine. The knowledge base is for the future you and the AI, not a safe.

## Getting started

1. What to install, where to put the vault, how to verify →
   [`references/prerequisites.md`](references/prerequisites.md)
2. Copy-paste prompts (initialize / resume in a new session / write back / record an operation route) →
   [`references/bootstrap-prompts.md`](references/bootstrap-prompts.md)

If the environment already has a generic `project-wiki-memory` skill it ships initializer and quality-check
scripts; if not, nothing is missing — **the method itself needs only folders and Markdown.**
