---
name: 芋道管理后台品牌定制
description: 芋道 yudao-ui-admin-vue3 管理后台的登录页精简、首页精简、Logo替换、标题修改完整流程
---

# 芋道管理后台品牌定制指南

## 适用场景
- 芋道 yudao-ui-admin-vue3 交付给客户前的品牌化
- 需要精简默认的复杂 UI

## 1. 系统标题修改

### 1.1 环境变量
```env
# .env
VITE_APP_TITLE=示例公司
```

### 1.2 HTML title
```html
<!-- index.html -->
<title>示例公司</title>
```

## 2. Logo 替换（3 处）

```
src/assets/imgs/logo.png      → 侧边栏 Logo
src/assets/imgs/logo.gif      → 登录页 Logo
src/assets/imgs/avatar.gif    → 默认头像
```

> ⚠️ 教训：客户 Logo 是 JPG，直接 copy 覆盖 PNG/GIF 文件即可（浏览器按 MIME 显示）。不要花时间让 AI 生成替代 Logo（7 次迭代都不满意，最终直接用原始文件）。

## 3. 登录页精简

### LoginForm.vue 保留项
```
✅ 租户名称
✅ 用户名
✅ 密码
✅ 登录按钮
❌ 手机号登录 tab
❌ 二维码登录
❌ 验证码
❌ 记住我
❌ 第三方登录图标
❌ "还没有账号？去注册"
```

### Login.vue 修改
```vue
<!-- 移除右上角语言/主题切换 -->
<!-- 移除底部装饰图片 -->
<!-- 保留居中登录卡片 -->
```

## 4. 首页精简

### Home/Index.vue
```vue
<!-- 原始首页元素 → 全部移除 -->
❌ ECharts 图表
❌ 项目通知
❌ 快捷入口
❌ GitHub 链接
❌ 贡献者列表

<!-- 保留 → 简单欢迎语 -->
✅ "欢迎使用 XXX 管理系统"
```

## 5. 移除危险操作按钮

在业务模块的 `index.vue` 中移除：
```vue
❌ 导出按钮（客户不需要）
❌ 批量删除按钮（误操作风险）
❌ 重置按钮（搜索栏够用）
```

## 6. 构建与部署

```bash
# 本地构建（服务器内存不够 npm install）
npm run build:prod

# 打包上传
tar -czf admin-dist.tar.gz dist/
scp admin-dist.tar.gz user@server:/opt/app/
```

## 检查清单

- [ ] `.env` 中 `VITE_APP_TITLE` 已改
- [ ] `index.html` 中 `<title>` 已改
- [ ] 3 处 Logo 文件已覆盖
- [ ] 登录页只剩租户+用户名+密码+按钮
- [ ] 首页只剩欢迎语
- [ ] 业务模块无导出/批量删除按钮

## 来源
提炼自该项目 C6，含 Logo 7 次迭代和登录页 3 次精简。
