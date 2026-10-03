# 电脑要提前装什么

一句话版本：**必须装的只有 Obsidian；想要历史和备份就再加 Git；其它都不是必需品。**
整套记忆就是一堆 `.md` 文件和文件夹，没有数据库、没有服务端、不需要联网。

## 必装

| 软件 | 为什么 | 备注 |
|---|---|---|
| **Obsidian**（桌面版，免费） | 把这堆 Markdown 当成"可点击链接的知识库"来读：双向链接、图谱、全文搜索、按页导航 | 个人使用免费；**不需要**买 Obsidian Sync——Git 就够备份 |

## 建议装

| 软件 | 为什么 | 备注 |
|---|---|---|
| **Git** | 版本历史 + 备份 + 回滚；知识库是纯文本，天然适合 | 在 vault 根目录 `git init` 即可；配一个远程仓库就多一层保险 |
| **一个能干活的 AI 客户端** | 这套方法的前提是"AI 持续维护" | Codex / Claude Code / Cursor / opencode 都行；能读文件、能写文件就够 |

## 按需装

| 软件 | 什么时候需要 |
|---|---|
| **Python 3** | 只有当你想跑自动化的"初始化 / 质量检查"脚本时（检查 frontmatter、断链、hot-cache 是否过长、是否还有未提交改动）。没有 Python 也能用，只是这些检查要手工做。 |
| **Obsidian 社区插件** | 默认**一个都不需要**。想要仪表盘可以装 Dataview，想要模板可以装 Templater——属于锦上添花，坏了也不影响记忆本体。 |

## 不需要（别被教程带偏）

- 数据库 / 向量库：`index.md` + 全文搜索在中小项目里比向量检索更好用，也更好核对。
- 付费同步服务：Git 或任意网盘（注意下面那条）足够。
- Obsidian 插件套装：插件越多，越容易"知识库打不开"。
- 特殊目录结构：**先能写、能读、能回写**，再谈优化。

## 放在哪里

- **放在磁盘上普通文件夹**即可，Obsidian 里"打开文件夹作为 vault"。
- **不要放在 OneDrive / 坚果云等实时同步目录里再叠加 Git**：两边同时改会产生冲突文件。二选一。
- 路径含中文没问题（Obsidian 与 Git 都能处理），但**在 shell 里跑脚本时要小心引号与编码**——这是实际
  踩过的坑：非 ASCII 路径会让某些命令的参数解析出错。

## 装完怎么验证（三步）

1. Obsidian 能打开这个文件夹为 vault，能看到 `wiki/index.md`，且里面的链接**可点击**。
2. 在 vault 根目录 `git status` 能正常返回（若用了 Git），没有莫名其妙的中文乱码文件名。
3. 让 AI 跑一次"新会话续接"提示词（`bootstrap-prompts.md` 第 2 条）：它应当**只**读 `index.md` 与
   `hot-cache.md` 就能用三行说清"在做什么 / 下一步 / 卡在哪"。做不到说明 `hot-cache.md` 没写好——
   回去把它写成"现在"，而不是写成历史。

## English version

**Must install: Obsidian (free, desktop).** It turns the Markdown files into a navigable, linkable, searchable
knowledge base. No sync subscription is required.

**Recommended: Git** (history, backup, rollback — the vault is plain text) **and one AI client** that can read
and write files (Codex, Claude Code, Cursor, opencode).

**As needed: Python 3**, only if you want the automated init / quality-check scripts. **Obsidian community
plugins: none required** (Dataview or Templater are optional conveniences).

**Not needed:** a database or vector store (an index page plus full-text search is easier to verify at small
and medium scale), a paid sync service, or a plugin pack.

**Where to put it:** an ordinary folder on disk, opened in Obsidian as a vault. Do not put it inside a
real-time cloud-sync folder *and* use Git — pick one, or you will get conflict files. Non-ASCII paths are fine
for Obsidian and Git, but be careful with quoting and encoding when running shell commands against them.

**Verify in three steps:** (1) Obsidian opens the folder and the links in `wiki/index.md` are clickable;
(2) `git status` works in the vault root without mojibake filenames; (3) the "resume in a new session" prompt
lets the AI state what we are doing, what is next and where we are stuck by reading only `index.md` and
`hot-cache.md` — if it cannot, the hot cache is written as history instead of as *now*.
