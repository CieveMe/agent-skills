---
name: 芋道C端接口快速接入
description: 在芋道 ruoyi-vue-pro 框架中为小程序/App等C端创建 /app-api/ 接口的标准流程，含安全配置和多租户处理
---

# 芋道 C 端接口快速接入指南

## 适用场景
- 基于芋道 ruoyi-vue-pro 单体版开发微信小程序/App 后端接口
- 需要 `/app-api/` 前缀的 C 端接口（区别于 `/admin-api/` 管理端）

## 核心规则

### 1. Controller 路径规范 ⚠️ 最易踩坑

```java
// ✅ 正确写法 — 只写业务路径，框架自动加 /app-api 前缀
@RestController
@RequestMapping("/lottery/activity")
public class AppLotteryController { ... }
// 实际访问路径: /app-api/lottery/activity/xxx

// ❌ 错误写法 — 手写前缀导致双重叠加 → 全部 404
@RestController
@RequestMapping("/app-api/lottery/activity")  // 会变成 /app-api/app-api/lottery/activity
```

**原理**: 芋道通过 `WebMvcConfiguration` 自动为 `app` 包下的 Controller 添加 `/app-api` 前缀。

### 2. 安全配置（3 处必改）

#### 2.1 匿名接口放行
```yaml
# application.yaml
yudao.security.permit-all_urls:
  - /app-api/your-module/**  # 无需登录的接口
```

#### 2.2 多租户忽略
```yaml
yudao.tenant.ignore-urls:
  - /app-api/your-module/**  # C端不传 tenant-id Header
```

#### 2.3 mock-enable 必须为 false
```yaml
yudao.security.mock-enable: false
# true 会劫持所有用户身份为 admin(user_type=1)
# 导致 member(user_type=2) 的社交绑定查不到
```

### 3. 模块结构

```
yudao-module-xxx/
├── yudao-module-xxx-api/         # 接口定义（可放 package-info.java 占位）
└── yudao-module-xxx-biz/         # 业务实现
    └── src/main/java/.../xxx/
        ├── controller/
        │   ├── admin/            # 管理端 Controller
        │   └── app/              # C端 Controller ← 框架按包名识别
        ├── service/
        ├── dal/
        │   ├── dataobject/
        │   └── mysql/
        └── enums/
```

### 4. Controller 按职责拆分建议

| Controller | 职责 | 鉴权 |
|------------|------|------|
| AppXxxController | 查询类接口 | 部分匿名 |
| AppXxxActionController | 写操作（支付/提交） | 必须登录 |
| AppXxxCallbackController | 第三方回调（支付通知） | 匿名 + 验签 |
| AppMockController | 开发测试 | 匿名（生产删除） |

### 5. 检查清单

- [ ] Controller 在 `app` 包下
- [ ] `@RequestMapping` 不含 `/app-api` 前缀
- [ ] `permit-all_urls` 已配置匿名接口
- [ ] `tenant.ignore-urls` 已配置
- [ ] `mock-enable: false`
- [ ] 编译后实际请求路径正确（curl 验证）

## 来源
提炼自一个生产级抽奖营销小程序项目，经 7 个 Conversation 实战验证。
