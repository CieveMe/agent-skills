---
name: obsidian-karpathy-memory
description: 用 Obsidian + Karpathy 的 LLM wiki 做法给项目建长期记忆——让 AI 持续维护一套纯 Markdown 知识库（index / hot-cache / log / 主题页 / 原始资料），而不是每次靠聊天记录。含可直接粘贴的启动提示词与"电脑要提前装什么"。适用于新项目开工、老项目补记忆、跨会话续接、交班回写。
---

# Obsidian + Karpathy 式长期记忆

核心主张只有一句：**别让项目的记忆住在聊天记录里，让它住在一套纯 Markdown 文件里，并由 AI 持续维护。**
出处是 Karpathy 那条把 LLM 当"wiki 维护者"而不是"问答机"的思路；这套文件在 Obsidian 里能读、能链接、
能搜索，同时是 Git 友好的普通文本。

## 为什么它省 token

不是"多写文档"就行了，关键是**读得少**：

- `hot-cache.md` 只放**当前上下文**（在做什么、下一步、卡在哪），保持短（建议 2500 字以内）；
- `index.md` 是目录，先读它再决定读哪页；
- 稳定事实沉到主题页/来源页，**不要**堆在 cache 里；
- 明确的"已知好路线"记在 `operations/operation-memory.md`，避免重复试错。

## 目录结构

```text
raw/                      原始资料，只增不改（文档、导出、日志、截图说明）
wiki/
  index.md                页面目录：我要靠它导航
  hot-cache.md            当前状态：在做什么 / 下一步 / 卡点（短）
  log.md                  追加式时间线：谁在哪天改了什么
  overview.md             目标、范围、约束、成功标准
  sources/                资料摘要：每份原始资料一页，带来源路径
  decisions/              决策与理由（含被否掉的方案与原因）
  operations/
    operation-memory.md   已跑通的命令/路径 + 明确不要走的路线 + 校验命令
AGENTS.md                 会话必守规则（开工顺序、回写规则、敏感信息策略）
```

## 三条让它长期不烂的规矩

1. **一页一主题。** 同一事实只在一处维护，其他地方链接过去。
2. **重要结论带来源。** 写清是"实测/文件路径/链接/用户确认"，还是"推断"。推断不能混进事实里。
3. **回写有顺序。** 状态变化 → 更新主题页 → 新页同步 `index.md` → 追加 `log.md` → 只有属于当前上下文的
   才写进 `hot-cache.md`。

## 长期跑下去会遇到的三种腐烂（以及对策）

| 症状 | 对策 |
|---|---|
| `hot-cache.md` 越写越长，最后没法读 | 稳定事实搬到主题页；cache 只留"现在"。定期跑质量检查。 |
| 两个会话各写一份，互相矛盾 | 约定**只追加自己标题的章节**，不改别人的段落；但发现对方留下的内容**已过期**要当场改掉，别让同一页自相矛盾。 |
| 结论过期了还在被引用 | 每次纠正都写进对应页（"不要用 X，用 Y，因为 Z"），并在 `log.md` 留痕。 |

## 敏感信息

密码、密钥、证件号、Token **只记"在哪里取"**，不写明文。可公开的联系方式与公开链接可以记。
一句话：知识库是给"未来的自己 + AI"读的，不是保险箱。

## 怎么开始

1. 装什么、放哪里、怎么验证 → [`references/prerequisites.md`](references/prerequisites.md)
2. 直接可粘贴的提示词（初始化 / 新会话续接 / 收尾回写 / 记操作路线）→
   [`references/bootstrap-prompts.md`](references/bootstrap-prompts.md)

如果你所在的环境已经装了通用的 `project-wiki-memory` 技能，它自带初始化与质量检查脚本
（`init_project_wiki.py` / `check_wiki_quality.py`），可以省掉手建目录；没有也不影响——**这套方法本身
只需要文件夹和 Markdown**。

