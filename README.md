# AI 个人档案

一份写给 AI agent 看的个人说明书：我的硬性底线、不可接受的行为、协作偏好、工作习惯、沟通风格、决策原则，以及四个真实反例。

**人看这个文件就够了，agent 读 [AGENTS.md](AGENTS.md) 就够了。**

## 30 秒接入

| 你在用什么 | 怎么做 |
| --- | --- |
| Claude Code | 在 `~/.claude/CLAUDE.md` 里加一行导入，指向本仓库的 `AGENTS.md`（用绝对路径） |
| Codex / Cursor / Copilot / Zed / Warp | 把 `AGENTS.md` 放到项目根；不放也行，让 agent 先读固定 URL |
| Gemini CLI | 把 `GEMINI.md` 放到工作目录（它默认读这个名字，不读 `AGENTS.md`） |
| 网页版对话 | 打开 `dist/profile.compact.md`，整段粘贴，再说一句「后续回答受此约束」 |

具体命令、路径写法、以及哪些步骤我尚未验证，都在 [docs/integration.md](docs/integration.md)。

## 档案里有什么

- **30 条规则**，分三级：`block` 21 条（无条件执行）／`ask` 6 条（命中即停，先问）／`warn` 3 条（可偏离须说明）。
- **七个维度**，其中五个是规则类别：事实与表达、交付与文件、获取与外部动作、协作方式、决策原则、工作习惯、元规则。
- **另外两个维度不是独立数据**：「硬性底线」= 所有 `block` 条目的集合，「不可接受行为」= 每条规则里的反例字段。同一件事只写一遍，避免两份内容互相打架。
- **4 个典型示例**，见 [docs/examples.md](docs/examples.md)。

## 改一条规则（三步）

```bash
# 1. 改 profile.json（VS Code 会按 schema 实时报错）
# 2. 重新生成派生文件
node scripts/build.mjs
# 3. 确认派生文件与事实源同步
node scripts/build.mjs --check
```

**不要直接改 `AGENTS.md`、`CLAUDE.md`、`GEMINI.md`、`dist/`、`docs/examples.md`。** 它们首行都标了 `GENERATED`，下次生成会被覆盖；CI 也会因为不同步而失败。

如果是**要新增一条规则**（比如刚才对话里说出的偏好），先看 [INBOX.md](INBOX.md)：明确要写进档案的当次就改 `profile.json` 并升版本；只是顺口提到的，先在那里记一行。

## 为什么这样设计

三个取舍最值得说：

1. **一个 JSON 事实源 + 脚本生成**，而不是手写四份入口文件。手写必然漂移，漂移的档案比没有档案更糟。
2. **不设判例法**。规则之间的真实冲突（例如「删除无法查证的内容」与「方法论交付物本身就是推理」）用 `appliesTo` 标签分流，并显式登记在一个字段里，不让 agent 每次现场裁决。
3. **上限写死在脚本里**：规则 ≤ 40 条、`AGENTS.md` ≤ 130 行、粘贴块 ≤ 1200 字。超了构建就失败，逼我归档而不是无限追加。

完整推理见 [docs/design-notes.md](docs/design-notes.md)。

## 隐私边界

- 公开仓库里没有真名、城市、出生信息、学校名、同学姓名。
- 私密条目写在 `profile.local.json`（已 gitignore），只能追加 `L-` 前缀的规则，不能覆盖公开规则。
- 判定线：**去掉具体人名地名后，agent 是否还能做出相同动作。** 能，则留公开；不能，则移入本地文件。

## 目录

```
AGENTS.md / GEMINI.md / CLAUDE.md   agent 入口（生成物）
profile.json                        唯一事实源
INBOX.md                            偏好收件箱（人读，不是约束）
schema/profile.schema.json          编辑器提示用，不是 CI 权威
dist/profile.compact.md             粘贴块（生成物）
docs/                               说明文档，见下表
scripts/build.mjs                   生成与校验
```

| 想知道 | 读哪里 |
| --- | --- |
| 怎么接进某个 agent | [docs/integration.md](docs/integration.md) |
| 哪条能技术强制、哪条只能靠自觉 | [docs/enforcement.md](docs/enforcement.md) |
| 某类任务该加载哪几条 | [docs/task-profiles.md](docs/task-profiles.md) |
| 每条规则的完整说明与反例 | [docs/examples.md](docs/examples.md)、`profile.json` |
| 字段怎么填、填错会怎样 | [docs/field-reference.md](docs/field-reference.md) |
| 改完怎么验证有没有生效 | [docs/compliance-checklist.md](docs/compliance-checklist.md) |
| 版本、评审、归档怎么管 | [docs/maintenance.md](docs/maintenance.md) |
| 用户新提出的偏好怎么入档 | [INBOX.md](INBOX.md)、[docs/maintenance.md](docs/maintenance.md) |

## 已知限制

- **指令文件是上下文，不是锁。** `AGENTS.md`／`CLAUDE.md` 在多数客户端里只是送给模型的参考文本，无法阻止它调用某个工具。真正能拦住的只有 hook，见 [docs/enforcement.md](docs/enforcement.md)。
- **没有自动验证。** 规则是自然语言，没法单元测试。唯一的验证手段是 [docs/compliance-checklist.md](docs/compliance-checklist.md) 里那 14 个场景，人工过一遍。
- **可能过期。** 本机副本不会自己更新，比对命令在 [docs/integration.md](docs/integration.md)。

## 来源

"AGENTS.md." *agents.md*, Agentic AI Foundation, https://agents.md/. 访问日期 2026-11-23.

"Claude Code Memory (CLAUDE.md)." *Anthropic Documentation*, https://code.claude.com/docs/en/memory. 访问日期 2026-11-23.
