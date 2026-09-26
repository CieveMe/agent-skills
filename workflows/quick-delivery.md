---
description: 基于现有抽奖模板快速交付新客户项目的完整流程
---

# 抽奖营销项目快速交付流程

// turbo-all

## 前置条件
- 已有上一版项目源码（backend_java/ + frontend/）
- 新客户已提供：公司名、Logo、微信支付商户号、服务器IP

## 第一步：复制项目骨架（30分钟）

1. 复制整个项目目录，重命名
2. 全局替换以下关键词：
   - `上一版项目名` → 新客户名
   - `old-domain.example` → 新域名
   - `wx0000000000000000` → 新 AppID
   - `0000000000` → 新商户号
3. 替换 Logo 文件（3处）：
   - `src/assets/imgs/logo.png`
   - `src/assets/imgs/logo.gif`
   - `src/assets/imgs/avatar.gif`

## 第二步：微信支付配置（1小时）

1. 将客户证书放入 `resources/cert/`
2. 提取证书序列号：`openssl x509 -in apiclient_cert.pem -noout -serial`
3. 修改 `application-prod.yaml` 中微信支付配置
4. 修改 `system_social_client` 表中 AppID/AppSecret
5. **检查 `mock-enable: false`**

## 第三步：服务器部署（2小时）

1. SSH 到服务器
2. 上传 `docker-compose-prod.yml` + `nginx.conf`
3. 修改 MySQL 密码、域名
4. `docker compose up -d`
5. 配置 SSL：`certbot certonly --standalone -d 新域名`
6. 上传前端构建产物
7. 修改 `infra_file_config.domain` 为生产域名

## 第四步：验收测试（1小时）

参照 总结.md 第十章 25个测试用例执行：
- [ ] 活动创建（时间不重叠）
- [ ] 奖品配置（概率=100%）
- [ ] 支付下单→回调→抽奖
- [ ] 领奖填信息
- [ ] 积分兑换
- [ ] 管理后台 CRUD

## 第五步：交付文档（已有模板）

复制并修改以下文档（在 总结.md 第十八章列出）：
- user_manual.md
- deployment_knowledge.md
- monitoring_troubleshooting.md

## 预计总耗时：半天（4-5小时）
