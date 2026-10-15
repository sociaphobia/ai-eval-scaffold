复核完成。以下为第三轮最终定向复核报告。

# T02 第三轮最终定向复核报告

- 批次：`20260927-T02-QODER-003`（T02 第三轮，达轮次上限，本轮后不再循环）
- 方法：只读回读 `审核任务-Qoder-T02最终复核.md`、前轮报告 `审核意见-Qoder-T02复核.md`、本轮三个受影响测试文件（`tests/test_imports.py`、`tests/test_repository.py`、`tests/test_smoke.py`）、被断言实现（`app/service/imports.py`、`app/main.py`、`app/repository/sqlite.py`）及 `架构设计.md` 第八节错误表（:176–:192）。
- 执行限制：本会话仅有文件读取工具，**未独立回算三个测试文件的 SHA-256（任务书 :5 所列固定哈希），未独立运行 pytest**。生产方声明的 `25 passed, 1 warning` 按声明记录；经逐文件清点测试函数数量（test_imports 16 个 `def test_`、test_rules 2 个、test_repository 3 个、test_smoke 5 个，合计 26，与前轮基线 24＋新增 1 例＋声明 25 的差值可由 test_smoke 第 5 例在缺 `web/dist` 时跳过（test_smoke.py:42-43）解释），声明与静态证据不矛盾。

## 一、三项闭环逐条判定

### ① 部分重合结果断言 → **接受**

- 要求（前轮报告 :24、:50）：补 `code`、`{i["case_id"] for i in issues}` 与两条不同 message 的断言。
- 落实证据：test_imports.py:167 断 `422`；:168 断 `code == "invalid_import"`；:169 断 `case_id` 集合精确等于 `{"旧题二", "新题"}`；:170 断 message 逐字等于「新旧用例标识不一致，请修正后重导」，与 imports.py:132 部分重合分支的三元取值一致；:177 断部分重合与无交集两消息不相等；:178 断无交集 `case_id` 集合为三个标识的严格集合（缺失∪多余，`old_ids ^ new_ids`，imports.py:131），:179 断无交集拒绝未污染已入库回归批次。
- 突变推演（静态）：把 imports.py:131 改为 `issues = []` → :169 由非空集合对比空集变红；把 :132 两分支消息改同一句 → :170 精确匹配变红；部分重合分支误用无交集消息 → :170 变红；反向误用（无交集取部分重合消息）→ :176 子串「先建立新用例基线」变红。三条消息均有锚定，前轮给出的最小复现路径本轮已被阻断（:177 在 :170/:176 之后为冗余加固，不构成弱点）。

### ② 统一错误中文／代码／空明细断言 → **接受**

- 要求（前轮报告 :24）：两条 422 断 `code == "invalid_import"`、404 断 `issues == []`、message 为非空中文字符串。
- 落实证据：test_imports.py:124 断三路状态码 `[422, 422, 404]`；:126 断统一键集合 `{code, message, issues}`；:127 以 Unicode 区间 `\u4e00–\u9fff` 断 message 含汉字，与 main.py:35「请求字段无效或缺失」、:42「未找到指定路径」的中文消息一致；:128 断两条 422 的 `code` 均为 `invalid_import`（对应 main.py:33 处理器）；:129 断 404 `code == "not_found"`、:130 断 404 `issues == []`，与架构 :180「404 → not_found、空数组」一致。
- 突变推演（静态）：处理器消息改英文 → :127 变红；改码 → :128/:129 变红；404 携带明细 → :130 变红。前轮指出的三处弱断言全部闭环。

### ③ 批内用例标识主键约束断言 → **接受**

- 要求（前轮报告 :38，可-5）：补一条插入同批重复标识断言 `IntegrityError` 的用例。
- 落实证据：test_repository.py:37-48 先建项目与批次（:40-44），同一 `(batch_id, case_id)` 首次插入成功（:46，隐式验证可插入性），重复插入断 `sqlite3.IntegrityError`（:47-48），对应 schema `cases` 表 `PRIMARY KEY(batch_id, case_id)`（sqlite.py:35）。前轮 :32 已证明 `IntegrityError` 在 `connect()` 上下文内不被回滚封装吞掉，该 raise 语义可信。
- 口径一致性：主键约束为 SQLite 默认 BINARY 比较，与架构 :188「保留原值、重复判定按原值严格相等」及实现层 `seen` 集合严格相等（imports.py:65-68）一致，数据库与代码两条防线语义对齐。非规范 Base64URL 路径段解码属 T03 接口面，不计入 T02 缺陷。

## 二、本轮测试改动是否引入阻断 → **未发现阻断**

- 新增 `已接纳基线` 辅助函数（test_imports.py:133-143）直接对 `baseline_events` 插行并改绑 `projects.current_baseline_id`，列名与 sqlite.py:37-45 schema 一致，`invalidated` 取默认 0，与 imports.py:122-125 的配对读取条件（`invalidated=0`）吻合；用例均先 201 再操作，DB 路径统一 `tmp_path`＋`AI_EVAL_DB`，无共享状态污染。
- test_imports.py:159、:165 用字典展开追加新键，不改动 `样例文档()` 原对象；:161 深拷贝后仅改 `answer`，验证了「改回答不影响配对资格」的方向，与架构 :192 摘要边界一致。
- 前轮次要残留「`criteria` 数组内顺序敏感未测」（前轮 :26）经回读认定**实际已覆盖**：test_imports.py:190 对含两要点的用例做 `reversed` 后摘要不等，直接证明要点顺序敏感；前轮该句与其自身引用的 :178-180 证据矛盾，予以更正。`version_label` 排除仍仅由 `basis_digest` 签名（imports.py:103-109）保证、无 API 层断言——维持非门禁。
- test_smoke.py:42-43 新增显式跳过条件，是对前轮增-5（干净副本全量测试依赖未提交 `web/dist`）的合规缓解；附带说明：`web/dist` 存在时 `/api/v1/unknown` 由挂载的 StaticFiles 兜底 404（main.py:73-75），其响应仍须命中自定义 JSON 处理器，:122 用例可作环境侧回归哨兵，生产方声明本机 25 例通过说明该路径当前成立，不新增门禁。

## 三、最终结论

**T02 可结项。** 前轮第五节要求的三项 T03 提审前闭环项全部闭环且判定为接受；本轮测试改动未见引入阻断；增量核查更正一处前轮笔误（要点顺序实际已测）。非门禁遗留维持前轮口径：主处理器 405 的 `message`/`code` 未逐字锚定（仅状态码，test_smoke.py:29）、403 处理器 `issues` 未断空（test_smoke.py:15-20 仅断 `code`）、`version_label` 无 API 层摘要断言、增-2～增-4 的口径缺口，连同本轮说明一并留待 T03/T05/T06 各阶段处理。本轮为第 3 轮，达上限，不再继续循环；后续 T02 范围内的争议以本批次报告为准。

批次：20260927-T02-QODER-003 ｜ 状态：已完成 ｜ 判定：接受
