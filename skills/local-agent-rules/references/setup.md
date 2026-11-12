# 安装与接入

## 1. 一份仓库正文，多个入口

本 skill 的 `SKILL.md` 与 `references/` 用于分发通用操作规范；克隆或上传仓库不等于安装 skill。

若本机的 `AGENTS.md` 等全局指令已包含同一套完整规则，就只使用全局指令，不再安装或加载本 skill，避免重复和冲突。以下安装步骤仅供选择 skill 方式的其他机器使用。

采用 skill 方式时，各 Agent 可复用整个 skill 目录，不分别维护正文副本。安装只负责让工具能发现 skill；若要求每次任务都遵守，仍需在全局入口加入简短加载指令，不能仅依赖 description 自动触发。

已存在的目录不能直接覆盖。先检查内容、软链接和本机安装路径；有用户定制时先备份并合并。

## 2. Agent 入口

按当前安装版本确认各工具的 skill 搜索范围，常用用户级目录如下：

| Agent | Skill 目录 | 全局规则入口 |
|---|---|---|
| Claude Code / 兼容客户端 | `~/.claude/skills/local-agent-rules/` | `~/.claude/CLAUDE.md` |
| Codex CLI | 支持该入口的版本：`~/.codex/skills/local-agent-rules/`；也应核对当前版本的共享 skills 目录 | `~/.codex/AGENTS.md` |
| Cursor IDE / CLI | `~/.cursor/skills/local-agent-rules/` | IDE 的 User Rules；CLI 的实际项目/父级规则发现机制另行验证 |
| Antigravity | `~/.gemini/config/skills/local-agent-rules/` | `~/.gemini/GEMINI.md` |
| Zed 原生 Agent | 按当前版本核实 skill 支持 | `~/.config/zed/AGENTS.md` 可显式要求读取本地 skill |

链接整个目录，而不只是 `SKILL.md`，避免相对链接和后续资源失效。以下只是 Claude 的安装示例，将 `/绝对路径/skills仓库` 替换为实际克隆位置：

```sh
mkdir -p "$HOME/.claude/skills"
ln -s "/绝对路径/skills仓库/skills/local-agent-rules" \
  "$HOME/.claude/skills/local-agent-rules"
```

不支持原生 skills 发现的工具，可以通过全局规则显式读取 `SKILL.md`。不能访问本机文件的云端聊天或远程 Agent 不适用本地链接，需要在其执行环境单独部署。

## 3. 全局加载指令

把下面的短指令放在各 Agent 实际生效的全局入口；替换为实际绝对路径，不要假定所有工具都支持同一种 `@include` 语法：

```text
处理代码检索、开发修改、本机命令、测试、SSH 排查或 Git 平台任务前，加载 local-agent-rules skill。
若没有原生 Skill 工具，读取 /绝对路径/skills仓库/skills/local-agent-rules/SKILL.md，并按需读取其中引用的文件。
本机差异只读取 ~/.config/local-agent-rules/machine.md；读取失败须说明，不能假定已加载。
CodeGraph 必须使用当前项目/worktree 的新鲜索引，普通变更优先 sync，不默认全量 index。
```

这是模型加载指令，不是自动内联或 hook。需要通过新会话检查实际是否读取了正文。

## 4. 本机配置隔离

本机配置位于仓库之外：`~/.config/local-agent-rules/machine.md`。可以记录：

- 当前机器使用的版本管理器和初始化要求。
- 已授权用途的 SSH 连接参数与密钥路径，不记录私钥内容。
- GitLab 地址、认证环境变量名，不记录 Token 值。

不要将该文件复制、软链接或提交进仓库；Token、私钥、认证 cookie 一律使用安全凭据存储。更换机器时人工核对该文件，不默认复用旧主机信息。

## 5. CodeGraph 管理块与更新

CodeGraph 等安装器可能维护 `CODEGRAPH_START/END` 指令块，升级时会重写块内内容。某些版本还会通过临时文件加 rename 的原子写入，把指令文件的软链接替换成普通文件。

- 自定义加载指令放在工具管理块之外，不直接改工具安装目录或管理模板。
- 不只依赖指令文件之间的软链接实现同步：让每个全局入口都保留同一条加载本 skill 的短指令。
- 安装器之后即使将软链接拆开，正文仍只存在于 skill 仓库；入口内容需要检查，但业务规则无需复制同步。
- 不设“升级后强行恢复链接”的后台任务，不覆盖其他会话或用户刚修改的全局规则。

## 6. 更新与验收

- 更新仓库前检查 `git status`，不要覆盖本地未提交改动；只从用户确认的远端获取更新，不在每个任务开始时自动联网拉取。
- 更新后用新会话验证 skill 被发现或正文被实际读取，本机配置存在时也应验证其读取情况。
- 仅有软链接、菜单显示或“设置已保存”不能宣称模型行为已经验证。
- 测试可以询问规则摘要，不读取真实 Token 值、不连接服务器、不执行全量测试或写操作。
- 不要将外部 Agent 不支持的加载机制或本机未部署的配置报告为成功。
