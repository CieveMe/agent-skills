# AI 辅助开发 Prompt 模板库

## 1. 项目交接 Prompt

```
你好，我要继续开发一个项目。请先阅读以下文件了解上下文：

1. `.agents/skills/` 目录下所有 SKILL.md（技术规范）
2. `总结.md`（项目完整知识库，1035行）
3. `task.md`（当前进度，如有）

当前需要继续的工作：
- [具体任务描述]

关键约束：
- mock-enable 必须为 false
- Controller 不手写 /app-api 前缀
- 支付链路禁止 Mock 代码
```

## 2. 需求分析 Prompt

```
我有一个新客户需要 [行业] 的 [产品类型]。

请基于我们的抽奖营销模板（总结.md）分析：
1. 哪些模块可以直接复用？
2. 哪些需要定制开发？
3. 预计工作量（参考总结.md第十一章的模块估算）
4. 报价建议（参考标准版¥79,800和经济版¥8,000的定价策略）

输出为 Markdown 表格。
```

## 3. 代码审计 Prompt

```
请对以下代码进行安全审计，重点关注：

1. 支付链路是否有 Mock/fallback 代码？
2. catch 块是否返回了假数据？
3. @Autowired(required=false) 是否用在了关键服务上？
4. YAML 配置是否有绑定可能失败的 @ConfigurationProperties？
5. mock-enable 是否为 true？

参考规则：.agents/skills/wechat-pay-v3-yudao/mock-zero-tolerance.md
```

## 4. 部署前检查 Prompt

```
即将部署到生产环境，请执行部署前检查：

1. infra_file_config.domain 是否为生产域名？（非 127.0.0.1）
2. mock-enable 是否为 false？
3. Redis 密码配置是否正确？（空密码需注释掉 password 行）
4. MySQL 密码含特殊字符是否正确转义？
5. Nginx 配置是否有 try_files $uri $uri/ /index.html？
6. SSL 证书是否挂载？
7. 微信支付回调 URL 是否为生产域名？

参考：.agents/skills/docker-springboot-production/SKILL.md
```

## 5. 新会话启动 Prompt

```
这是一个新会话。请先执行以下操作：

1. 读取 .agents/skills/ 下所有 SKILL.md
2. 读取总结.md 的目录结构（第一章）
3. 读取 task.md（如有）

然后告诉我你已了解的项目背景摘要（3句话内）。
```

## 适用场景
- 复制粘贴到新 AI 会话中
- 每个 Prompt 都经过实际项目验证
- 可根据客户需求修改关键词
