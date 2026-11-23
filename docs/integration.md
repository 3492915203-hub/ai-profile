# 接入：让 agent 真的读到这份档案

档案最大的失效方式不是写错，而是**没人读**。仓库放在那里，跨项目工作时任何 agent 都不会主动去看。这份文件解决的就是这件事。

标注 **[已验证]** 的条目，依据是官方文档或本机实测；标注 **[未验证]** 的条目，我只能给出可行思路，你需要在自己的环境里试一次。

## 一、Claude Code

Claude Code 原生读 `CLAUDE.md`，**不读** `AGENTS.md`。官方文档给的桥接方式是在 `CLAUDE.md` 里写 `@文件路径` 做导入。[已验证：Anthropic 官方 memory 文档明确写 "Claude Code reads CLAUDE.md, not AGENTS.md"，并给出 `@AGENTS.md` 的导入写法。]

本仓库已经提供生成的 `CLAUDE.md`，其首行是 `@AGENTS.md`。两种接法：

**接法 A：项目级（每个项目各放一份）**

```bash
# 在项目根目录
curl -sL https://raw.githubusercontent.com/<你的用户名>/ai-profile/v0.1.0/CLAUDE.md -o CLAUDE.md
curl -sL https://raw.githubusercontent.com/<你的用户名>/ai-profile/v0.1.0/AGENTS.md -o AGENTS.md
```

两份都要，因为 `CLAUDE.md` 只是导入器，正文在 `AGENTS.md` 里。

**接法 B：用户级（一次配置，所有项目生效）**

官方文档列出的用户级位置是 `~/.claude/CLAUDE.md`（原文：`User: ~/.claude/CLAUDE.md, your preferences for all projects`）。本机对应 `C:/Users/<你的用户名>/.claude/CLAUDE.md`。

在这个文件里写一行导入，指向仓库里的 `AGENTS.md` **绝对路径**：

```md
@C:/Users/<你的用户名>/Desktop/ai-profile/AGENTS.md
```

**[未验证]** 导入是否接受绝对路径、是否接受 `~` 展开。官方文档只说「相对路径从持有导入语句的文件所在目录解析」，没有明说绝对路径。若这一行不生效，退化为两个办法：把 `AGENTS.md` 的正文整段复制进 `~/.claude/CLAUDE.md`，或在本机建符号链接（Windows 需管理员权限或开发者模式）。

**判断有没有生效**：新开一个会话，问「你读到的档案版本是多少」。若答不出 `v0.1.0`，就是没加载。

## 二、Codex / Cursor / Copilot / Zed / Warp / Amp

这一组原生读 `AGENTS.md`。[已验证：agents.md 站点与各厂商文档均确认，OpenAI Codex 文档原话 "Codex reads AGENTS.md files before doing any work"。]

**关键限制：最近的那份优先。** AGENTS.md 的语义是「离被修改文件最近的文件拥有最终解释权」。这意味着把档案放进某个项目后，该项目自己的 `AGENTS.md` 会盖住它。

三条应对：

1. **档案只放在独立目录或用户级位置**，不塞进别人的项目里。
2. 项目自己的 `AGENTS.md` 里加一句：`本项目的约定与个人档案冲突时，个人档案优先。` 并写明个人档案的位置。
3. 临时使用：直接在对话里贴固定版本 URL，让 agent 先读。

```
https://raw.githubusercontent.com/<你的用户名>/ai-profile/v0.1.0/AGENTS.md
```

**用 tag 而不是 `main`**：tag 指向确定的版本，`main` 会随我改动而变，行为不可复现。

## 三、Gemini CLI

Gemini CLI 默认读 `GEMINI.md`，不默认读 `AGENTS.md`。[已验证：官方配置文档写 context 文件 "defaulting to GEMINI.md but configurable via context.fileName setting"。]

两种接法：

**接法 A**：把本仓库生成的 `GEMINI.md` 放进工作目录。它与 `AGENTS.md` 同源同正文。

**接法 B**：改 `.gemini/settings.json`，把上下文文件名指向 `AGENTS.md`：

```json
{ "context": { "fileName": "AGENTS.md" } }
```

**[未验证]** 上述 JSON 键名与层级。agents.md 官方 FAQ 给的是 `{ "contextFileName": "AGENTS.md" }` 的扁平写法，两者可能因版本而异。若不生效，直接用接法 A。

## 四、网页版对话（无文件权限）

这类 agent 读不到仓库，只能粘贴。用 `dist/profile.compact.md`，它是自包含的：不含任何「详见某文件」的引用，粘完即可用。

```bash
# 取最新版
curl -sL https://raw.githubusercontent.com/<你的用户名>/ai-profile/main/dist/profile.compact.md
```

粘贴后在后面补一句：

> 以上是我的个人档案，后续所有回答受其约束。冲突时以它为准。

## 五、检查本机副本是否过期

```bash
# 远端版本
curl -s https://raw.githubusercontent.com/<你的用户名>/ai-profile/main/VERSION
# 本地版本
cat ai-profile/VERSION
```

两个数字不一致，就重新拉取。**[未验证]** 本机网络对 `raw.githubusercontent.com` 的可达性——本机此前实测过直连 GitHub 时通时不通，拉不下来时不要改用第三方镜像，先报告实测结果。

## 六、不要做的事

- 不要把 `AGENTS.md` 复制成一个「副本 v2」长期使用。两份内容一定会在两周内分叉。
- 不要让 agent 自己「更新档案」。它的改动没有经过你的确认，而且会把这条路径变成注入入口（见 `R-TRUST-01`）。
- 不要用第三方加速域名拉取。原因见 `R-NET-01` 与 `docs/examples.md` 的 EX-01。
