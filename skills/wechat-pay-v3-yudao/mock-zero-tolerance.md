---
name: 微信支付V3芋道集成
description: 在芋道框架中集成微信支付V3(JSAPI)的完整流程，含证书部署、回调验签、Mock零容忍策略
---

# Mock 零容忍支付安全策略

## 适用场景
- 任何涉及真金白银的系统（支付、转账、退款）
- AI 辅助开发环境下尤其关键

## 核心规则

### 规则 1: catch 块永远不返回假数据

```java
// ❌ 致命错误 — AI 最喜欢写的代码
try {
    return wxPayService.createOrderV3(request);
} catch (Exception e) {
    log.warn("支付失败，使用模拟数据", e);
    return mockPayResult();  // 💀 用户以为付成功了
}

// ✅ 正确写法
try {
    return wxPayService.createOrderV3(request);
} catch (Exception e) {
    log.error("支付下单失败", e);
    throw new ServiceException("微信支付服务异常，请稍后重试"); // 直接抛
}
```

### 规则 2: 强制注入，不允许 required=false

```java
// ❌ 给 Mock 留后门
@Autowired(required = false)
private WxPayService wxPayService;

// ✅ 缺失即启动失败
@Resource
private WxPayService wxPayService;
```

### 规则 3: 物理删除，不是注释

```java
// ❌ 注释 = 定时炸弹（AI 会自动取消注释）
// private WxPayUnifiedOrderV3Result mockPayResult() { ... }

// ✅ 彻底删除方法 + 所有引用 + import
// （什么都不留）
```

## 为什么 AI 开发中特别危险？

| 触发场景 | AI 行为 | 后果 |
|----------|---------|------|
| 模型切换 | 新模型不知道之前删了 Mock | 恢复 Mock |
| 编译报错 | AI 为快速修复加回 try-catch + Mock | 绕过真实支付 |
| 上下文过长 | AI 忘记"零容忍"决策 | 恢复 Mock |

**本项目实际发生 3 次回退**，最终确立规则。

## 检查清单

- [ ] 全文搜索 `mock`/`Mock`/`MOCK` — 必须零结果
- [ ] 全文搜索 `required = false` — 支付相关必须零结果
- [ ] 所有支付 catch 块只有 `throw`，无 `return`
- [ ] Skill 文件中写明 "禁止恢复 Mock"

## 来源
提炼自该项目 C3，3 次真实回退后确立。
