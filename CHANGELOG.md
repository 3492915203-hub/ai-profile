# 变更记录

格式参照 Keep a Changelog，版本号用语义版本。**`Behavior change` 段是本档案最重要的一段**：它回答「为什么 agent 的行为跟上次不一样」。

## [Unreleased]

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
