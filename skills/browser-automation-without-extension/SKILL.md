---
name: browser-automation-without-extension
description: 浏览器扩展（@chrome 类插件）不可用时，用命令行 + CDP 接管用户已登录的浏览器完成网页操作——复用登录态、填表单、选下拉、传文件、发布并回读核对。用于"扩展挂了/没有扩展但要操作网站"的场景。
---

# 无扩展情况下的浏览器操作

扩展挂了不等于浏览器操作做不了。只要浏览器**已经由你（用户）登录好**，就可以用命令行通过 CDP 接管它。
本 skill 记录的是踩过坑之后剩下的那套做法。

## 唯一正确的目标

**操作的是用户那个已登录的浏览器，不是新建的干净实例。** 新建实例没有登录态，等于什么都做不了——
而且它还会伪装成"连接成功"，这是最费时间的假象（见坑 1）。

## 核心循环

```bash
agent-browser --auto-connect tab list        # 看到用户真实标签页 ⇒ 通道正常
agent-browser --auto-connect tab <n>          # 选中目标页
agent-browser --auto-connect open <url>       # 导航
agent-browser --auto-connect snapshot -i      # 拿可交互元素
agent-browser --auto-connect click <sel>
agent-browser --auto-connect fill <sel> "文本"
agent-browser --auto-connect upload <sel> <文件...>
agent-browser --auto-connect keyboard type "文本"   # 真实按键（下拉联想必须用这个）
agent-browser --auto-connect press Enter
agent-browser --auto-connect eval '(<js>)()'        # 读值/校验/必要时直接触发点击
```

两条铁律：

1. **每次操作前重新选标签**——进程之间不保证保持选中，标签会漂移。
2. **每步都回读验证**（用 `eval` 读字段值、chip、列表项），不要相信"点了就成功"。发布/提交类动作
   完成后必须回读页面文本或 DOM 才算完成。

## 必须记住的坑

1. **不要用 `connect <port>` 做诊断。** 它会顺手启动一个全新的空白浏览器实例，之后
   `--auto-connect` 会一直连那个空实例——`tab list` 只显示一条 `about:blank`，看起来完全像"通道坏了"。
   修法：`agent-browser close` 关掉误启实例，再 `--auto-connect`，就会回到用户真实的浏览器。
2. **`os error 10060`（读不到标签页）不等于浏览器没开。** 先按上一条排查是不是被空实例接管，不要
   让用户去重装/重启浏览器。
3. **不要靠探测调试端口判断通道健康。** Chrome 即使监听 9222，`/json/version` 与 `/json/list` 也可能
   返回 404；端口在监听 ≠ 你能控制它。
4. **React 受控表单的两种世界。** 注入 `input` 事件通常能把值写进字段；但**下拉选择与对话框提交
   必须真实点击**，JS `click()` 在这类组件上经常无效。
   - MUI 自动补全：候选项是 `li.MuiAutocomplete-option`，按文本精确匹配后再点。
   - 键盘 `Type + Enter` 会选中**第一个**候选，不是精确匹配——我们因此误加过 `Python Fire`。
     正确顺序是：先 dump 候选 → 再决定点哪个；误加之后要会用 chip 的"移除"控件删掉（它可能没有
     `aria-label`，只能按外层文本 `Remove <X> Tag` 定位）。
   - 对话框里的提交按钮：全局 `button:has-text("Add")` 会匹配到多个元素，必须在对话框容器内查找。
   - **只按回车不会提交**某些对话框（例如"添加链接"），必须点它自己的 `Add`。
5. **hover 才出现的按钮**：合成的 `mouseover` 事件无效，要用真实鼠标移动
   （`mouse move <x> <y>`）之后再点。
6. **文件上传**：页面上经常有多个 `input[type=file]`，用结构定位（如
   `div.air3-modal-body > input[type=file]`），不要用裸 `input[type=file]`（会匹配到 2 个以上而报错）。
7. **链接预览卡在 `Verifying…` 通常不影响提交。** 典型原因是短链 302 跨域跳转（如 `doi.org` → 目标站）
   抓不到 OG 标签；换成目标站直连地址即可出卡片。校验一般只看"有没有内容项"，不看预览是否完成。
8. **反爬/风控。** Cloudflare / PerimeterX 的"请稍候…"交用户手动过一次，或换代理节点，**不要反复重试**。
   注意页面里显示的出口 IP 可能不是本机 IP（说明走了代理），这解释了很多"我这边没问题"的分歧。
9. **改过视口要还原。** 为了截图 `set viewport <w> <h>` 之后，记得设回接近用户原本的尺寸。
10. **页面加载超时先看是不是已经加载好了**：`page.goto: Timeout` 仍可能页面已在，直接 `eval` 读内容往往
    就能继续，不必重来。

## 操作纪律

- 只做用户要求的事；**发布 / 提交 / 付款类动作做完必须回读核对**，并把证据（URL、DOM 里的字段值、
  条目所在分栏）写回知识库。
- 需要**人脸、证件、验证码、密码、付款**的步骤交用户本人；反机器人控件不代点。
- 破坏性动作（删除、退款、清空、改权限）先确认对象是谁，宁可多一步核对。
- 平台侧规则（如"只走站内托管、绝不垫钱、绝不代收款"）属于业务底线，写进对应平台页，不靠记忆。

## 其它可以做浏览器操作的 agent / 工具

如果这套命令行方案不可用，或你想换环境，见
[`references/browser-agent-options.md`](references/browser-agent-options.md)：Playwright MCP、Chrome DevTools
MCP、browser-use、OpenHands、Stagehand、Skyvern、Browserless，以及各家的 computer-use 模型。

