# Other agents and tools that can drive a browser

Every entry below was verified to exist on 2026-09-29 (GitHub API, star counts are a rough popularity signal
only). Capabilities change quickly — **check the project's own documentation before relying on a specific
feature**; this page is a starting list, not a specification.

## AI editors and coding assistants (the layer this question is usually about)

The distinction that matters is not the brand, it is **whether the assistant can attach to the browser you are
already logged into**, read the page back after every action, and perform actions that need real events
(file upload, dropdown selection, dialog submission).

Verified against vendor documentation on 2026-09-29:

| Editor / assistant | Browser capability | Evidence |
|---|---|---|
| **Cursor** | Built-in **Browser**: the agent can control a browser to test applications, audit accessibility and turn designs into code, with console and network access. | `cursor.com/docs/agent/browser`, read directly |
| **Cline** (VS Code extension) | Its own docs describe it as an agent that "can read and write files, run terminal commands, **use a browser**". | `docs.cline.bot`, read directly |
| **Claude Code** (terminal agent) | No built-in browser, but first-class **MCP** support — add Playwright MCP or chrome-devtools-mcp and you get the same shape as this skill. | `docs.claude.com/.../claude-code/mcp`, read directly |
| **VS Code + Copilot Chat** | Supports MCP tools, so a browser MCP server can be attached. Whether a native browser tool ships in your version: check locally. | not verified — the docs page failed to load during writing |
| **Windsurf (Cascade)** | Has browser preview and web search; no documentation found for an agent driving third-party sites. | not verified |
| **Zed / JetBrains (Junie) / Gemini CLI / Trae / Qoder** | Generally MCP-capable, so a browser MCP server can usually be attached; how much is native differs per product. | not verified |

**Three questions that decide whether the experience matches Codex:**

1. Can it attach to **your existing browser profile** (not a fresh, logged-out instance)?
2. Does it **read the page back** after each action (DOM or screenshot), rather than assuming the click worked?
3. Can it do actions that require **real events** — file upload, autocomplete selection, dialog submission?

All three → comparable to Codex. Only the first two → it can look and click simple elements, and will fail on
the forms that matter.

**The portable answer if you do not want to switch editors:** attach
[`microsoft/playwright-mcp`](https://github.com/microsoft/playwright-mcp) or
[`ChromeDevTools/chrome-devtools-mcp`](https://github.com/ChromeDevTools/chrome-devtools-mcp) with a persistent
profile. That reproduces this skill's route inside almost any editor that speaks MCP.

**Keep a command-line fallback.** The native/bridge route depends on the vendor's own plumbing (a browser
extension, a native host, an app bridge) and that plumbing can break independently of the model. A CLI + CDP
path — what `SKILL.md` describes — is the fallback that keeps working when the built-in one does not.

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
