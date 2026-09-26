---
name: weighted-random-draw-engine
description: A general-purpose draw engine — weighted probability, optimistic-lock stock, a three-level fallback, sandbox simulation and points compensation
---

# Weighted random draw engine

## When this applies

- Marketing draws (wheel, 3×3 grid, scratch cards, …)
- Requirements: configurable probability, stock that cannot go negative, and results that can be explained afterwards

## 1. Core algorithm

```java
public PrizeDO executeRealDraw(List<PrizeDO> prizes, Long activityId) {
    // Step 1: keep only prizes that are actually available
    List<PrizeDO> available = prizes.stream()
        .filter(p -> p.getRemainCount() == -1 || p.getRemainCount() > 0)
        .collect(Collectors.toList());

    // Step 2: weighted random pick (probabilities are expressed as percentages)
    double random = Math.random() * 100;
    double cumulative = 0;
    PrizeDO selected = null;
    for (PrizeDO prize : available) {
        cumulative += prize.getProbability();
        if (random < cumulative) {
            selected = prize;
            break;
        }
    }

    // Step 3: fallback when the random number landed in no slice
    if (selected == null) {
        selected = findFallbackPrize(prizes);   // the points prize (type = 2)
    }

    // Step 4: deduct stock with an optimistic lock
    if (selected.getRemainCount() != -1) {      // -1 means unlimited
        int rows = prizeMapper.deductStock(selected.getId());
        if (rows == 0) {
            selected = findFallbackPrize(prizes); // someone else took the last one
        }
    }

    return selected;
}
```

## 2. The optimistic-lock stock deduction

```sql
UPDATE lottery_prize
SET remain_count = remain_count - 1
WHERE id = #{id} AND remain_count > 0
```

> Why this is concurrency-safe: with ten users racing for the last unit, exactly one gets `rows = 1`; the other nine get `rows = 0` and fall through to the fallback prize. No row ever goes negative, and no distributed lock is needed.

## 3. Three levels of fallback

```
Level 1: isAvailable() filters out prizes with zero stock
Level 2: random number matches no slice  → findFallbackPrize()
Level 3: optimistic-lock deduction fails → findFallbackPrize() again

Guarantee: as long as a fallback prize is configured, the draw never returns null
```

The important part is not the three levels — it is that "no prize" is a *defined* outcome rather than an exception path. A draw that returns null is a support ticket.

## 4. Modelling prizes

| type | Meaning | Stock | Counts as a win |
|:--:|---|---|:--:|
| 1 | physical prize | `remainCount >= 0` | yes |
| 2 | points prize (the fallback) | `remainCount = -1` (unlimited) | no |

> A points prize is recorded with `isWin = false`, so "win rate" statistics only count physical prizes. Otherwise a campaign looks like it has a 100% win rate and the client loses trust in the numbers.

## 5. Sandbox simulation (pure in-memory)

```java
public Map<String, Object> testDraw(Long activityId, int count) {
    // no stock deduction, no database writes — probability only
    Map<String, Integer> hitCount = new HashMap<>();
    for (int i = 0; i < count; i++) {
        PrizeDO prize = simulateDraw(prizes);   // skips deductStock
        hitCount.merge(prize.getName(), 1, Integer::sum);
    }
    return Map.of("count", count, "results", hitCount);
}
```

Run this for 10,000 draws before a campaign goes live. It is the cheapest way to catch a probability table that does not add up to 100%, or a "rare" prize that actually lands 30% of the time.

## 6. Anti-cheat design

- **Deduct stock before showing the result.** The animation the user sees is a replay of a decision the backend already made.
- **The backend decides everything.** The frontend receives "land on sector N" — it never computes probability.
- **Poll, don't trust push.** Client-side modification of a WebSocket message is harder to detect than a response to a poll.

*Extracted from real delivery work (two sessions) and sanitised.*
