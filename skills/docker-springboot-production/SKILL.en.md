---
name: docker-compose-spring-boot-production
description: A production Docker Compose template for Spring Boot + MySQL + Redis + Nginx, including HTTPS, file storage and data migration
---

# Docker Compose production template

## When this applies

- Spring Boot backend + Vue admin frontend + MySQL + Redis
- Single-host deployment (runs on 1 vCPU / 2 GB)
- HTTPS required (WeChat mini-programs and most payment callbacks insist on it)

## 1. Topology

```
Internet → Nginx(443/80) → Java(48080) → MySQL(3306) + Redis(6379)
                ↓
        Admin UI (static files)
        Platform verification files
        SSL certificates
```

## 2. `docker-compose-prod.yml`

```yaml
version: '3.8'
services:
  app-mysql:
    image: mysql:8.4
    container_name: app-mysql
    environment:
      MYSQL_ROOT_PASSWORD: 'change-me-in-an-env-file'
      MYSQL_DATABASE: your_database
    command: >
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_unicode_ci
    volumes:
      - mysql_data:/var/lib/mysql
    restart: always

  app-redis:
    image: redis:6-alpine
    container_name: app-redis
    restart: always
    # NOTE: no password on purpose — Redisson still sends AUTH when it is
    # configured with an empty password, which fails against a fresh Redis.

  app-server:
    image: eclipse-temurin:17-jre
    container_name: app-server
    working_dir: /app
    volumes:
      - ./app-server.jar:/app/app-server.jar
      - uploaded_files:/data/uploaded-files
    command: >
      java -jar app-server.jar
      --spring.profiles.active=prod
      --spring.datasource.url=jdbc:mysql://app-mysql:3306/your_database?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8mb4
      --spring.datasource.password=${DB_PASSWORD}
      --spring.redis.host=app-redis
    depends_on: [app-mysql, app-redis]
    restart: always

  app-nginx:
    image: nginx:alpine
    container_name: app-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - ./admin-dist:/usr/share/nginx/admin
      - ./ssl:/etc/nginx/ssl
      - uploaded_files:/data/uploaded-files
    depends_on: [app-server]
    restart: always

volumes:
  mysql_data:
  uploaded_files:
```

## 3. Known traps

| Trap | Fix |
|---|---|
| MySQL 8.4 rejects `--default-authentication-plugin` | Remove that flag |
| A password containing `#` gets truncated by the shell | Quote it, or keep it in an env file |
| Empty Redis password makes Redisson send AUTH and fail | Comment out the `password` line |
| Nginx `$host` gets expanded by PowerShell | Write the file locally, upload with `scp` |
| A `.sql` file in UTF-16 fails | Run `mysqldump` inside the container instead |
| CRLF line endings | `sed -i 's/\r$//' file.sql` |

## 4. Frontend deployment when the server has no memory for a build

```bash
# build locally
npm run build:prod
tar -czf admin-dist.tar.gz dist/

# ship it
scp admin-dist.tar.gz user@server:/opt/app/
ssh user@server "cd /opt/app && tar -xzf admin-dist.tar.gz && mv dist admin-dist"

# docker-compose picks the directory up through the volume mount
```

## 5. HTTPS with Let's Encrypt

```bash
apt install certbot
certbot certonly --standalone -d your-domain.com

cp /etc/letsencrypt/live/your-domain.com/fullchain.pem ./ssl/
cp /etc/letsencrypt/live/your-domain.com/privkey.pem ./ssl/

# renew every 60 days
crontab: 0 0 1 */2 * certbot renew && docker restart app-nginx
```

## 6. File storage on local disk

```yaml
app.file.base-path: /data/uploaded-files

# mount the same volume into both nginx and the JVM
volumes:
  - uploaded_files:/data/uploaded-files
```

> After deploying, run: `UPDATE infra_file_config SET domain = 'https://your-domain.com'` — otherwise generated file URLs still point at the old host.

*Extracted from a production delivery and sanitised (hostnames, container names and credentials replaced).*
