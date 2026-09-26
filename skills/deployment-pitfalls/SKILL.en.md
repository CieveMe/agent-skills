---
name: remote-deployment-pitfalls
description: Six fatal Windows-to-Linux deployment traps learned the hard way — PowerShell syntax, encoding, file locks, and SSH quoting hell
---

# Remote deployment pitfalls — quick reference

> Source: a real deployment session (2026-04-09)
> Scenario: Windows PowerShell → SSH → commands inside a Docker container

## Trap 1: PowerShell does not support `&&`

**Symptom**
```
The token '&&' is not a valid statement separator in this version.
```

**Cause**: PowerShell 5.x does not understand the bash-style `&&`.

**Fix**: use `;` instead — but be aware `;` does *not* short-circuit: the second command runs even if the first failed.

```powershell
# Wrong
scp file server:/path && ssh server "restart"

# Right
scp file server:/path ; ssh server "restart"

# Right, with short-circuit behaviour
scp file server:/path ; if ($LASTEXITCODE -eq 0) { ssh server "restart" }
```

---

## Trap 2: non-ASCII paths make SSH/SCP fail silently

**Symptom**
```
Warning: Identity file D:/Desktop/xxx/<non-ascii-folder>/deploy-key.pem not accessible
Permission denied (publickey)
```

**Cause**: when PowerShell passes arguments to `ssh.exe` / `scp.exe`, a path containing non-ASCII characters gets mangled by the console code page.

**Fix, in order of preference**

1. Keep the key file on a **pure ASCII path** (e.g. `D:\keys\deploy-key.pem`).
2. Use a relative path when your setup allows it.
3. Copy the key to `%USERPROFILE%\.ssh\` and use a `Host` alias in `~/.ssh/config`.

Recommended header for deployment scripts:
```powershell
$PEM_KEY = "D:\keys\deploy-key.pem"  # ASCII path only
$SERVER = "root@<your-server-ip>"
```

---

## Trap 3: hot-swapping a JAR causes `ClassNotFoundException`

**Symptom**
```
java.lang.NoClassDefFoundError: ch/qos/logback/classic/spi/ThrowableProxy
```
and the web UI keeps timing out / spinning.

**Cause**: overwriting a running `*.jar` over SCP. The JVM's class loader sees a partially replaced archive mid-flight and the class-loading chain breaks.

**Correct order**
```bash
# 1. stop the container first
docker compose stop server

# 2. now overwrite the artefact
scp app-server.jar server:/opt/app/backend/target/

# 3. start it again
docker compose start server
```

One-liner version:
```bash
ssh server "cd /opt/app && docker compose stop server" ; \
scp app-server.jar server:/opt/app/backend/target/ ; \
ssh server "cd /opt/app && docker compose start server"
```

---

## Trap 4: files written on Windows carry `\r` and break on Linux

**Symptom**

- MySQL reports a `syntax error` on a statement that looks perfectly fine.
- A shell script fails with `command not found` because of a hidden `\r` at the end of the line (`#!/bin/bash\r`).

**Cause**: Windows editors default to CRLF (`\r\n`); Linux tooling expects LF (`\n`).

**Fix**: convert immediately after upload.
```bash
tr -d '\r' < input.sh > output.sh      # always available
sed -i 's/\r$//' file.sh               # in-place
dos2unix file.sh                       # if installed
```

Best practice — bake it into the deployment script:
```bash
tr -d '\r' < /opt/app/run_fix.sh > /opt/app/run_fix_unix.sh
bash /opt/app/run_fix_unix.sh
```

---

## Trap 5: `docker exec` quoting hell

**Symptom**: going PowerShell → SSH → `docker exec` → `bash -c` → `mysql`, the quotes get eaten layer by layer.
```
unexpected EOF while looking for matching `"'
```

**Cause**: four nested interpreters, each consuming one level of quoting.

**Golden rule: pass files, not commands.**
```bash
# Wrong — SQL nested inside the command line
ssh server "docker exec mysql bash -c 'mysql -uroot -p\"$PASS\" db -e \"SELECT ...\"'"

# Right — write the SQL to a file, ship the file
# 1. write fix.sql locally
scp fix.sql server:/opt/app/
ssh server "docker cp /opt/app/fix.sql app-mysql:/tmp/"

# 2. wrap the mysql call in a script
cat > run.sh << 'EOF'
#!/bin/bash
mysql -uroot -p"$MYSQL_ROOT_PASSWORD" --default-character-set=utf8mb4 dbname < /tmp/fix.sql
EOF

# 3. upload and run the script
scp run.sh server:/opt/app/
ssh server "docker cp /opt/app/run.sh app-mysql:/tmp/ && docker exec app-mysql bash /tmp/run.sh"
```

---

## Trap 6: PowerShell console encoding corrupts CJK data in the database

**Symptom**: every Chinese field in the admin UI renders as mojibake (`ä¹Œæµ·å¸‚å†…`).

**Cause**: the PowerShell console uses a legacy code page (GBK/CP936). Passing UTF-8 Chinese text through a PowerShell SSH pipe double-encodes it, and it lands in the database as ISO-8859-1.

**Fix**

1. Write the SQL to a standalone `.sql` file (UTF-8, no BOM).
2. Ship it with `scp` + `docker cp`.
3. Execute inside the container with `--default-character-set=utf8mb4`.

```bash
mysql -uroot -p"$MYSQL_ROOT_PASSWORD" --default-character-set=utf8mb4 dbname < /tmp/fix.sql
```

**Prevention**: in deployment scripts, any SQL containing non-ASCII data must go through a file — never inline it in a PowerShell command line.

---

## Pre-deployment checklist

- [ ] Is the key file path pure ASCII?
- [ ] Are PowerShell commands using `;` instead of `&&`?
- [ ] Is the container stopped *before* the JAR is replaced?
- [ ] Do uploaded files need `tr -d '\r'`?
- [ ] Does every non-ASCII SQL statement go through a file?
- [ ] Is `--default-character-set=utf8mb4` set on the MySQL call?

*Extracted from real deployment incidents and sanitised (no client names, hostnames or keys).*
