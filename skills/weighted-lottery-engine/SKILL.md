---
name: 加权随机抽奖引擎
description: 概率算法+乐观锁库存+三级兜底的通用抽奖引擎设计，含沙盘测试和积分补偿
---

# 加权随机抽奖引擎

## 适用场景
- 营销抽奖活动（大转盘/九宫格/刮刮乐等）
- 需要概率可配、库存安全、结果可追溯

## 1. 核心算法

```java
public PrizeDO executeRealDraw(List<PrizeDO> prizes, Long activityId) {
    // Step 1: 过滤可用奖品（库存>0 或 无限库存-1）
    List<PrizeDO> available = prizes.stream()
        .filter(p -> p.getRemainCount() == -1 || p.getRemainCount() > 0)
        .collect(Collectors.toList());
    
    // Step 2: 加权随机选择
    double random = Math.random() * 100;  // 概率总和为100%
    double cumulative = 0;
    PrizeDO selected = null;
    for (PrizeDO prize : available) {
        cumulative += prize.getProbability();
        if (random < cumulative) {
            selected = prize;
            break;
        }
    }
    
    // Step 3: 兜底 — 概率未命中
    if (selected == null) {
        selected = findFallbackPrize(prizes); // 找积分奖品(type=2)
    }
    
    // Step 4: 乐观锁扣库存
    if (selected.getRemainCount() != -1) { // -1=无限库存，跳过扣减
        int rows = prizeMapper.deductStock(selected.getId());
        if (rows == 0) {
            selected = findFallbackPrize(prizes); // 扣减失败再兜底
        }
    }
    
    return selected;
}
```

## 2. 乐观锁扣库存 SQL

```sql
UPDATE lottery_prize 
SET remain_count = remain_count - 1 
WHERE id = #{id} AND remain_count > 0
```

> 并发安全：10 人同时抢最后 1 件，只有 1 人 `rows=1`，其他 9 人 `rows=0` 走兜底。

## 3. 三级兜底机制

```
Level 1: isAvailable() 预过滤库存为 0 的奖品
Level 2: 概率未命中 → findFallbackPrize() 找积分奖品
Level 3: 乐观锁扣减失败 → 再次 findFallbackPrize()

保证: 只要配了兜底奖品，永不返回 null
```

## 4. 奖品类型设计

| type | 说明 | 库存 | isWin |
|:----:|------|------|:-----:|
| 1 | 实物奖品 | remainCount ≥ 0 | true |
| 2 | 积分奖品（兜底） | remainCount = -1（无限） | false |

> 积分奖品命中后 `isWin=false`，统计中奖率只算实物。

## 5. 沙盘测试（纯内存模拟）

```java
public Map<String, Object> testDraw(Long activityId, int count) {
    // 不扣库存不写数据库，纯概率模拟
    Map<String, Integer> hitCount = new HashMap<>();
    for (int i = 0; i < count; i++) {
        PrizeDO prize = simulateDraw(prizes); // 不执行 deductStock
        hitCount.merge(prize.getName(), 1, Integer::sum);
    }
    return Map.of("count", count, "results", hitCount);
}
```

## 6. 防作弊设计

- **先扣库存再出结果**：用户看到的动画 = 后端已确认的结果
- **结果后端决定**：前端只接收"转到第几个扇区"，不参与概率计算
- **轮询而非 WebSocket**：防止前端篡改消息

## 来源
提炼自该项目 C2/C4 会话。
