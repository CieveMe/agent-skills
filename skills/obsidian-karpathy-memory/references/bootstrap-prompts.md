# 可直接粘贴的提示词

四条按用途分开。**第一条是开工用的**，其余三条是日常用的。

---

## 1. 初始化：给项目建长期记忆

```
我要给这个项目建立长期记忆，用 Obsidian + Karpathy LLM wiki 的做法。请按下面执行：

1) 在项目根目录创建：
   raw/                                  原始资料，只增不改
   wiki/index.md                         页面目录（我靠它导航）
   wiki/hot-cache.md                     当前状态（严格控制在 2500 字内）
   wiki/log.md                           追加式时间线
   wiki/overview.md                      目标 / 范围 / 约束 / 成功标准
   wiki/sources/                         资料摘要，每份来源一页并注明路径
   wiki/decisions/                       决策与理由（含被否掉的方案）
   wiki/operations/operation-memory.md   已跑通的命令与路径 + 不要走的路线 + 校验命令
   AGENTS.md                             每次会话必守的规则

2) 在 AGENTS.md 里写清三件事：
   - 开工顺序：先读 wiki/index.md；涉及当前状态再读 wiki/hot-cache.md；只在需要时读对应主题页；
     不要全量扫描 wiki/ 和 raw/。
   - 回写规则：状态变化 / 踩坑 / 用户纠正 → 更新主题页 → 新页同步 index.md → 追加 log.md；
     只有属于当前上下文的才进 hot-cache，并保持它短。
   - 敏感信息：密码、密钥、证件号、Token 只记"在哪里取"，不写明文。

3) 把这几条写进 overview.md：
   一个页面只讲一个主题；重要结论必须带来源（文件路径 / 链接 / 实测 / 用户确认 / 推断）；
   同一事实只在一处维护，其他地方链接过去。

4) 现在拿我接下来要说的第一件事做一次完整演示：资料进 raw/、摘要进 wiki/sources/、结论进主题页，
   更新 index.md 与 log.md，并让 hot-cache.md 里明确写着"当前在做什么 / 下一步 / 卡在哪"。

演示完，把你创建的目录树和 AGENTS.md 全文贴给我看。
```

---

## 2. 新会话续接：接上状态

```
先读 wiki/index.md 与 wiki/hot-cache.md 接上状态，再读与我这次任务相关的主题页；
不要全量扫描 wiki/ 和 raw/。

读完先用三行回答我：① 当前在做什么 ② 下一步是什么 ③ 卡在哪（没卡就说没有）。
然后等我给任务。
```

---

## 3. 收尾回写：把本轮成果沉淀下来

```
收尾，把本轮的东西写回知识库：

1) 结论/状态变化 → 写进对应主题页（不要新建重复页；已有页就更新它）
2) 新页 → 同步 wiki/index.md
3) 追加一条 wiki/log.md（日期 + 改了什么 + 依据）
4) wiki/hot-cache.md 只留"当前上下文"，多余的稳定事实搬到主题页，保持精简
5) 需要我本人做或决定的事，列成清单放进 hot-cache 的 Next Checks

写完后把你改动的文件清单发我。
```

---

## 4. 记录操作路线（避免下次重复踩坑）

```
把这次跑通的命令与路径，还有失败过的路线，记到 wiki/operations/operation-memory.md：

- 已跑通的路线：完整命令（可直接照抄）、预期输出、注意事项
- 明确不要走的路线：为什么失败、失败现象长什么样
- 校验命令：怎么确认这次是成功的

只记能改变下次决策的内容，不要把整段日志粘进去。
```

---

## English versions

### 1. Initialize

```
Give this project long-term memory, the Obsidian + Karpathy LLM-wiki way. Do this:

1) Create at the project root:
   raw/                                  original material, append-only
   wiki/index.md                         the page directory I navigate by
   wiki/hot-cache.md                     current context (keep under ~2500 characters)
   wiki/log.md                           append-only timeline
   wiki/overview.md                      goal / scope / constraints / success criteria
   wiki/sources/                         one summary page per source, with its path
   wiki/decisions/                       decisions and why (including rejected options)
   wiki/operations/operation-memory.md   working commands and paths + routes not to take + validation
   AGENTS.md                             rules every session must follow

2) In AGENTS.md, state three things:
   - Reading order: read wiki/index.md first; read wiki/hot-cache.md when current state matters;
     read only the relevant topic pages; never scan all of wiki/ or raw/.
   - Write-back: change / pitfall / user correction -> update the topic page -> link new pages from
     index.md -> append to log.md -> put only what is current into hot-cache and keep it short.
   - Sensitive material: passwords, keys, ID numbers and tokens are recorded as where to get them,
     never in plaintext.

3) Put these in overview.md: one topic per page; every important conclusion carries its source
   (file path / link / measured / user-confirmed / inference); a fact is maintained in one place.

4) Demonstrate the whole loop on the first thing I give you: material into raw/, a summary into
   wiki/sources/, conclusions into a topic page, then update index.md and log.md, and make
   hot-cache.md say what we are doing, what is next, and where we are stuck.

When done, show me the directory tree and the full AGENTS.md.
```

### 2. Resume in a new session

```
Read wiki/index.md and wiki/hot-cache.md to pick up state, then only the topic pages relevant to my
task; do not scan all of wiki/ or raw/. Answer in three lines: what we are doing, what is next,
and where we are stuck (say "nothing" if we are not). Then wait for my task.
```

### 3. Write back

```
Wrap up and write this session into the knowledge base:
1) conclusions/state changes -> the relevant topic page (update it; do not create a duplicate)
2) new pages -> link them from wiki/index.md
3) append one entry to wiki/log.md (date + what changed + the basis)
4) keep wiki/hot-cache.md about the current context only, and move stable facts out of it
5) list anything that needs me personally in the hot-cache Next Checks
Then send me the list of files you changed.
```

### 4. Record an operation route

```
Record this session's working commands and paths, plus the routes that failed, into
wiki/operations/operation-memory.md: the working route (copy-pasteable command, expected output,
caveats), the routes not to take (why, and what the failure looks like), and a validation command.
Keep only what would change a future decision; do not paste whole logs.
```

