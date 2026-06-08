---
name: project-rule-chain-init
description: "为任意软件项目初始化 Agent 规则链路。创建或合并 `.agent-rules/` 四层目录（rules/mechanisms/dev-plans/handoffs）和 Claude、Codex 等 Agent 的桥接入口，让后续任务能按固定链路读取项目规则、机制文档、计划、交接。不要用它编写业务规则、梳理机制、维护开发计划、处理交接、修改业务代码或执行除规则链路初始化以外的工作。"
---

# Agent 项目规则链路初始化

本技能面向任意软件项目，唯一能力是**初始化 Agent 规则链路**：铺设或合并项目根目录下的 `.agent-rules/` 目录和宿主 Agent 桥接入口。

不要用本技能编写业务规则、梳理机制、维护开发计划、处理交接、修改业务代码、运行构建测试，或执行任何规则链路初始化以外的工作。

> **TL;DR** — 本技能只初始化项目规则链路骨架。
> `.agent-rules/` 负责承载项目规则、机制、计划和交接。
> Agent 先通过本链路知道该读什么，再按链路定位具体文档。

## 核心模型

```
宿主工具桥接入口（默认只生成以下两个）
  ├─ AGENTS.md        (Codex / 通用 Agent 桥接入口，简短，只指向 README)
  └─ CLAUDE.md        (Claude 桥接入口，简短，只指向 README)
        └──► .agent-rules/README.md   (项目规则总纲 + 机制索引 + 读取链路)
                ├─ rules/        项目执行规则（必须/不要/优先）
                ├─ mechanisms/   项目稳定机制（系统如何协作 / 接口 / 字段 / 风险）
                ├─ dev-plans/    开发计划
                └─ handoffs/     跨会话接力摘要（做到哪 / 未提交什么 / 下步怎么接）

额外 Agent 桥接入口（按需，由 Agent 自行决定文件名和内容）
  └─ <agent-decided>.md  (仅当 AGENTS.md / CLAUDE.md 不覆盖时，由当前 Agent 自建)
```

额外桥接入口创建规则：
- 初始化时告诉当前 Agent：如果你一定会读的入口文件名不是 `AGENTS.md` 也不是 `CLAUDE.md`，请自行创建对应的桥接文件，文件名和内容由你决定，但首条必须指向 `.agent-rules/README.md`。
- 如果 `AGENTS.md` 或 `CLAUDE.md` 已经覆盖，不额外创建。

五个标准不变量，初始化与后续维护都必须守住：

1. **适用范围**：本技能适用于任意需要 Agent 协作的软件项目。若目标项目已有成熟的规则体系，按步骤 0 的冲突检查处理，尊重已有规则，不覆盖。
2. **能力边界**：本技能只初始化规则链路骨架，不负责编写规则、机制、计划或交接内容，也不修改业务代码。
3. **统筹真来源**：项目级规则、机制梳理、开发计划和接力摘要只存活在根 `.agent-rules/`。桥接文件只指路，不复制规则正文。
4. **固定读取链路**：任何任务开工前先读桥接入口 -> 根 `.agent-rules/README.md` -> `rules/README.md` 索引，按任务定位相关规则文件再精读 -> 按任务读 `mechanisms/` / `dev-plans/` / `handoffs/`。
5. **优先级**：系统/宿主工具指令 > 用户当前要求 > 根 `.agent-rules/rules/` > 根 `.agent-rules/mechanisms/` > 桥接文件摘要 > 其他项目文档。
6. **分层判据**：
   - "必须/不要/优先"这类执行约束 → `rules/`
   - "系统现在如何工作、接口如何协作、字段如何解释" → `mechanisms/`
   - "本期改哪些文件、按什么步骤验收、进度如何" → `dev-plans/`
   - "上一轮做到哪、哪些文件未提交、下一轮怎么接" → `handoffs/`

## 执行步骤

### 步骤 0：确认目标项目与冲突检查

1. 确认目标项目根目录（默认当前工作目录）。非软件项目目录时提醒用户确认。
2. **总览问答**：向用户确认以下配置（用一句话汇总提问，不要逐项单独问）：
   - **项目名称**：检测到目录名为 `{dir}`，用这个作为项目名称吗？（默认取目录名，用户可改）
   - **桥接入口**：你使用哪些 Agent？选项：① Claude Code（生成 CLAUDE.md）② Codex / 其他（生成 AGENTS.md）③ 两个都生成（默认）
   - 将用户选择记为变量，后续步骤 2 按选择生成。
3. 检查是否已存在根 `.agent-rules/` 和目标宿主 Agent 使用的桥接文件（如 `AGENTS.md`、`CLAUDE.md`）：
   - **不存在** → 直接铺设模板。
   - **已存在但结构不完整**（缺少 `rules/`、`mechanisms/`、`dev-plans/`、`handoffs/` 中任意一个子目录或其 README）→ 告知用户"检测到已有 `.agent-rules/`，但标准四层目录不完整"，汇报哪些目录/文件缺失，用户选择：
     - **补齐**（推荐）：只创建缺失的目录和文件，不动已有内容。
     - **跳过**：保持现状，不补充。
   - **已存在且结构完整** → 不要覆盖。先读出现有内容，向用户汇报差异，按用户选择"合并 / 重命名备份 / 跳过"。绝不静默覆盖用户已有规则。
   - **`CLAUDE.md` 已存在时特别处理**：告知用户"我们的 CLAUDE.md 包含两部分：① 项目规则加载入口（指向 `.agent-rules/` 读取链路）② 通用行为准则（写代码前先思考、简单优先、手术式修改、目标驱动执行）。如果保留你的 CLAUDE.md，这些行为准则不会生效。"让用户选择：
     - **合并**：把我们的"项目规则加载"段合并到现有 CLAUDE.md 中（推荐）。
     - **覆盖**：用我们的 CLAUDE.md 替换现有文件。
     - **跳过**：保留现有 CLAUDE.md，不修改。
4. **Git 工具检测**：检测项目使用的 Git 远端类型：
   - 检查 `git remote -v`，根据远端域名判断平台：
     - **GitHub**（`github.com`）→ 推荐安装 `gh`（GitHub CLI），说明其 PR/Issue/CI 管理能力。若用户选择安装 → 安装后执行 `gh auth login` 引导认证，记录 `gh` 状态用于填充 `rules/git-operations.md`。
     - **GitLab**（`gitlab.com` 或自建域名）→ 推荐安装 `glab`（GitLab CLI），说明其 MR/Issue/流水线管理能力。若用户选择安装 → 安装后收集 GitLab 配置信息（实例地址、用户名、协议偏好），执行 `glab auth login` 引导认证，并在 `rules/git-operations.md` 中替换 glab 相关占位符。**自建 GitLab** 需额外确认域名和 Personal Access Token。
     - **其他平台**或**无远端** → 仅保留通用 Git 约定（约定式提交、中文提交），工具占位符按「未配置」状态填充。
   - 用户可同时选择 `gh` 和 `glab`（项目同时使用 GitHub 和 GitLab 时），也可只选一个或都不装。

### 步骤 1：铺设 `.agent-rules/` 骨架

把 `templates/agent-rules/` 整棵目录复制到项目根的 `.agent-rules/`：

- `README.md` — 项目规则总纲（含读取链路、优先级、分层判据、维护规则）。
- `rules/README.md` — 项目规则层说明 + 规则文件清单占位。
- `rules/git-operations.md` — Git 操作规则（预置，替换 `{{GIT_PLATFORM}}` 及 glab 相关占位符）。
- `mechanisms/README.md` — 机制层说明 + 机制文档清单占位（含七段强制结构约定）。
- `dev-plans/README.md` — 开发计划层说明（命名约定、结构约定、归档流程、在册清单占位）。
- `handoffs/README.md` — 交接层说明 + 命名约定。
- `dev-plans/archived/.gitkeep` — 占位，保留归档目录。

替换 `{{PROJECT_NAME}}` 占位符。`rules/git-operations.md` 中的占位符在步骤 0 收集到信息后替换。四层目录只放 README 索引和预置规则，**不预置机制/计划/交接条目**。这些要靠后续真实工作沉淀，避免一开始就塞进没人验证过的规则。

### 步骤 2：落桥接入口

根据步骤 0 用户确认的选择生成桥接文件：

- **用户选了 Codex / 其他**：复制 `templates/bridge/AGENTS.md` -> 项目根 `AGENTS.md`。
- **用户选了 Claude Code**：复制 `templates/bridge/CLAUDE.md` -> 项目根 `CLAUDE.md`。
- **用户选了两个都生成**（默认）：两个都复制。

两个桥接模板本身不含 `{{PROJECT_NAME}}` 等占位符，复制后无需替换。所有桥接文件都必须保持简短，首条指向 `.agent-rules/README.md`，不复制四层正文。

> **额外桥接入口（按需）**：初始化时告诉当前 Agent —— 如果你一定会读的入口文件名不是 `AGENTS.md` 也不是 `CLAUDE.md`，请自行创建对应的桥接文件，文件名和内容由你决定，但首条必须指向 `.agent-rules/README.md`。如果 `AGENTS.md` 或 `CLAUDE.md` 已经覆盖，不额外创建。
>
> 注意：`templates/bridge/CLAUDE.md` 只含"规则入口摘要"段。若目标项目已有一份带通用行为准则的 CLAUDE.md，应把入口摘要段**合并**进去，而不是整文件覆盖。

### 步骤 3：自检

铺完后逐项核对：

- [ ] `.agent-rules/README.md` 存在，且四层目录均建好。
- [ ] `rules/ mechanisms/ dev-plans/ handoffs/` 各有 README。
- [ ] 已生成或合并的所有桥接入口首条都指向 `.agent-rules/README.md`，表述一致。
- [ ] 优先级链在 README 与所有桥接入口中一致。
- [ ] 没有残留 `{{...}}` 占位符（除模板里明确保留的示例占位清单）。
- [ ] 没有覆盖用户原有的任何规则文件（如有冲突，已按步骤 0 处理）。

### 步骤 4：向用户交付

用一段话说明：建了什么、读取链路是什么、接下来怎么往四层目录里填真实规则/机制/计划，以及后续 Agent 完成分析或梳理后应更新机制文档并通过索引定位文档。
**不要主动跑构建或提交**。是否提交由用户决定。

## 维护提示（交付给用户的长期约定）

以下是初始化产物中的长期约定，不代表本技能能代替用户完成后续维护：

- 新的项目级长期执行约束 → 根 `rules/`，并在 `rules/README.md` 索引追加一行。
- 复杂跨模块功能实施时强制生成机制文档 → 当实现涉及多模块调用、非直链交互、连锁状态变更或公共契约修改时，必须遵循 `rules/complex-feature-mechanism.md` 的触发条件与执行方式，在实施前/中/后同步维护机制文档，防止后期链路断裂。
- Agent 完成分析、梳理、排障复盘、架构理解或项目链路确认后，若得到可复用稳定事实 → 按 `mechanisms/README.md` 七段强制结构新增或更新 `mechanisms/` 下的机制文档。七段为：结论、涉及对象（表格式）、运行链路（ASCII 图）、使用点、修改点（表格式）、关键约束（≤6 条）、维护方式。禁止写入大段代码、方案论证、DDL、长篇背景或超 150 行。
- 新的稳定机制梳理 → `mechanisms/`，按七段强制结构编写，并在 `mechanisms/README.md` 与根 `.agent-rules/README.md` 索引追加一行。
- 新的开发计划 → `dev-plans/`，顶部链接配套机制文档，并在 `dev-plans/README.md` 在册清单追加。
- 跨会话接力摘要 → `handoffs/`，命名 `YYYY-MM-DD-主题-handoff[-agent].md`。
- 桥接文件永远保持简短，只指向 `.agent-rules/`；不同 Agent 只允许入口文件名不同，不允许维护多套规则正文。
- 文档很多时先读目录 `README.md` 索引，用关键词定位后再精读；不要全量读取，也不要逐个文件盲找。
- 单目录文档过多时按业务域、子系统或能力主题拆分子目录，每个子目录保留自己的 `README.md` 索引。
- 索引与实际文件、计划顶部的配套机制链接，是最容易漂移的两处，定期核对。

## 与 lcbox-rule-chain-init 的关系

本技能是 `lcbox-rule-chain-init` 的通用化版本，差异如下：

| | lcbox-rule-chain-init | project-rule-chain-init |
|---|---|---|
| 适用范围 | 仅 lcbox 系列工作区 | 任意软件项目 |
| 项目模型 | 工作区含多个子项目 | 单项目 |
| 子项目规则 | 读取并优先遵守 | 不适用 |
| glab | 必装 | 可选（按远端类型推荐） |
| 产品边界检查 | 有 | 无 |

两个技能独立维护，互不依赖。lcbox 系列工作区继续使用 `lcbox-rule-chain-init`；其他项目使用本技能。

## 技能自身安装

当用户要求"安装 project-rule-chain-init 技能"时，使用全局软链接安装，不复制文件：

### 软链接拓扑

```
~/.claude/skills/project-rule-chain-init  →  ~/.agents/skills/project-rule-chain-init  →  <本仓库根目录>
```

- **~/.claude/skills/** — Claude Code 的技能扫描目录（必须装到这里 Claude 才能识别）。
- **~/.agents/skills/** — 其他 Agent（Codex、OpenClaw 等）的共享技能目录。
- `.claude` 下的链接指向 `.agents`，`.agents` 下的链接指向仓库源码真身。
- 双跳设计确保各 Agent 各自扫描到同一份源，且改源码即时生效，无需重装。

### 安装步骤

1. **定位仓库路径**：确认本技能仓库的绝对路径（即本文件所在目录 `<REPO>`，如 `D:\code\skills\project-rule-chain-init`）。验证 `<REPO>/SKILL.md` 存在且 frontmatter `name:` 值与目录名一致。

2. **清理残留**：检查 `~/.agents/skills/project-rule-chain-init` 和 `~/.claude/skills/project-rule-chain-init` 是否已存在：
   - 若为**软链接**（`lrwxrwxrwx`，`readlink` 可解引用）→ 删除旧链。
   - 若为**真实目录**（`drwxr-xr-x`）→ 这是过时拷贝，删除整个目录。
   - 若不存在 → 跳过。

3. **创建原生软链接**：
   ```
   MSYS=winsymlinks:nativestrict ln -s <REPO> ~/.agents/skills/project-rule-chain-init
   MSYS=winsymlinks:nativestrict ln -s ~/.agents/skills/project-rule-chain-init ~/.claude/skills/project-rule-chain-init
   ```
   **重要**：Windows 下必须设置 `MSYS=winsymlinks:nativestrict`，否则 `ln -s` 会静默退化为**目录拷贝**（`drwxr-xr-x` 而非 `lrwxrwxrwx`），导致改源码不生效且之后无法自动更新。macOS / Linux 直接用原生 `ln -s` 即可。

4. **验证**：
   - `ls -ld <每个链接>` 输出以 `lrwxrwxrwx` 开头（是软链接，不是目录）。
   - `readlink ~/.agents/skills/project-rule-chain-init` 返回 `<REPO>` 绝对路径。
   - `grep "name:" ~/.claude/skills/project-rule-chain-init/SKILL.md` 看到 `name: project-rule-chain-init`（通过链接链成功读到源码）。
   - 不要触碰 `~/.agents/.skill-lock.json`（软链接安装不入册，该文件只登记远程安装的技能）。

5. **告知用户**：说明已安装、链接拓扑、改源码即时生效。提醒**重启 Agent 会话**后技能列表中才会出现。

### 卸载

删除两个软链接即可，不删源码：

```
rm ~/.claude/skills/project-rule-chain-init
rm ~/.agents/skills/project-rule-chain-init
```

## 文件清单

```
project-rule-chain-init/
├─ SKILL.md                                  # 本文件
├─ agents/
│  └─ openai.yaml                            # OpenAI / Codex 技能接口描述
└─ templates/
   ├─ agent-rules/
   │  ├─ README.md                           # 链路总纲模板
   │  ├─ rules/README.md
   │  ├─ rules/git-operations.md             # 预置规则：Git 操作
   │  ├─ mechanisms/README.md                # 含七段强制结构约定
   │  ├─ dev-plans/README.md
   │  ├─ dev-plans/archived/.gitkeep
   │  └─ handoffs/README.md
   └─ bridge/
      ├─ AGENTS.md                           # Codex / 通用 Agent 桥接入口模板（默认生成）
      └─ CLAUDE.md                           # Claude 桥接入口摘要段模板（默认生成）
```
