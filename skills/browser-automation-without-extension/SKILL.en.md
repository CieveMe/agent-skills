---
name: browser-automation-without-extension
description: When a browser extension (an @chrome-style plugin) stops working, drive the user's already-logged-in browser from the command line over CDP — reuse the session, fill forms, pick from dropdowns, upload files, publish, and verify by reading the page back. For "the extension is dead / there is no extension, but the site still has to be operated".
---

# Browser operations without the extension

A dead extension does not mean browser work is impossible. As long as the browser is **already logged in by
the user**, you can attach to it over CDP from a shell. This skill is what is left after the failures.

## The one requirement

**Operate the user's logged-in browser — never a fresh instance.** A fresh instance has no sessions, so it
can do nothing useful, and it *looks* like a successful connection. That illusion costs the most time
(pitfall 1).

## The core loop

```bash
agent-browser --auto-connect tab list        # real user tabs => the channel is healthy
agent-browser --auto-connect tab <n>          # select the target tab
agent-browser --auto-connect open <url>
agent-browser --auto-connect snapshot -i
agent-browser --auto-connect click <sel>
agent-browser --auto-connect fill <sel> "text"
agent-browser --auto-connect upload <sel> <files...>
agent-browser --auto-connect keyboard type "text"   # real keystrokes; required by autocompletes
agent-browser --auto-connect press Enter
agent-browser --auto-connect eval '(<js>)()'        # read values / verify / click as a last resort
```

Two rules:

1. **Re-select the tab before every action** — selection does not survive between processes, and the current
   tab drifts.
2. **Read the page back after every step** (fields, chips, list items via `eval`). "I clicked it" is not
   evidence. Publish/submit actions are only done when the page itself says so.

## Pitfalls (all measured, not guessed)

1. **Do not use `connect <port>` as a diagnostic.** It happily launches a brand-new blank browser instance,
   after which `--auto-connect` keeps attaching to *that* — `tab list` shows a single `about:blank` and the
   channel looks broken. Fix: `agent-browser close` to kill the stray instance, then `--auto-connect` again
   to get back to the user's browser.
2. **`os error 10060` (cannot read the tab list) does not mean the browser is closed.** Check pitfall 1
   first; do not send the user off to reinstall their browser.
3. **Do not probe the debugging port to test the channel.** Chrome may listen on 9222 while `/json/version`
   and `/json/list` return 404. Listening ≠ controllable.
4. **React form fields live in two worlds.** Injecting `input` events usually writes the value; **dropdown
   selection and dialog submission need real clicks** — JS `click()` often does nothing there.
   - MUI autocomplete: options are `li.MuiAutocomplete-option`; match the text exactly, then click.
   - Keyboard `Type + Enter` selects the **first** suggestion, which is not an exact match — that is how we
     once added `Python Fire` by accident. Dump the options first, then choose; and know how to remove a chip
     afterwards (its remove control may have no `aria-label`; locate it by the outer text `Remove <X> Tag`).
   - Dialog buttons: a global `button:has-text("Add")` matches several elements; scope it to the dialog.
   - **Pressing Enter does not submit every dialog.** Some need their own `Add` button clicked.
5. **Hover-only buttons**: synthetic `mouseover` does nothing. Move the real mouse (`mouse move <x> <y>`),
   then click.
6. **File uploads**: pages often contain several `input[type=file]` elements. Target by structure (e.g.
   `div.air3-modal-body > input[type=file]`), never a bare `input[type=file]` (it matches more than one and
   errors out).
7. **A link preview stuck on `Verifying…` usually does not block submission.** The common cause is a 302
   cross-domain redirect (e.g. `doi.org` → the target site) that the preview fetcher cannot follow. Use the
   target site's direct URL and the card renders; validation generally only checks that a content item exists.
8. **Anti-bot walls.** Cloudflare / PerimeterX "Please wait…" — ask the user to clear it by hand or change
   proxy node; **do not retry in a loop**. Note that the exit IP shown on the page may not be the machine's
   real IP (a proxy is in use), which explains many "but it works on my side" disagreements.
9. **Restore the viewport.** After `set viewport <w> <h>` for a screenshot, set it back near the user's size.
10. **A navigation timeout does not mean the page is empty**: after `page.goto: Timeout` the page is often
    already loaded; try `eval` before starting over.

## Working rules

- Do only what was asked. **After any publish / submit / payment action, read the result back** and record the
  evidence (URL, field values, which tab/section the item landed in) in the project's memory.
- Steps needing **a face, an ID document, a CAPTCHA, a password or a payment** belong to the user. Do not click
  anti-bot controls.
- Destructive actions (delete, refund, wipe, permission change) — confirm *what* is targeted first.
- Platform-level rules (e.g. "platform escrow only, never advance money, never receive on someone's behalf")
  belong in the platform's own notes, not in your memory.

## Other agents and tools that can drive a browser

If the CLI route is unavailable or you want a different environment, see
[`references/browser-agent-options.md`](references/browser-agent-options.md): Playwright MCP, Chrome DevTools
MCP, browser-use, OpenHands, Stagehand, Skyvern, Browserless, and the vendors' computer-use models.

