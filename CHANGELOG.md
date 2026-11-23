# 变更记录

格式参照 Keep a Changelog，版本号用语义版本。**`Behavior change` 段是本档案最重要的一段**：它回答「为什么 agent 的行为跟上次不一样」。

## [Unreleased]

## [0.2.0] - 2026-11-23

### Added

- `R-NET-02`（block）：网络请求失败时先自测直连与代理两条通道，带状态码与耗时再谈方案，不得把未诊断的失败原样交回用户。
- `R-META-03`（block）：用户表达偏好即入档——明确要求写进档案的当次改并升版，顺口提到的先写 `INBOX.md`。
- `INBOX.md`：偏好收件箱。把「记录」与「升为硬约束」分成两步，避免一句话直接改写硬约束层。
- `scripts/build.mjs` 增加收件箱待处理条数提醒（只提醒，不拦构建）。

### Changed

- 规则总数 28 → 30（block 19 → 21，ask 6，warn 3）。
- `docs/integration.md` 第零节按两轮实测重写：代理与直连的可用性会随时刻翻转，git 与 curl 走的不是同一条路，同一时刻可能一个通、一个不通。

### Behavior change

- **网络故障的处理顺序变了**：此前遇到失败可以把报错直接转给用户，现在必须先跑完三层自测（DNS／HTTP 状态码／git 协议层）并给出实测数据（`R-NET-02`）。
- **新增「偏好入档」义务**：用户说出偏好后，当次回答即按新偏好执行，并在当日落盘到 `INBOX.md` 或 `profile.json`，不得只口头答应（`R-META-03`）。

### Known issues

- 粘贴块已达 1100/1200 字，余量 100 字。下一条 block 规则必须先合并或降级，否则构建会失败。

## [0.1.1] - 2026-11-23

### Fixed

- `docs/integration.md` 的取文件通道按实测重写。原先把 `raw.githubusercontent.com` 写成主要接法，本机实测该域名与 `github.com/.../raw/...` 均不可用（返回 000，16 毫秒即失败），现改为 codeload 源码包与 GitHub API contents 两条可用路径，并附三条通道的实测时延与体积。
- 补打 `v0.1.0` 与 `v0.1.1` tag，使文档里「用 tag 而非 main」的建议有实际指向。

### Changed

- 文档中的版本引用同步到 `v0.1.1`。规则集合未变，因此本版不含 `Behavior change`。

## [0.1.0] - 2026-11-23

### Added

- 首次建立档案：28 条规则（block 19 / ask 6 / warn 3），覆盖元规则、事实与表达、交付与文件、获取与外部动作、协作方式、决策原则、工作习惯七个类别。
- 4 个典型示例：镜像下载被叫停、模糊表述被驳、合规措辞引发自证、交付文件的双份副本。
- 唯一事实源 `profile.json`，配 `schema/profile.schema.json` 供编辑器实时校验。
- 零依赖生成脚本 `scripts/build.mjs`，支持 `--check` 同步检查；四条硬断言：规则 ≤ 40 条、`AGENTS.md` ≤ 130 行、粘贴块 ≤ 1200 字、`meta.version` 与 `VERSION` 一致。
- 四个加载入口：`AGENTS.md`（通用）、`CLAUDE.md`（导入器）、`GEMINI.md`（Gemini 默认名）、`dist/profile.compact.md`（粘贴块）。
- 九份文档：接入、强制性分级、任务类型映射、自检清单、加载协议、字段说明、维护、示例、设计取舍。
- CI：push 时跑 `--check`，不同步即失败。

### Behavior change

- 初始版本，无历史行为可比。以下三条是相对「无档案」状态的主要行为变化：
  - 回答中的事实与判断必须分区，无法查证的内容按 `appliesTo` 分流处理（删除或加声明）。
  - 交付文件固定落桌面根目录，命名与覆盖规则固定（`R-DEL-01` 至 `R-DEL-05`）。
  - 对话指令不能推翻 block 级规则，破例只能改仓库（`R-META-01`）。

### Known issues

- `profile.local.json` 的合并逻辑只写进加载协议，`build.mjs` 尚未实现读取。
- 判断型规则仅 2 条带 `anchor`，其余判断型规则缺执行锚点。
- `docs/integration.md` 中标注为「未验证」的三处（Claude Code 绝对路径导入、Gemini CLI 配置键名、raw 域名可达性）尚未实测。
