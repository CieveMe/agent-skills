# 电脑要提前装什么

一句话版本：**什么都不装也能用。** 这套记忆就是一堆 `.md` 文件和文件夹，没有数据库、没有服务端、不需要
联网；**Obsidian 只是"最好用的阅读器"，不是依赖**——它没装，记忆本体一样工作。

## 按"装多少"分四档（选一档就行）

| 档位 | 需要什么 | 你能得到 | 适合谁 |
|---|---|---|---|
| **A. 零安装** | 一个能读写文件的 AI 客户端 + 一个文件夹 | 记忆的全部功能：AI 按 `index.md`/`hot-cache.md` 读，按规矩回写 | 只想先跑起来；或机器受管控、不方便装软件 |
| **B. 轻量** | 加 **VS Code**（免费） | Markdown 预览、全文搜索（Ctrl+Shift+F）、链接可点 | 多数开发者本来就有 |
| **C. 完整体验** | 加 **Obsidian**（免费，桌面版） | 双向链接面板、图谱、标签、按页导航、手机端查看 | 想长期经营这个知识库 |
| **D. 给人看 / 异地访问** | 把 vault 推成 GitHub 仓库（公开或私有） | 网页上就能渲染 Markdown 并点链接，顺带拿到版本历史 | 要分享、要异地访问 |

**关键前提（决定换不换编辑器都不退化）**：链接一律用**标准相对 Markdown 链接**
`[标题](sources/xxx.md)`，**不要用 Obsidian 专属的 `[[wikilink]]`**。标准链接在 Obsidian、VS Code、
GitHub、Typora 里都能点开；`[[...]]` 出了 Obsidian 就只是普通文字。

## 各档的明细

| 软件 | 为什么 | 备注 |
|---|---|---|
| **Obsidian**（桌面版，免费，**可选**） | 把这堆 Markdown 当成"可点击链接的知识库"来读：双向链接、图谱、全文搜索、按页导航 | 只有 C/D 档需要；个人使用免费；**不需要**买 Obsidian Sync——Git 就够备份 |
| **VS Code**（免费，**可选**） | 有些人不想再装一个 App：VS Code 的 Markdown 预览 + 全文搜索已经够日常读 | B 档；顺带能编辑脚本 |

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

**Nothing has to be installed.** The memory is a set of `.md` files; Obsidian is the nicest *reader*, not a
dependency. Pick one of four configurations:

| Configuration | What you install | What you get |
|---|---|---|
| **A. Zero-install** | an AI client that can read/write files + a folder | the whole method: the AI reads `index.md` / `hot-cache.md` and writes back by the rules |
| **B. Light** | add **VS Code** (free) | Markdown preview, full-text search (`Ctrl+Shift+F`), clickable links |
| **C. Full** | add **Obsidian** (free, desktop) | backlinks, graph, tags, page navigation, mobile |
| **D. Shared / remote** | push the vault to GitHub (public or private) | Markdown rendered in the browser with working links, plus version history |

**The requirement that keeps this portability:** use standard relative Markdown links (`[title](sources/x.md)`),
never Obsidian-only `[[wikilinks]]`. Standard links open in Obsidian, VS Code, GitHub and Typora alike.

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
