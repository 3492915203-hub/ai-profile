# 接入：让 agent 真的读到这份档案

档案最大的失效方式不是写错，而是**没人读**。仓库放在那里，跨项目工作时任何 agent 都不会主动去看。这份文件解决的就是这件事。

标注 **[已验证]** 的条目依据是官方文档或本机实测；标注 **[未验证]** 的条目我只能给出可行思路，需要你在自己的环境里试一次。

## 零、先解决「取文件」这一步

本机实测三种取文件通道（2026-11-23）：

| 通道 | 命令 | 实测结果 |
| --- | --- | --- |
| codeload 源码包 | `curl -L -o p.tar.gz https://codeload.github.com/<用户名>/<仓库>/tar.gz/refs/tags/v0.1.1` | **200，40 KB，0.9 秒，可用** |
| GitHub API | `curl -s https://api.github.com/repos/<用户名>/<仓库>/contents/VERSION?ref=v0.1.1` | **可用**，返回 JSON，内容为 base64 |
| raw 域名 | `curl -sL https://raw.githubusercontent.com/<用户名>/<仓库>/main/AGENTS.md` | **不通**，返回 000，16 毫秒即失败 |
| 网页 Raw 按钮 | 浏览器打开文件页后点 `Raw` | 未测，取决于浏览器的网络环境 |

**结论：脚本里不要依赖 raw 域名。** 拉整仓用 codeload，取单个文件用 API，或直接在浏览器里下载。

`github.com/.../raw/...` 这种写法会 302 跳转到 raw 域名，跟着跳也拿不到内容，实测同样失败。

## 一、Claude Code

Claude Code 原生读 `CLAUDE.md`，**不读** `AGENTS.md`。官方文档给的桥接方式是在 `CLAUDE.md` 里写 `@文件路径` 做导入。[已验证：Anthropic 官方 memory 文档明确写 "Claude Code reads CLAUDE.md, not AGENTS.md"，并给出 `@AGENTS.md` 的导入写法。]

本仓库已经提供生成的 `CLAUDE.md`，其首行是 `@AGENTS.md`。两种接法：

**接法 A：项目级（每个项目各放一份）**

```bash
# 取整仓（raw 域名不通，用 codeload）
curl -L -o profile.tar.gz https://codeload.github.com/<用户名>/ai-profile/tar.gz/refs/tags/v0.1.1
tar -xzf profile.tar.gz
# 再把解压目录里的 CLAUDE.md 与 AGENTS.md 复制到项目根
```

两份都要，因为 `CLAUDE.md` 只是导入器，正文在 `AGENTS.md` 里。也可以在浏览器里逐个文件下载。

**接法 B：用户级（一次配置，所有项目生效）**

官方文档列出的用户级位置是 `~/.claude/CLAUDE.md`（原文：`User: ~/.claude/CLAUDE.md, your preferences for all projects`）。本机对应 `C:/Users/<用户名>/.claude/CLAUDE.md`。

在这个文件里写一行导入，指向本机仓库里 `AGENTS.md` 的**绝对路径**：

```md
@C:/Users/<用户名>/Desktop/ai-profile/AGENTS.md
```

**[未验证]** 导入是否接受绝对路径、是否接受 `~` 展开。官方文档只说「相对路径从持有导入语句的文件所在目录解析」，没有明说绝对路径。若这一行不生效，退化为两个办法：把 `AGENTS.md` 的正文整段复制进 `~/.claude/CLAUDE.md`，或在本机建符号链接（Windows 需管理员权限或开发者模式）。

**判断有没有生效**：新开一个会话，问「你读到的档案版本是多少」。若答不出 `v0.1.1`，就是没加载。

## 二、Codex / Cursor / Copilot / Zed / Warp / Amp

这一组原生读 `AGENTS.md`。[已验证：agents.md 站点与各厂商文档均确认，OpenAI Codex 文档原话 "Codex reads AGENTS.md files before doing any work"。]

**关键限制：最近的那份优先。** AGENTS.md 的语义是「离被修改文件最近的文件拥有最终解释权」。把档案放进某个项目后，该项目自己的 `AGENTS.md` 会盖住它。

三条应对：

1. **档案只放在独立目录或用户级位置**，不塞进别人的项目里。
2. 项目自己的 `AGENTS.md` 里加一句：`本项目的约定与个人档案冲突时，个人档案优先。` 并写明个人档案的位置。
3. 临时使用：直接给文件页地址 `https://github.com/<用户名>/ai-profile/blob/v0.1.1/AGENTS.md`，让它在浏览器侧读取。**不要给 raw 地址**，本机实测不通。

**用 tag 而不是 `main`**：tag 指向确定版本，`main` 会随改动而变，行为不可复现。本仓库已有 `v0.1.1` tag。

## 三、Gemini CLI

Gemini CLI 默认读 `GEMINI.md`，不默认读 `AGENTS.md`。[已验证：官方配置文档写 context 文件 "defaulting to GEMINI.md but configurable via context.fileName setting"。]

**接法 A**：把本仓库的 `GEMINI.md` 放进工作目录。它与 `AGENTS.md` 同源同正文。

**接法 B**：改 `.gemini/settings.json`，把上下文文件名指向 `AGENTS.md`：

```json
{ "contextFileName": "AGENTS.md" }
```

**[未验证]** 上述键名。agents.md 官方 FAQ 给的是 `contextFileName` 这种扁平写法，而 Gemini CLI 自己的配置文档提到的是 `context.fileName`，两者可能因版本而异。若不生效，直接用接法 A。

## 四、网页版对话（无文件权限）

这类 agent 读不到仓库，只能粘贴。用 `dist/profile.compact.md`，它自包含：不含「详见某文件」的引用，粘完即可用。

获取方式：在 GitHub 打开 `dist/profile.compact.md`，点 `Raw` 后全选复制；或用第一节的 codeload 方式拉整仓后看本地文件。**不要写 raw 域名的 curl 命令，本机实测不通。**

粘贴后补一句：

> 以上是我的个人档案，后续所有回答受其约束。冲突时以它为准。

## 五、检查本机副本是否过期

```bash
# 远端版本（走 API，本机可用）
curl -s "https://api.github.com/repos/<用户名>/ai-profile/contents/VERSION?ref=main" \
  | node -e "let d='';process.stdin.on('data',c=>d+=c).on('end',()=>console.log(Buffer.from(JSON.parse(d).content,'base64').toString().trim()))"
# 本地版本
cat ai-profile/VERSION
```

两个数字不一致就重新拉取。拉不下来时**不要改用第三方镜像**，先把实测速度与结果报出来再定（见 `R-NET-01`）。

## 六、不要做的事

- 不要把 `AGENTS.md` 复制成「副本 v2」长期使用。两份内容一定会在两周内分叉。
- 不要让 agent 自己「更新档案」。它的改动没经过你确认，而且会把这条路径变成注入入口（`R-TRUST-01`）。
- 不要用第三方加速域名拉取。原因见 `R-NET-01`，案例见 `docs/examples.md` 的 EX-01。
