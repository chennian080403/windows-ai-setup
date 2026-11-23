# 给你的 AI：按顺序帮用户把 Windows 调到「追平 Mac 的 AI 顺手体验」

收到这个文件夹（或压缩包）后，你的任务是：**在不自己造重软件、不破坏系统安全与更新服务的前提下，按三大板块帮用户把这台 Windows 电脑调顺手**。终端 CLI 最容易做到满血贴近 Mac 体验；如果用户主要用 **Claude 桌面端、ZCode、CodeBuddy、Cursor、Codex 桌面端**，底层约 75%～80% 同样互通，先分流再按需装配。

> **执行原则**：先体检并问清使用形态（终端为主 / 桌面端为主 / 两者都用）再列清单等用户确认，改任何已有配置前先备份；遇到需要管理员权限、安装软件、改电源或系统策略时，先停下来说明原因并等用户拍板，其余安装、合并、验证全由你完成。若用户只点名做其中某一个板块，就只执行对应板块。

---

## 1. 先查清对方电脑（只查不改，查完汇总问用户确认）

| 检查项 | 怎么查 | 要确认的事 |
|---|---|---|
| **Windows 版本与机型** | `Get-CimInstance Win32_OperatingSystem \| Select-Object Caption, Version` 与 `Get-CimInstance Win32_Battery` | 区分 Home（家庭版）还是 Pro/Enterprise（专业/企业版）；区分台式机还是笔记本（Modern Standby 约束） |
| **主力使用形态与代理状态** | 问清用户常用哪类 AI，并查已安装的客户端目录及系统代理设置（`HTTP_PROXY` / `HTTPS_PROXY`） | **前置分流**：问清平时以**终端 CLI 为主**、**桌面端为主**（Claude 桌面端 / ZCode / CodeBuddy / Cursor / Codex）还是**两者都用**；**纯桌面端用户直接跳过 Windows Terminal 界面美化与粗体标黄**，只配底层壳、UTF-8、MCP、代理与全局规则 |
| **当前 Shell 与终端** | 查 `C:\Program Files\PowerShell\7\pwsh.exe`、`C:\Program Files\Git\bin\bash.exe`、`wt -v`、`$env:PYTHONUTF8` | 查清用户主力 AI 工具（Claude Code / Codex / ZCode / CodeBuddy / Cursor / Pi 等）实际调用的是哪个 Shell |
| **Node / winget / Git** | `node -v`、`winget --version`、`git --version` | 决定能否用 `npm` 装 `agent-desktop`，以及用 `winget` 装工具 |
| **现有电源与常用工具** | `powercfg /getactivescheme`、`powercfg /query`（记录 AC 原值）、查是否已装 `PowerToys` / `Everything` | 记录原设置以便回滚，问清用户要装全套 3 板块还是只做其中几块 |

---

## 2. 核心原理（不能踩的 5 条红线）

1. **绝不删除系统自带的 PowerShell 5.1**：它是系统组件且 pwsh 7 兼容层需要它；靠 `pwsh 7` 装在默认路径 `C:\Program Files\PowerShell\7\pwsh.exe` + 系统 `PATH` 置顶来压制它。
2. **绝不要求用户关掉 Windows Defender 或智能应用控制（SAC）**：若 `agent-desktop` 被 SAC 拦截，立刻切微软官方签名的 `winapp ui`，不要劝人关安全保护。
3. **绝不暴力禁用 Windows Update 服务（`wuauserv`）**：家庭版靠「18 小时使用时段 + 最长 5 周暂停更新」，专业版才可选组策略/官方策略键。
4. **绝不让 AI 偷偷抢真实鼠标**：操作桌面先命令行/浏览器 CDP，必须点桌面软件只走后台无障碍语义动作；遇到最小化、锁屏、UAC 弹窗、拖拽或画布点击必须停下来问人。
5. **不展开本地大模型折腾**：大众 Windows 显卡多不适合跑大模型，直接调云端 AI；若用户确实装了 Ollama，只帮设好 `OLLAMA_CONTEXT_LENGTH=8192` 防爆显存即可。

---

## 3. 总流程与三板块路由（详细步骤与胶水排错表都在 `3-分板块参考/`）

按以下顺序读取 `3-分板块参考/` 中对应文档并执行（每做完一块就按该板块验收清单实测）：

| 顺序 | 对应板块文档 | 核心要落地的东西 |
|---|---|---|
| **第一步（桌面不抢鼠标）** | `3-分板块参考/01-板块1-无障碍不抢鼠标.md` | 装 `agent-desktop`（被 SAC 拦则切微软 `winapp ui`）；用记事本做一次后台无障碍读写测试；将「L1→L2→L3 不抢鼠标铁律」写入全局 AI 规则 |
| **第二步（终端与桌面端底层满血）** | `3-分板块参考/02-板块2-终端命令行与MCP防坑.md` | **全员必做**：装默认路径 PowerShell 7 + Git for Windows；设用户级 `PYTHONUTF8=1` 与 `$PROFILE`；体检 MCP 与桌面端代理继承（避开裸 `npx`、单反斜杠、Store 版假路径、ZCode 代理留空陷阱、`/mnt/c` 跨盘掉速）；合入「Windows AI 执行防打架铁律」到对应 CLI/桌面端规则位置。<br>**仅终端用户/双修用户做（纯桌面端跳过）**：装 Windows Terminal 并配置 **`intenseTextStyle: "all"` + `AI Focus Dark` 配色（`foreground == white` + `brightWhite: "#FFB86C"`）** 实现 AI 粗体标橙黄 |
| **第三步（过夜挂机与体验补齐）** | `3-分板块参考/03-板块3-长开机挂机与系统体验补齐.md` | 装 `Microsoft.PowerToys`（仅开 `Awake`、`Command Palette`、`Peek`、`Text Extractor`、`Always On Top`、`Advanced Paste` 6 个核心模块）与可选 `voidtools.Everything`；设插电 AC 不睡眠不休眠 + 10 分钟息屏（绝不动 DC 电池策略，笔记本警告严禁合盖塞包）；按 Home/Pro 配防半夜重启；提醒开启 `Win+V` 等原生 4 神技 |

---

## 4. 关键格式与实现细节（跨电脑必踩的坑）

1. **Windows Terminal 粗体标橙黄条件（必须同时满足）**：
   - `profiles.defaults` 中 `"intenseTextStyle": "all"`（默认 `"bright"` 只提亮不加粗）；
   - 自定义方案 `"AI Focus Dark"` 中 **`"foreground"` 必须严格等于 `"white"`**（例如同为 `"#DCDFE4"`），才能把默认前景色加粗 `\x1b[1m` 自动映射到 `"brightWhite": "#FFB86C"`；
   - 合并 `settings.json` 前必须先备份，且确保写入后能通过标准 JSON 校验（`Get-Content '<settings.json>' -Raw | ConvertFrom-Json | Out-Null`），不要写入 `//` 注释；低于 WT 1.21 版本跳过 `autoMarkPrompts` 与 `showMarksOnScrollbar`；
   - 并向用户说明边界：若某程序显式输出 ANSI 亮白字也会变成橙黄，不习惯可把 `brightWhite` 改回 `#FFFFFF`；若某 CLI 自己硬编码 RGB 颜色则不覆盖。
2. **PowerShell `@ref` 传参防坑**：调用 `agent-desktop` 带 `@s1:e8` 的引用编号时，在 PowerShell 里必须加单引号 `'@s1:e8'`，否则 `@` 会被当成 splatting 语法报错。
3. **环境变量与编码双兜底**：因为很多 AI 工具启动子进程带 `-NoProfile` 会跳过 `$PROFILE`，**必须**在用户级环境变量设好 `PYTHONUTF8=1` 和 `PYTHONIOENCODING=utf-8`。
4. **PowerToys `Peek` 与 `QuickLook` 二选一**：PowerToys 0.95+ 的 `Peek` 默认快捷键就是 `Space`，和 `QuickLook` 同时开会抢空格键，装了新版 PowerToys 就别再额外装 `QuickLook`。
5. **防循环熔断**：同一条命令报错出现第 2 次必须停下来读全文换方法，严禁原样试第 3 次，严禁在 PowerShell 和 bash 写法之间盲猜试运气。

---

## 5. 落地验收清单（全部跑完后再汇报给用户）

1. **板块 1 验收**：在未移动物理鼠标、未夺走当前焦点的前提下，完成一次记事本后台读写测试，并说明装的是 `agent-desktop` 还是 `winapp ui`。
2. **板块 2 验收（按使用形态分流）**：
   - **全员必验**：新开 `pwsh` 核对 `$PSVersionTable.PSVersion`（≥7.4）、`[Console]::OutputEncoding.WebName`（`utf-8`）；汇报 MCP、桌面端内置壳/全局规则与代理体检结果（如改过桌面端环境变量，提醒彻底退出托盘图标再重开生效）。
   - **仅终端/双修用户额外验收（纯桌面端跳过）**：在 Windows Terminal 打印一行粗体测试：
     `Write-Host "普通白字 $([char]27)[1m这是AI粗体标黄测试$([char]27)[0m 普通白字"`
     让用户亲眼确认粗体已变粗且标成橙黄。
3. **板块 3 验收**：运行 `powercfg /query` 确认 AC 睡眠/休眠为 `0`、息屏为 `10` 分钟，列出已记录的**原电源参数及一键恢复命令**（`powercfg -restoredefaultschemes` 等），确认 PowerToys / Everything 可正常打开。
4. **排错兜底**：任何步骤报错，先查对应 `3-分板块参考/*.md` 文末的「胶水速查表」定位根因再修。
