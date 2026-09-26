---
name: Docker Compose Spring Boot生产部署
description: Spring Boot + MySQL + Redis + Nginx 四容器 Docker Compose 生产部署模板，含HTTPS、文件存储、数据迁移
---

# Docker Compose 生产部署模板

## 适用场景
- Spring Boot 后端 + Vue 管理前端 + MySQL + Redis
- 单机部署（1C2G 即可运行）
- 需要 HTTPS（微信小程序等强制要求）

## 1. 架构

```
Internet → Nginx(443/80) → Java(48080) → MySQL(3306) + Redis(6379)
                ↓
        Admin UI（静态文件）
        微信验证文件
        SSL 证书
```

## 2. docker-compose-prod.yml 模板

```yaml
version: '3.8'
services:
  yudao-mysql:
    image: mysql:8.4
    container_name: yudao-mysql
    environment:
      MYSQL_ROOT_PASSWORD: 'YourStr0ngP@ssword'
      MYSQL_DATABASE: ruoyi-vue-pro
    command: >
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_unicode_ci
    volumes:
      - mysql_data:/var/lib/mysql
    restart: always

  yudao-redis:
    image: redis:6-alpine
    container_name: yudao-redis
    restart: always
    # ⚠️ 不设密码 — Redisson 空密码时仍发 AUTH 命令导致报错

  yudao-server:
    image: eclipse-temurin:17-jre
    container_name: yudao-server
    working_dir: /app
    volumes:
      - ./yudao-server.jar:/app/yudao-server.jar
      - uploaded_files:/data/uploaded-files
    command: >
      java -jar yudao-server.jar
      --spring.profiles.active=prod
      --spring.datasource.dynamic.datasource.master.url=jdbc:mysql://yudao-mysql:3306/ruoyi-vue-pro?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8mb4
      --spring.datasource.dynamic.datasource.master.password=YourStr0ngP@ssword
      --spring.redis.host=yudao-redis
    depends_on: [yudao-mysql, yudao-redis]
    restart: always

  yudao-nginx:
    image: nginx:alpine
    container_name: yudao-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - ./admin-dist:/usr/share/nginx/admin
      - ./ssl:/etc/nginx/ssl
      - uploaded_files:/data/uploaded-files
    depends_on: [yudao-server]
    restart: always

volumes:
  mysql_data:
  uploaded_files:
```

## 3. 已知坑

| 坑 | 解决 |
|----|------|
| MySQL 8.4 不支持 `--default-authentication-plugin` | 删掉该参数 |
| 密码含 `#` 在 shell 被截断 | 用单引号包裹或写环境变量文件 |
| Redis 空密码触发 Redisson AUTH 报错 | 注释掉 YAML 中的 password 行 |
| Nginx `$host` 被 PowerShell 展开 | 本地写文件 → SCP 上传 |
| SQL 文件 UTF-16 编码 | `mysqldump` 在容器内执行 |
| `\r` 换行符 | `sed -i 's/\r$//' file.sql` |

## 4. 前端部署（服务器内存不足时）

```bash
# 本地构建
npm run build:prod
tar -czf admin-dist.tar.gz dist/

# 上传到服务器
scp admin-dist.tar.gz user@server:/opt/app/
ssh user@server "cd /opt/app && tar -xzf admin-dist.tar.gz && mv dist admin-dist"

# docker-compose 自动挂载 admin-dist 目录
```

## 5. HTTPS (Let's Encrypt)

```bash
# 安装 certbot
apt install certbot
certbot certonly --standalone -d your-domain.com

# 复制到挂载目录
cp /etc/letsencrypt/live/your-domain.com/fullchain.pem ./ssl/
cp /etc/letsencrypt/live/your-domain.com/privkey.pem ./ssl/

# 90天续期
crontab: 0 0 1 */2 * certbot renew && docker restart yudao-nginx
```

## 6. 文件存储（本地磁盘）

```yaml
# 应用配置
yudao.file.base-path: /data/uploaded-files

# docker-compose 中挂载同一 volume 到 nginx + java
volumes:
  - uploaded_files:/data/uploaded-files
```

> ⚠️ 部署后必须执行：`UPDATE infra_file_config SET domain = 'https://your-domain.com'`

## 来源
提炼自一个生产级抽奖营销小程序项目 C6 会话。
