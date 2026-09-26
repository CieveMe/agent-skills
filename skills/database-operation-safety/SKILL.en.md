---
name: database-operation-safety
description: A mandatory decomposition step before any data-changing SQL, so a filter condition hidden in a sentence never gets dropped
---

# Database change safety rule

## The mandatory step: turn the instruction into a condition list *before* writing SQL

Before generating any `UPDATE` / `DELETE` / `INSERT`, do this first:

1. **Split the user's instruction word by word.** Pull out every qualifier, adjective and adverbial phrase.
2. **Write a condition list.** Map each item to a SQL predicate — or mark it "no mapping" and say why.
3. **Show the list to the user for confirmation.** Only then write the SQL.

### Example

Instruction: *"For the active campaign 1, change the prizes named '100 yuan', '200 yuan' and '500 yuan' so they no longer include the word yuan."*

Decomposition:

| Qualifier | SQL condition |
|---|---|
| the active campaign | `a.status = 0` |
| campaign 1 | `a.name = 'campaign 1'` |
| prize name | target column: `lottery_prize.name` |
| '100 yuan', '200 yuan', '500 yuan' | `p.name IN ('100 yuan','200 yuan','500 yuan')` |
| remove the word "yuan" | `SET name = REPLACE(name, '元', '')` |

→ result: three rows, one column, one string operation. If any of those mapped conditions feels wrong, the user can correct it *before* anything is executed.

## Why this rule exists

In natural language, qualifiers are easy to misread as "descriptive background" rather than "filter conditions" — especially when the instruction already sounds precise enough to act on. Forcing the decomposition breaks that shortcut: every word has to be either mapped to a condition or explicitly dismissed.

The failure mode this prevents is not a syntax error. It is a perfectly valid `UPDATE` that quietly touches the wrong rows.

*Extracted from real delivery work and sanitised.*
