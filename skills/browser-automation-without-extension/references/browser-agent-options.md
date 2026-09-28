# Other agents and tools that can drive a browser

Every entry below was verified to exist on 2026-09-29 (GitHub API, star counts are a rough popularity signal
only). Capabilities change quickly — **check the project's own documentation before relying on a specific
feature**; this page is a starting list, not a specification.

## Protocol-level: give any MCP-capable agent a browser

| Project | What it is | Why it matters here |
|---|---|---|
| [`microsoft/playwright-mcp`](https://github.com/microsoft/playwright-mcp) (~37.6k★) | Playwright exposed as an MCP server | The commonest way to hand Claude Code / Codex / any MCP client real browser control. Can drive a persistent profile, so logins survive. |
| [`ChromeDevTools/chrome-devtools-mcp`](https://github.com/ChromeDevTools/chrome-devtools-mcp) (~52.7k★) | "Chrome DevTools for coding agents" | Official-adjacent, CDP-level: closer to what this skill does by hand, plus performance/network tooling. |
| [`modelcontextprotocol/servers`](https://github.com/modelcontextprotocol/servers) (~90.6k★) | Reference MCP servers, incl. puppeteer/playwright-style ones | The lowest-effort starting point if you only need "navigate, click, extract". |

## Agent products with built-in browsing

| Project | What it is |
|---|---|
| [`browser-use/browser-use`](https://github.com/browser-use/browser-use) (~116k★) | Python library for agents that use the browser; the most common self-hosted route. |
| [`All-Hands-AI/OpenHands`](https://github.com/All-Hands-AI/OpenHands) (~89k★) | Open-source software-development agent with a browser environment. |
| [`browserbase/stagehand`](https://github.com/browserbase/stagehand) (~25k★) | SDK for acting on real websites (Browserbase's hosted-browser product). |
| [`Skyvern-AI/skyvern`](https://github.com/Skyvern-AI/skyvern) (~23k★) | Automating browser workflows with AI, aimed at repetitive back-office flows. |
| [`browserless/browserless`](https://github.com/browserless/browserless) (~13.7k★) | Headless browsers in Docker; the infrastructure layer under several of the above. |

## Vendor computer-use models

Anthropic publishes a reference computer-use implementation in
[`anthropics/claude-quickstarts`](https://github.com/anthropics/claude-quickstarts) (~17.7k★). OpenAI and Google
also ship computer-use / browser-agent capabilities. These are **model** offerings rather than finished
products: they drive a VM or a browser through screenshots and clicks, so they are slower and more expensive
per step than a CDP-level tool, but they handle sites with no API and heavy JS.

## How to choose

- **You need the user's existing sessions** (an account you are not allowed to re-login) → attach to their
  browser over CDP, which is what `SKILL.md` describes, or Playwright MCP with a persistent profile.
- **You need reliability on one site you control** → plain Playwright/Puppeteer script; no model in the loop.
- **You need "figure it out from a description" on an unfamiliar site** → a browser agent (browser-use,
  Stagehand, Skyvern) or a computer-use model.
- **You need volume and isolation** → Browserless or a hosting provider, one container per job.

Two cautions that apply to all of them:

1. **A fresh instance has no logins.** Most "the agent cannot reach the site" reports are really "the agent is
   in a clean profile". Confirm which browser and which profile you are attached to before debugging anything
   else.
2. **Who pays for the mistake.** Publishing, paying, deleting and permission changes are the actions where an
   autonomous browser agent is most expensive to be wrong about; keep those behind an explicit check.
