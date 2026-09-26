---
name: mock-zero-tolerance-in-payment-code
description: Rules that keep fabricated payment results out of a codebase — written for AI-assisted development, where mock code has a habit of reappearing
---

# Zero tolerance for mock code in payment paths

## When this applies

- Any system that moves real money: payments, transfers, refunds.
- Especially when an AI assistant is writing or refactoring the code.

## The rules

### Rule 1 — a catch block never returns fabricated data

```java
// FATAL — and it is the pattern AI assistants reach for first
try {
    return wxPayService.createOrderV3(request);
} catch (Exception e) {
    log.warn("payment failed, using simulated data", e);
    return mockPayResult();     // the user believes they paid
}

// Correct
try {
    return wxPayService.createOrderV3(request);
} catch (Exception e) {
    log.error("payment order creation failed", e);
    throw new ServiceException("payment service unavailable, please retry");
}
```

### Rule 2 — force injection; never leave a `required = false` back door

```java
// Leaves a hole for mock behaviour
@Autowired(required = false)
private WxPayService wxPayService;

// Missing bean = application fails to start
@Resource
private WxPayService wxPayService;
```

### Rule 3 — delete mock code physically, do not comment it out

```java
// Commented-out code is a time bomb: the agent will uncomment it
// private WxPayUnifiedOrderV3Result mockPayResult() { ... }

// Delete the method, every reference, and the imports.
```

## Why AI-assisted development makes this worse

| Trigger | What the agent does | Result |
|---|---|---|
| Model switch | The new model never knew the mock was deleted | Mock comes back |
| Compile error | "Quick fix": wrap in try/catch and return a fallback | Real payments bypassed |
| Context too long | The zero-tolerance decision falls out of context | Mock comes back |

In the original project this happened **three times** before the rule became an explicit, written instruction that every session reads.

## Checklist — run before every release

- [ ] Search the whole repository for `mock` / `Mock` / `MOCK` → must return nothing in payment paths
- [ ] Search for `required = false` → must return nothing around payment beans
- [ ] Every payment `catch` block contains only `throw`, never `return`
- [ ] The skill file states "do not restore mock code"
- [ ] A test asserts that a failed payment produces an error, not a success

*Extracted from real delivery work after three genuine rollbacks.*
