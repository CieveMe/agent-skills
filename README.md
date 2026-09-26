# agent-skills

Reusable **agent skills** extracted from real delivery work — a production WeChat mini-program with payments, a video-surveillance platform, and a few dozen deployment incidents I would rather not repeat.

These are the rule sets I actually give my coding agents (Claude Code / Codex / Cursor). They are not prompt collections: each one is a `SKILL.md` that describes a failure mode, how to detect it, and the order to do things in.

> The skill bodies are written in Chinese (my working language). Where an English version exists it is named `SKILL.en.md` next to the original; the table below always summarises the skill in English.

## Skills

| Skill | What it prevents |
|---|---|
| [`ai-dev-best-practices`](skills/ai-dev-best-practices/SKILL.md) · [EN](skills/ai-dev-best-practices/SKILL.en.md) | Doing AI-assisted development in the wrong order: no plan, no checkpoints, token burn, agents re-doing work they already did, and the five reasons the same bug survives five "fixes". 35 pitfalls collected across 7 sessions. |
| [`deployment-pitfalls`](skills/deployment-pitfalls/SKILL.md) · [EN](skills/deployment-pitfalls/SKILL.en.md) | Six remote-deployment traps: non-ASCII paths silently breaking `ssh`/`scp` on Windows, hot-swapping a JAR and getting `ClassNotFoundException`, CRLF poisoning shell scripts, PowerShell console encoding corrupting CJK data, and quoting hell through four nested interpreters. |
| [`docker-springboot-production`](skills/docker-springboot-production/SKILL.md) · [EN](skills/docker-springboot-production/SKILL.en.md) | Shipping a Spring Boot app with a dev-grade Docker setup: MySQL 8.4 rejecting the old auth flag, `#` in a password truncated by the shell, Redisson failing AUTH against a passwordless Redis, wrong file-upload domain after deploy. |
| [`database-operation-safety`](skills/database-operation-safety/SKILL.md) · [EN](skills/database-operation-safety/SKILL.en.md) | A data-changing `UPDATE`/`DELETE` that quietly touches the wrong rows, because a filter condition in the instruction was read as background prose. |
| [`wechat-pay-v3-yudao`](skills/wechat-pay-v3-yudao/SKILL.md) · [EN](skills/wechat-pay-v3-yudao/SKILL.en.md) | Getting WeChat Pay V3 integration wrong: certificate placement, silent `@ConfigurationProperties` binding failures, callback verification, callback idempotency, and mixing up the callback result type with the order result type. |
| [`wechat-pay-v3-yudao/mock-zero-tolerance.md`](skills/wechat-pay-v3-yudao/mock-zero-tolerance.md) · [EN](skills/wechat-pay-v3-yudao/mock-zero-tolerance.en.md) | Shipping payment code that "works" against mocks. Written after three real rollbacks — and specifically about how AI assistants resurrect mock code. |
| [`weighted-lottery-engine`](skills/weighted-lottery-engine/SKILL.md) · [EN](skills/weighted-lottery-engine/SKILL.en.md) | Weighted-random draw engines that are unfair, unreproducible, or cannot change probability at runtime: optimistic-lock stock, a three-level fallback so a draw never returns null, a 10,000-draw sandbox, and anti-cheat design. |
| [`yudao-hot-config-pattern`](skills/yudao-hot-config-pattern/SKILL.md) · [EN](skills/yudao-hot-config-pattern/SKILL.en.md) | Campaign parameters that require a redeploy to change. The pattern: an independent endpoint plus a lightweight modal, editable at any time with zero intrusion into the existing code paths. |
| [`yudao-codegen-pitfalls`](skills/yudao-codegen-pitfalls/SKILL.md) · [EN](skills/yudao-codegen-pitfalls/SKILL.en.md) | Trusting generated CRUD code: output landing in the parent POM, hyphens in package names, duplicate menu component names, dictionaries that were never inserted, empty modules breaking the build. |
| [`yudao-appapi-integration`](skills/yudao-appapi-integration/SKILL.md) · [EN](skills/yudao-appapi-integration/SKILL.en.md) | Wiring a mini-program client to the framework's member/auth system the long way — including the double-prefix 404 and the `mock-enable` flag that silently turns every user into an admin. |
| [`yudao-admin-branding`](skills/yudao-admin-branding/SKILL.md) · [EN](skills/yudao-admin-branding/SKILL.en.md) | White-labelling the admin console by hand and missing half the places the old name appears; also why a framework dashboard should be emptied before handover. |

Plus one workflow and one resource:

- [`workflows/quick-delivery.md`](workflows/quick-delivery.md) — turning a previous delivery into a new client's branded instance in half a day.
- [`resources/prompt-templates.md`](resources/prompt-templates.md) — the prompt shapes I reuse for requirements, code review and debugging.

## How to use them

Skills are just directories containing a `SKILL.md`. Copy the ones you want into your agent's skills folder:

```bash
# Claude Code (user scope)
cp -r skills/* ~/.claude/skills/

# Codex
cp -r skills/* "$CODEX_HOME/skills/"
```

Or drop a single folder into `.claude/skills/` inside a project. Each skill is self-contained; nothing here phones home or needs credentials.

## Notes on provenance and privacy

- Extracted from real projects, then **sanitised**: no client names, no domains, no server addresses, no merchant IDs, no keys, no customer data. Placeholders such as `<your-server-ip>` or `your-domain.com` are what you will find.
- Numbers and incidents are kept because they are the point; identities are not.
- Everything is my own write-up. Where a lesson came from a framework's behaviour, the framework is named (e.g. `yudao` / ruoyi-vue-pro, which is open source) — no proprietary code is included.

## Roadmap

- ~~English translations of the skill bodies~~ **done: 11 of 11** (every skill has a `SKILL.en.md` beside it)
- The `mysql-ops-mcp` pattern as a skill: read-only data access for agents
- A checklist skill for "deliverable" — what must exist before a handover counts as done

## License

MIT — see [LICENSE](LICENSE). Use them, fork them, adapt them.

---

**Zhen He** · Java / AI application engineer · [GitHub](https://github.com/CieveMe) · [Case studies](https://github.com/CieveMe/portfolio) · [LinkedIn](https://www.linkedin.com/in/zhen-he-a2336a43a)
