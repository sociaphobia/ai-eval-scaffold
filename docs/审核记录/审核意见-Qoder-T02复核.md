# T02 第二轮定向复核报告

- 批次：`20260927-T02-QODER-002`
- 方法：只读回读 `审核意见-Qoder-T02初版.md`、`审核处理-Qoder-T02初版.md`、`架构设计.md` 第八节增量、`开发计划.md` T02、`app/main.py`、`app/service/imports.py`、`app/service/rules.py`、`app/api/{projects,batches}.py`、`app/model/errors.py`、`app/repository/sqlite.py`、`tests/{test_imports,test_repository,test_rules,test_smoke}.py`、`HANDOFF.md`、`pyproject.toml`、`web/dist/index.html`。
- 执行限制：本会话仅有文件读取能力，**未独立回算 5 个送审快照 SHA-256，未独立运行 pytest**；`24 passed, 1 warning` 按生产方声明记录，不写成实跑。

## 一、总体判定：修正性接受

三项重要缺口实质推进：重-1 已闭环（含一处断言偏弱）、重-2 大部分闭环（一处分支未真正检查结果）、重-3 系原审核方漏数，予以撤回。增量未引入会造成错误放行的代码缺陷，架构第八节四处新增条款与实现逐条一致。剩余两项小补可在 T03 提审前闭环。

## 二、四项必核逐条结论

| 必核项 | 结论 | 出处 |
| --- | --- | --- |
| ① 三个默认错误路径统一结构且含中文消息 | 实现符合；测试断言偏弱 | 结构：main.py:33-36、:43；中文消息「请求字段无效或缺失」「未找到指定路径」「请求方法不受支持」；测试 test_imports.py:116-128 仅断 key 集合与 `isinstance(message, str)`，未断中文内容、未断两条 422 的 `code=="invalid_import"`、未断 404 `issues==[]` |
| ② 部分／无交集、无基线 409、摘要边界、超长与缺项是否真正检查结果 | 三项符合，一项不符合 | 无基线 409：test_imports.py:144-150 断状态码、`code` 且回归列表为空 ✓；无交集：:163-168 断 `code` 与 message 子串「先建立新用例基线」✓；摘要边界：:172-180 以等／不等双向断言排除 `answer`、用例文件顺序，含 `input` 与要点顺序 ✓；超长＋缺项：:183-192 断 `issues>=2` 且零写入 ✓；**部分重合：:160-162 只断 422 状态码，未检查结果内容** |
| ③ 旧计数 14／15 分歧的文件证据 | 15 成立，原重-3 撤回 | 见第三节重-3 证据链（HANDOFF.md:46、:190；test_smoke.py:6,12,20,29,38；web/dist/index.html:6；开发计划.md:75-77） |
| ④ 四条架构新增条款与实现一致 | 全部一致 | 重复键：架构:151 ↔ imports.py:18-24、:30-31 ↔ test_imports.py:195-204；单空格规则值：架构:151 ↔ imports.py:96（`not rule["text"]` 只拒空串，不 strip）↔ test_imports.py:207-212；未知项目优先：架构:151 ↔ imports.py:113-116（先查 projects 再 `validate_document`）↔ test_imports.py:215-222；case_id 原值／256／纯空白：架构:188 ↔ imports.py:63-68 |

补充核对：回归配对读的是 `baseline_events JOIN batches` 且带 `invalidated=0`（imports.py:122-125），与架构:82、:86 的资格语义一致；配对判定为标识集合严格相等（:130），未被放宽为子集；摘要按 `case_id` 排序、`criteria`/`rules` 保序（:103-109）。

## 三、逐条判定

**重-1（默认 404／422 泄漏 `detail`）→ 修正性接受。** 两个处理器已注册（main.py:27、:39），三路最小复现已被 test_imports.py:119-128 覆盖为统一 `{code,message,issues}`。剩余条件：把该用例断言补强为「两条 422 的 `code == "invalid_import"`、404 的 `issues == []`、`message` 为非空中文字符串」，否则未来改成英文消息测试仍绿。

**重-2（回归门禁与摘要缺直接测试）→ 修正性接受。** 开发计划:86 明定项已由 5 个新用例覆盖（:144、:153、:172、:183、:116）。唯一未闭环：**部分重合分支**只断 422。最小复现：把 imports.py:131 的 `issues` 改为 `[]`、或把 :132 的三元消息改成同一句，24 例仍全绿；而开发计划:53 与架构:182 要求「部分重合也整批拒绝并列缺失／多余题」。请补 `rejected.json()["code"]`、`{i["case_id"] for i in issues}` 与两条不同 message 的断言。次要残留：`rules` 数组内顺序敏感未测（test_imports.py:178-180 只改 `criteria` 顺序）；`version_label` 被排除仅由 `basis_digest(cases)` 签名保证，无 API 层断言。

**重-3（生产方证据与工作区不符）→ 驳回原意见，接受处理表反驳。** 证据链：`test_smoke.py` 现有 5 个 `def test_`（:6 健康、:12 陌生主机、:20 外站来源、:29 开发来源、:38 构建产物），其中第 5 例断言 `GET /` 返回含「AI 应用评测脚手架」的页面（:39-41），该页文案来自 `web/dist/index.html:6`，属 T01「构建后静态前端由同一进程提供」交付面（开发计划:75-77、架构:17）；`HANDOFF.md:46` 与 `:190` 记录 T01 收尾时烟测为 `5 passed`，即 T02 送审前 `test_smoke.py` 已是 5 例。故初版送审时四文件合计 7+2+1+5=15，与当时命令输出一致；原报告的 14 是漏数静态页面烟测所致。限定说明：本会话无 Git 对象历史与哈希工具，该结论基于工作区现存文档与测试文件交叉证据，属可复核推断，不回算 24 例倒推。

**可-1（纯空格规则值）→ 接受。** 口径已写入架构:151，实现只拒空串，用例先红后绿。

**可-2（事务中途失败）→ 接受。** 回滚封装 sqlite.py:68-73，新增用例 test_repository.py:19-34 以主键冲突证明 `projects`、`batches` 均为 0。

**可-3（未知项目与坏文件先后）→ 接受。** 已裁定 404 优先并写入架构:151，实现 imports.py:113-116，用例 test_imports.py:215-222 用坏 JSON 反证未解析文件。

**可-6（JSON 重复键静默覆盖）→ 接受。** `object_pairs_hook` 已在架构:151 升为契约，实现 imports.py:18-24。

**可-4、可-5（本次未处理，处理表也未声称处理）→ 维持可选不门禁。** 其中「批内 `case_id` 主键实际生效」仍属开发计划:86 明定自测项而无用例（schema 见 sqlite.py:35，测试仅覆盖外键 test_repository.py:8-16），建议 T03 顺带补一条插入同批重复标识断言 `IntegrityError`。

## 四、增量引入的新发现（均为可选，不门禁）

- **增-1** 重复键与非标准数值共用同一消息：`_unique_object` 抛的 `ValueError` 被 imports.py:34-35 统一映射成「文件必须是有效的 UTF-8 JSON」，中文但语义误导项目方去查编码；相关用例（test_imports.py:112、:204）只断 `path == "$"`，未断消息。
- **增-2** main.py:41-43 的兜底分支把「非 404、非 405」一律标成 `workflow_conflict`＋「请求无法完成」；当前无可触发路径，但 T03/T04 若引入 413、400 等 `HTTPException` 将与架构:184 的 409 门禁语义冲突，建议改为中性 code 或在架构错误表补一行。
- **增-3** main.py:35 对 `RequestValidationError` 一律返回 `invalid_import`，该码在架构:182 的语义是导入文件问题，现也会出现在 `POST /api/v1/projects` 的 body 校验（test_imports.py:120）；与 projects.py:15-16 已用 `invalid_import` 表达名称校验属同一口径缺口，建议在错误表补一行裁定。
- **增-4** 要点 `text` 按 `strip()` 拒绝纯空白（imports.py:82），架构:151 只对规则 `text` 补了单空格口径，要点文字口径未写；与 case_id「非空且非纯空白」（架构:188）方向一致，建议补一句以消除不对称。
- **增-5** `test_构建产物由本机服务提供` 使后端全量测试依赖未提交的 `web/dist`（开发计划:75 声明 .gitignore 忽略构建产物）；干净工作副本先跑 `pytest` 时 main.py:73-75 不挂载，`GET /` 返回 404 而该用例变红。属 T01 遗留，T06 全流程须先 `npm run build` 或加显式跳过条件。

## 五、T02 能否结束并进入 T03

可以进入 T03，且 T03 与下列两项小补可并行：① 重-2 的「部分重合」分支补结果断言（imports.py:131-132）；② 重-1 的统一错误用例补中文／code／空 `issues` 断言（test_imports.py:116-128）。这两项与可-5 的批内唯一约束用例须在 **T03 提审证据中一并闭环**，闭环前 T02 不宜对外宣告自测全部完成；增-1～增-5 与是否门禁无关，按 T03/T05/T06 各自阶段处理。本轮无阻断项，轮次计为第 2 轮，仍在 3 轮上限内。

批次：20260927-T02-QODER-002 ｜ 状态：已完成 ｜ 判定：修正性接受
