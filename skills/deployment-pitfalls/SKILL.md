---
name: 远程部署踩坑速查手册
description: 从实战中提炼的Windows本地→Linux服务器部署6大致命坑，含PowerShell语法、编码、文件锁、SSH引号地狱等
---

# 远程部署踩坑速查手册

> 来源：该项目 C8 会话（2026-04-09）
> 场景：Windows PowerShell → SSH → Docker容器内操作

## 坑1：PowerShell 不支持 `&&`

**现象**：
```
The token '&&' is not a valid statement separator in this version.
```

**原因**：PowerShell 5.x 不识别 bash 风格的 `&&`。

**解决**：用 `;` 替代（注意：`;` 不会短路，前一条失败后一条仍执行）。
```powershell
# ❌ 错误
scp file server:/path && ssh server "restart"

# ✅ 正确
scp file server:/path ; ssh server "restart"
```

**如需短路逻辑**：
```powershell
scp file server:/path ; if ($LASTEXITCODE -eq 0) { ssh server "restart" }
```

---

## 坑2：中文路径导致 SSH/SCP 静默失败

**现象**：
```
Warning: Identity file D:/Desktop/xxx/某中文目录/deploy-key.pem not accessible
Permission denied (publickey)
```

**原因**：PowerShell 在向 `ssh.exe`/`scp.exe` 传参时，中文路径被 GBK 编码截断。

**解决方案**：
1. **优选**：PEM 密钥文件存放在**纯英文路径**（如 `D:\keys\deploy-key.pem`）
2. **次选**：用相对路径 `..\..\your-folder\deploy-key.pem`（某些场景可行）
3. **终选**：把密钥复制到 `%USERPROFILE%\.ssh\` 并在 `~/.ssh/config` 中配置 Host 别名

**推荐加入 deploy 脚本头部**：
```powershell
$PEM_KEY = "D:\keys\deploy-key.pem"  # 纯英文路径
$SERVER = "root@<your-server-ip>"
```

---

## 坑3：热替换 JAR 触发 JVM ClassNotFoundException

**现象**：
```
java.lang.NoClassDefFoundError: ch/qos/logback/classic/spi/ThrowableProxy
```
前端页面持续超时/转圈。

**原因**：通过 SCP 直接覆盖正在运行中的 `yudao-server.jar`，JVM 的类加载器在运行时发现 JAR 文件被部分替换，导致类加载链路断裂。

**正确流程**：
```bash
# 1. 先停容器
docker compose stop server

# 2. 再覆盖文件
scp yudao-server.jar server:/opt/lottery-java/.../target/

# 3. 最后启动
docker compose start server
```

**或者一行流（推荐）**：
```bash
ssh server "cd /opt/lottery-java && docker compose stop server" ; \
scp jar server:/path/ ; \
ssh server "cd /opt/lottery-java && docker compose start server"
```

---

## 坑4：Windows 写入的 SQL/Shell 文件含 `\r` 导致执行失败

**现象**：
- MySQL 报 `syntax error` 但 SQL 语句看起来没问题
- Shell 脚本报 `command not found`（行尾有隐藏的 `\r`）

**原因**：Windows 编辑器默认用 CRLF（`\r\n`），Linux 只认 LF（`\n`）。

**解决**：上传到服务器后立即转换：
```bash
# 方法一：tr（所有 Linux 都有）
tr -d '\r' < input.sh > output.sh

# 方法二：sed
sed -i 's/\r$//' file.sh

# 方法三：dos2unix（需安装）
dos2unix file.sh
```

**最佳实践**：在 deploy 脚本模板中固定加一行：
```bash
tr -d '\r' < /opt/lottery-java/run_fix.sh > /opt/lottery-java/run_fix_unix.sh
bash /opt/lottery-java/run_fix_unix.sh
```

---

## 坑5：Docker exec 嵌套引号地狱

**现象**：通过 PowerShell → SSH → docker exec → bash -c → mysql 执行 SQL 时，引号被多层解析吞噬。

```
unexpected EOF while looking for matching `"'
```

**原因**：4 层嵌套（PowerShell → SSH → docker exec → bash），每层都会解析并吞掉一层引号。

**解决方案（黄金法则：文件传参，不传命令）**：
```bash
# ❌ 错误：试图在命令行中嵌套 SQL
ssh server "docker exec mysql bash -c 'mysql -uroot -p\"$PASS\" db -e \"SELECT ...\"'"

# ✅ 正确：先把 SQL 写成文件，传进容器执行
# Step 1: 本地写 .sql 文件
# Step 2: SCP 上传到服务器
scp fix.sql server:/opt/app/

# Step 3: 拷贝进容器
ssh server "docker cp /opt/app/fix.sql yudao-mysql:/tmp/"

# Step 4: 用 shell 脚本包装 mysql 命令
cat > run.sh << 'EOF'
#!/bin/bash
mysql -uroot -p"$MYSQL_ROOT_PASSWORD" --default-character-set=utf8mb4 dbname < /tmp/fix.sql
EOF

# Step 5: 上传并执行脚本
scp run.sh server:/opt/app/
ssh server "docker cp /opt/app/run.sh yudao-mysql:/tmp/ && docker exec yudao-mysql bash /tmp/run.sh"
```

---

## 坑6：PowerShell 编码污染数据库中文数据

**现象**：管理后台弹窗中所有中文字段显示为 `ä¹Œæµ·å¸‚å†…` 类乱码。

**原因**：PowerShell 控制台默认使用 GBK (CP936) 编码。当通过 PowerShell 的 SSH 管道执行包含中文的 SQL 时，UTF-8 中文被二次编码为 ISO-8859-1 存入数据库。

**解决**：
1. 将 SQL 写成**独立 .sql 文件**（编辑器保存为 UTF-8 无 BOM）
2. 通过 `SCP + docker cp` 直接传入 MySQL 容器
3. 在容器内用 `--default-character-set=utf8mb4` 参数执行

```bash
mysql -uroot -p"$MYSQL_ROOT_PASSWORD" --default-character-set=utf8mb4 dbname < /tmp/fix.sql
```

**预防**：deploy 脚本中所有含中文的 SQL 操作，**严禁**在 PowerShell 命令行直接拼接，必须走文件。

---

## 快速参考 Checklist

部署前过一遍这个清单：

- [ ] PEM 密钥路径是否纯英文？
- [ ] PowerShell 命令是否用 `;` 而非 `&&`？
- [ ] 是否先停容器再覆盖 JAR？
- [ ] 上传的文件是否需要 `tr -d '\r'` 去 Windows 换行？
- [ ] 含中文的 SQL 是否走文件传参而非命令行拼接？
- [ ] MySQL 执行时是否加了 `--default-character-set=utf8mb4`？

## 来源
提炼自该项目 C8 部署实战。
