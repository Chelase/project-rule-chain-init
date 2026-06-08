# Git 操作规则

## 1. 本地检查

本地只读检查使用 `git`，例如：

- `git status`
- `git diff`
- `git log`
- `git show`

## 2. 远端平台

当前 Git 远端平台：{{GIT_PLATFORM}}

{{GIT_PLATFORM_BLOCK}}

## 3. 提交内容语言

git 提交内容必须使用中文，包括：

- 提交信息（commit message）的标题和正文。
- 合并请求（MR/PR）的标题和描述。
- 议题（issue）的标题和描述。

## 4. 提交信息格式

git 提交信息必须同时遵守约定式提交规范（Conventional Commits）和中文描述要求。

提交标题格式：

```text
<type>(<scope>): <中文说明>
```

约束：

- `type` 必须使用约定式提交类型，例如 `feat`、`fix`、`docs`、`refactor`、`test`、`chore`、`build`、`ci`、`perf`、`style`、`revert`。
- `scope` 可选；能准确表达模块或功能域时优先填写，例如 `feat(auth): 支持 OAuth2 登录`。
- 冒号后的提交说明必须使用中文。
- 不要使用缺少 `type:` 前缀的纯中文提交标题，例如 `完善登录功能`。

## 5. 提交粒度

- 一个提交只做一件事。
- 不要在一个提交里混入无关改动（例如修 bug 的同时重构另一个模块）。
- 提交前用 `git diff --staged` 确认改动范围。

<!-- 以下为占位符模板，初始化时由 Agent 根据远端类型选择填充 -->

---

### GitHub（使用 `gh` 工具）

当前状态：{{GH_STATUS}}

<!-- 初始化时保留以下内容或删除，取决于用户是否选择 gh -->

- gh 工具用于 PR 创建、Issue 管理、CI 查看等 GitHub 协作操作。
- 安装：`winget install GitHub.cli`（Windows）/ `brew install gh`（macOS）
- 认证：`gh auth login`（浏览器 OAuth 流程）
- 查看认证状态：`gh auth status`

```powershell
# 常用命令
gh pr create --title "feat: xxx" --body "描述"
gh issue list
gh run list
gh repo view
```

---

### GitLab（使用 `glab` 工具）

当前状态：{{GLAB_STATUS}}

<!-- 初始化时保留以下内容或删除，取决于用户是否选择 glab -->

- glab 工具用于 MR 创建、Issue 管理、流水线查看等 GitLab 协作操作。
- 安装：`winget install GLab.GLab`（Windows）/ `brew install glab`（macOS）
- 查看认证状态：`glab auth status`

**自建 GitLab 实例**：若 GitLab 为公司自建服务（非 `gitlab.com`），需额外配置：

```powershell
# 告诉 glab 你的 GitLab 域名（不带 https://）
glab config set host <your-gitlab-domain>

# 使用 Personal Access Token 认证（GitLab → Settings → Access Tokens，需 api scope）
glab auth login --hostname <your-gitlab-domain> --token <your-pat>
```

**自建实例常见注意事项**：
- 域名必须可解析（DNS 或 hosts 文件），否则 `glab` 无法访问 API。
- 若使用自签名证书，需设置 `GIT_SSL_NO_VERIFY=true` 或导入 CA 证书。
- Token 需在目标 GitLab 实例上生成，不能跨实例使用。

```powershell
# 常用命令
glab mr create --title "feat: xxx" --description "描述"
glab issue list
glab ci list
glab repo view
```
