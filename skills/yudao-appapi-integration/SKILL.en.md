---
name: yudao-client-api-integration
description: Adding /app-api/ endpoints for a mini-program or app in the yudao stack — path conventions, security configuration, multi-tenancy
---

# Adding client (`/app-api/`) endpoints in the yudao stack

## When this applies

- Building backend endpoints for a WeChat mini-program or app on `ruoyi-vue-pro` (monolith)
- You need the `/app-api/` prefix, as opposed to the admin-facing `/admin-api/`

## Rule 1 — never write the prefix yourself ⚠️ the most common mistake

```java
// Correct — write only the business path
@RestController
@RequestMapping("/lottery/activity")
public class AppLotteryController { ... }
// effective path: /app-api/lottery/activity/...

// Wrong — the prefix is added twice and every call 404s
@RestController
@RequestMapping("/app-api/lottery/activity")
// becomes /app-api/app-api/lottery/activity
```

**Why**: yudao's `WebMvcConfiguration` adds `/app-api` automatically for controllers inside the `app` package.

## Rule 2 — three configuration points

**2.1 Anonymous endpoints**
```yaml
yudao.security.permit-all_urls:
  - /app-api/your-module/**
```

**2.2 Multi-tenancy bypass** (client requests do not send a `tenant-id` header)
```yaml
yudao.tenant.ignore-urls:
  - /app-api/your-module/**
```

**2.3 `mock-enable` must be false**
```yaml
yudao.security.mock-enable: false
# when true, every request is treated as admin (user_type = 1), so the
# member (user_type = 2) social bindings can never be found
```

## Rule 3 — module layout

```
yudao-module-xxx/
├── yudao-module-xxx-api/         # interface definitions (package-info.java is enough)
└── yudao-module-xxx-biz/         # implementation
    └── src/main/java/.../xxx/
        ├── controller/
        │   ├── admin/            # admin endpoints
        │   └── app/              # client endpoints ← identified by package
        ├── service/
        ├── dal/
        │   ├── dataobject/
        │   └── mysql/
        └── enums/
```

## Rule 4 — split controllers by responsibility

| Controller | Responsibility | Auth |
|---|---|---|
| `AppXxxController` | reads | partly anonymous |
| `AppXxxActionController` | writes (payments, submissions) | login required |
| `AppXxxCallbackController` | third-party callbacks (payment notifications) | anonymous + signature verified |
| `AppMockController` | development only | anonymous — **delete before production** |

## Checklist

- [ ] Controller sits in the `app` package
- [ ] `@RequestMapping` contains no `/app-api` prefix
- [ ] Anonymous endpoints listed in `permit-all-urls`
- [ ] `tenant.ignore-urls` configured
- [ ] `mock-enable: false`
- [ ] Effective path verified with a real `curl` after building

*Extracted from a production mini-program delivery, verified across seven working sessions.*
