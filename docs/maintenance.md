# 维护

## 一、改一条规则的标准流程

```bash
# 1. 改 profile.json
# 2. 生成派生文件
node scripts/build.mjs
# 3. 确认同步
node scripts/build.mjs --check
# 4. 更新 CHANGELOG.md 与 VERSION（见下）
# 5. 跑 docs/compliance-checklist.md 里相关的场景
git add -A && git commit -m "feat(R-XXX-01): 说明改了什么"
```

第 5 步不要跳。构建通过只证明文件同步，不证明新表述真的能约束行为。

## 二、版本怎么升

版本号绑定**整个规则集合**，不绑单条规则。

| 改动类型 | 版本 | CHANGELOG 段位 |
| --- | --- | --- |
| 改错别字、调整措辞但语义不变 | patch（0.1.0 → 0.1.1） | `Changed` |
| 新增一条非 block 规则 | minor（0.1.0 → 0.2.0） | `Added` |
| 新增、删除或改变 block 级规则的语义 | minor，且必须写进 `Behavior change` | `Behavior change` |
| 改了 `severity`、`appliesTo`、`exemption` 这类会改变执行结果的字段 | minor，且必须写进 `Behavior change` | `Behavior change` |

`Behavior change` 段是本档案最重要的一段。它回答的是「为什么 agent 的行为跟上次不一样了」。

## 三、删一条规则的正确姿势

不要直接删。三步：

1. 把 `status` 改成 `deprecated`，保留 `id`、`statement`、`rationale`。
2. 跑构建。它不会再进生成物，但 ID 仍在，历史引用不会指向空。
3. 过一段时间确认无人引用后，才考虑彻底移除。

**ID 永不复用。** 复用会让旧的「依据 R-XXX-02」指向一条完全不同的规则，这是最难查的一类错误。

## 四、到了 40 条怎么办

上限是 40 条，硬性。到顶时只有三条路：

1. **合并**：两条讲同一件事的规则合成一条，`statement` 用更强的那一句。
2. **降级**：把次要的 `block` 降成 `warn`，或直接 `deprecated`。
3. **归档**：把过期情境的规则标 `deprecated`，理由里写清「为什么当初需要它，现在不需要了」。

不允许的是「临时把上限调大」。上限存在的意义就是逼你做上面三选一。

## 五、复核节奏

- 每条规则有 `reviewBy`，到期时构建会打印提醒（不失败）。
- 建议每季度做一次：跑 `--check`，过一遍 `docs/compliance-checklist.md`，把过期条目标 `deprecated` 或推迟复核日期。
- 场景不符的记录格式在 `docs/compliance-checklist.md` 末尾。

## 六、脱敏复核

每次新增规则或示例后，用一次关键词搜索自查：

```bash
grep -rniE "真实姓名|城市名|出生|学校名|同学姓名" . --include="*.json" --include="*.md"
```

把引号里的词换成你的实际敏感词表。命中就要判断：这条信息是不是必须出现？去掉之后 agent 还能否做出相同动作？

**判定线**：去掉具体人名地名后，agent 是否仍能做出相同动作。能，则留公开；不能，则移入 `profile.local.json`。

## 七、派生文件冲突了怎么办

`--check` 报不一致时，只有两种正确处理：

1. 你改了 `profile.json` 但忘了跑生成 → 跑 `node scripts/build.mjs`。
2. 你直接改了派生文件 → 把改动挪回 `profile.json`，再重新生成。

**不要**为了让检查通过而把派生文件的改动提交上去。那等于制造第二个事实源，是这个仓库最想避免的失败。

## 八、CI 的作用与边界

`.github/workflows/verify.yml` 只做一件事：push 时跑 `--check`。不一致就红了。

它不检查：
- 规则写得对不对（那是人的判断）
- 措辞是否真的改变了行为（那要靠自检清单）
- 脱敏是否彻底（那要靠第六节的搜索）

CI 只保证「派生文件没有偏离事实源」。别指望它做更多。
