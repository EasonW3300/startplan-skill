# startplan

一个“先规划、后执行”的 Agent Skill：先读环境、收敛意图、产出完整计划，等待用户明确批准后才进入实现。

适用于 OhMyAgent / 百智办公，也适用于会扫描 `skills/` 目录的宿主。

## 它解决什么问题

普通对话里，模型容易在需求还没收敛时直接改代码。`startplan` 把流程拆成两层：

- `startplan`：薄路由，收到 `/startplan` 后立即加载 `startplan-core`。
- `startplan-core`：负责 Ground → Intent → Implementation，激活 `plan` Workflow 和 `plan-readonly` 白名单，最后只输出一个 `<proposed_plan>`。

规划阶段的白名单不包含 `Write`、`Edit`、`Bash`、`Agent` 等会改变环境的工具，也不包含 `ExitPlanMode`。计划输出后本轮结束；只有用户明确说“按当前计划执行”或等价指令，才开始实现。

## 快速开始（百智办公）

1. 下载 `dist/startplan.zip` 和 `dist/startplan-core.zip`。
2. 在「技能资源」中分别导入，得到两个独立资源：
   - `startplan`
   - `startplan-core`
3. 两个资源都启用。
4. 新建会话，发送：

```text
/startplan 你的需求
```

也可以直接把本项目链接 `https://github.com/EasonW3300/startplan-skill` 丢给百智办公，发送指令：`帮我部署这个skill`。如果宿主不支持自动安装，它会根据仓库里的安装说明给出导入步骤。

> 只导入 `startplan` 不导入 `startplan-core`，会报 `Skill not found: startplan-core`。百智办公只把当前选中的技能资源传给引擎，所以 core 必须作为第二个资源存在。

详细步骤和校验值见 [`dist/百智办公-安装说明.md`](dist/百智办公-安装说明.md)。

## 其他安装方式

| 宿主 | 使用文件 | 说明 |
| --- | --- | --- |
| 百智办公 / MonkeyAI | `startplan.zip` + `startplan-core.zip` | 两个资源都导入并启用（推荐） |
| 只允许一个技能资源的宿主 | `startplan-desktop.zip` | 自包含单包，入口和 core 已合并 |
| 会扫描技能目录的宿主 | `startplan-skill-bundle.zip` | 解压得到 `startplan/` 和 `startplan-core/` |

## 工作方式

```text
/startplan <需求>
  └─ startplan（薄路由）
       └─ Skill("startplan-core")
            └─ Workflow(select_route, route="plan")
                 └─ planning（plan-readonly）
                      └─ complete_step(plan_ready)
                           └─ <proposed_plan>
                                └─ 用户明确批准
                                     └─ 执行
```

### 规划阶段的白名单

`plan-readonly` 只允许只读和规划相关工具：

`Read`、`Glob`、`Grep`、`LSP`、`Skill`、`SkillSearch`、`ToolSearch`、`ToolLoad`、`AskUserQuestion`、`Workflow`、`EnterPlanMode`、`MemoryRead`、`MemorySearch`、`ListMcpResourcesTool`、`ReadMcpResourceTool`、`WebFetch`、`WebSearch`。

`Write`、`Edit`、`Bash`、`Agent` 等不在白名单内。

## 仓库结构

```text
.
├── README.md
├── skills/
│   ├── startplan/SKILL.md
│   └── startplan-core/
│       ├── SKILL.md
│       └── workflow.json
└── dist/
    ├── startplan.zip
    ├── startplan-core.zip
    ├── startplan-skill-bundle.zip
    ├── startplan-desktop.zip
    ├── startplan-desktop/
    │   ├── SKILL.md
    │   └── workflow.json
    ├── README.md
    └── 百智办公-安装说明.md
```

## 验证

导入后新建会话，确认：

- 会话的 `execution_resources.skills` 同时包含 `startplan` 和 `startplan-core`。
- 模型先调用 `Skill("startplan-core")`，再调用 `Workflow(select_route, route="plan")`。
- 规划阶段尝试 `Write` 会被 `plan-readonly` 拒绝。
- 输出一个 `<proposed_plan>` 后本轮结束，目标文件没有变化。
- 回复“继续”“可以”“行”不会触发执行；明确回复“按当前计划执行”后才执行。

## 文件校验

| 文件 | SHA256 |
| --- | --- |
| `startplan.zip` | `c673925a2026b7e816b886294675aef7bf2f33afc1e3cc68e68a44e17d06ace4` |
| `startplan-core.zip` | `29c09fc1239a7d720daa1d11af2ae43fbd43dbc9aa871317b5a639b7a1b09879` |
| `startplan-skill-bundle.zip` | `cf5309a229f07889625072f3672c52f39ff869d3dc371735c8febb99867df2db` |
| `startplan-desktop.zip` | `a27d816f00c00a5faf913cec92f84a72539efd6ed2781c647744d4c3e8547d80` |

## 兼容性

已在 OhMyAgent `95da8528970f` / 百智办公 `26.1015.1` 上验证。Workflow 配置格式为 `version: 2`。

## 说明

- 导入 `startplan-core` 后，斜杠菜单可能多出一个 `/startplan-core`；主入口仍然是 `/startplan`。
- 嵌套的 `startplan/startplan-core/SKILL.md` 不会被引擎发现；core 必须是独立技能目录或独立资源。
- `startplan-desktop.zip` 是单资源宿主的备选方案，行为等价，但没有独立的 core 名字。
