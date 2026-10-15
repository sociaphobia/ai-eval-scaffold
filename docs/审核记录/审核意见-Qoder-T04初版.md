所有必读文件已回读完毕。以下为 T04 独立逐条审核报告。

---

# 审核意见-Qoder-T04（回归与报告）

- 批次：`20260927-T04-QODER-001`
- 审核对象：`审核清单-Qoder-T04.json` 所列 13 个文件（清单自述 SHA-256 `9687a22c…0363`）
- 审核方式：只读回测，未修改任何项目文件

## 一、审核限制声明

1. **哈希未独立回算**：本会话工具集仅暴露 `Read`（可确认：本会话未暴露 Bash/Glob/Grep 等执行与检索工具），无法运行 SHA-256 命令，清单中各文件哈希与自身哈希均标"未独立回算"，按任务约定由生产方审核后逐项复算。
2. **测试未独立运行**：无法执行 `python.exe -m pytest -q`，生产方证据 `55 passed, 1 warning` 标"未独立运行"，仅以回读的测试源码与断言内容作静态判定。
3. 前置结论（T02 第三轮接受、T03 第二轮接受）只作为背景，以下均为对当前文件的独立核对。

## 二、逐条审核结果

### 阻断项

无。

### 重要项

**重-1：T04 自测遗漏"回归逐题路径非法令牌 404"断言**

- 出处：`开发计划.md:100` 明确要求 T04 自测包含"逐题路径非法令牌 404；项目下的回归批次与报告列表按所属项目过滤"。
- 现状：基线侧已有 `tests/test_baselines.py:59-66` 覆盖非规范令牌 `6aKY5LiA=` → `404/not_found`；但 `tests/test_regressions.py` 全文（42–161 行）没有任何对 `PUT /api/v1/regression-batches/{batch_id}/cases/{token}/decision` 非法令牌的断言。实现层 `app/service/regressions.py:108` 确实先调 `decode_case_id(token)`，机制与基线同源（`app/service/case_ids.py:15-25`），但回归路由该行为目前无测试守卫。
- 最小复现：将 `regressions.py:108` 的 `decode_case_id(token)` 改为直接 `token`（即接受任意令牌、查库后报 404——恰好 `not_found` 也是 404，语义偏差在"填充令牌被静默宽容匹配"的路径上；若改成 `base64.b64decode` 宽松解码则完全无守卫），现有测试全绿。
- 影响：编码契约（`架构设计.md:193`）的回归侧一致性仅靠与基线共用函数间接保证，违反计划"前后端共享一份书面编码样例及接口测试，不能各自定义不同语义"的守卫要求。
- 要求：在 `tests/test_regressions.py` 补一条与 `test_baselines.py:59-66` 对称的断言（非规范/填充令牌 → `404/not_found`），并在 T05 前一并落实 `HANDOFF.md:48` 已记录的共享编码向量断言。

### 普通建议（不阻断）

- **建-1（并发写可用性）**：`app/repository/sqlite.py:71` 未设 `busy_timeout`，SQLite 默认约 5 秒。`create_report`（`app/service/reports.py:10`）与 `accept_baseline`/`invalidate_baseline`（`app/service/baselines.py:66,111`）用 `BEGIN IMMEDIATE` 正确互斥，但并发下第二请求会收到原生 `sqlite3.OperationalError: database is locked` → HTTP 500，而非统一错误结构。单进程单用户界面下窗口极小，且不会造成错误放行（见三-7），属体验与一致性缺口，建议 T05/T06 处理。
- **建-2（边际消息误导）**：批次绑定基线后、改绑前，若生产方直接重接纳同一接纳事件导致其失格（罕见交错），`reports.py:18-23` 的基线查询不过滤 `invalidated`，此时报告门禁返回的 message 是"评测依据变化后须先重新评测并接纳基线"，与真实原因（所引用事件已失效）不符；放行判断本身仍正确（同时被 `status='paused'` 与 `baseline_id != current_baseline_id` 拦截），仅提示语偏差。
- **建-3（跨项目过滤实测缺失）**：`reports.py:52-57` 与 `batches.py:11-19` 均按 `project_id` 过滤，实现正确；但 T04 测试均单项目运行，"报告列表按所属项目过滤"的断言未实测。建议 T05/T06 补双项目断言。
- **建-4（无交集提示消息）**：`imports.py:133-136` 的"没有可配对标识，请先建立新用例基线"专项 message 属开发计划可-3 门禁，T04 测试未覆盖，按任务口径归属 T02；登记为归属确认而非缺陷。

### 必核项逐条对照（任务书第 8 条）

| 必核点 | 结论 | 证据 |
| --- | --- | --- |
| 同标识输入变化仍须新基线 | ✅ | `imports.py:137-138`（摘要含 `input`，见 `basis_digest` 103-109）；导入即 `paused`；确认接口因摘要不符不可续接（`regressions.py:88-98`）；报告门禁双查（`reports.py:15-23`）；测试 `test_regressions.py:56-84` 断言 `paused`、差异非空、报告 409、指针未变 |
| 要点/规则变化复用旧回答后逐题重评 | ✅ | `regressions.py:56-58` 用当前 criteria 对基线回答重跑规则与线索（符合需-05"两版运行相同规则"，`架构设计.md:90`）；正向全流程测试 `test_regressions.py:87-127` |
| 新基线匹配才改绑、清旧决定与确认；不匹配暂停 | ✅ | `regressions.py:123-137`：标识集合＋`basis_digest` 双条件，改绑时 `decision/reason` 清空且 `basis_confirmed=0`；不匹配仅置 `paused`、保留旧 `baseline_id`；`accept_baseline` 在同一事务内调用（`baselines.py:100-102`）；测试 113-118 行断言改绑后 `regressed is None`、`basis_confirmed is False`、报告仍 409；不匹配维持暂停见 145-161 |
| 项目方依据确认是否真持久记录 | ✅ | `basis_confirmations` 表逐条追加（`sqlite.py:46-52`），`regressions.py:100-103` 持久写入，GET 取最新记录并回显理由（42-45、70 行）；测试 `test_regressions.py:142` 断言理由可回读 |
| 资格异常及未全确认阻止报告 | ✅ | `reports.py:17`（`require_qualified`）、24-29（未全确认/依据未确认 409）；测试 `test_reports.py:86-105` 直接改库注入失真后报告 409 且列表为空；`test_regressions.py:41-53` 覆盖未确认拦截 |
| 规则线索与人工结论分离 | ✅ | `get_regression_batch` 的 `clues`/`rule_facts` 与 `regressed`/`reason` 独立字段（`regressions.py:50-61`）；线索仅作题级提示（`rules.py:18-21`），符合需求第五节"多要点只影响线索、人工只记一个题级结论" |
| 三数按题计数 | ✅ | `reports.py:30-31`，与 `架构设计.md:191` 示例一致；`test_reports.py:26-28` 实测 `2/1/1` |
| 报告保存后不随当前指针或旧事件失效改变 | ✅ | 报告不可变快照存 `snapshot_json`（`reports.py:33-46`），`get_report` 只读快照（73-77）；`test_reports.py:38-64` 新基线接纳后报告仍引用旧事件；67-84 失效后报告与 `reported` 批次保持终态 |
| 未报告与已报告批次在失效时不同 | ✅ | `baselines.py:126-128` 仅暂停 `status!='reported'`；两条路径分别由 `test_baselines.py:148-149` 与 `test_reports.py:82` 断言 |
| 跨项目列表 | ✅ 实现正确 | `reports.py:53-57`、`batches.py:15-17`；断言缺口见建-3 |
| 测试能否抓住错误放行 | ⚠️ 总体能 | 改绑、未确认、资格损坏、指针漂移均有端到端断言（见上表）；唯一守卫缺口即重-1 的回归令牌 404；关键突变验证按计划属 T06 |
| 状态、事务与并发交错 | ✅ 无确定性缺陷 | 详见下节 |

### 状态／事务／并发专项核对

1. 状态集合 `pending/paused/reported`（回归）与 `pending/accepted/invalidated`（基线批次）与 `架构设计.md:173` 一致；导入、确认、改绑、失效、报告各迁移路径互洽。
2. 写事务边界正确：`accept_baseline`、`invalidate_baseline`、`create_report` 均 `BEGIN IMMEDIATE` 并在首条语句取写锁；`connect()` 上下文异常回滚（`sqlite.py:75-81`）；T03 已有触发器回滚实证（`test_qualification.py:28-41`）。
3. 交错推演：导入与接纳竞态下，即使批次先以旧事件 `pending` 落库，报告仍被 `baseline_id==current_baseline_id`、`basis_digest` 相等、`basis_confirmed` 与逐题全覆盖四重门禁拦截（`reports.py:12-29`），不会产生错误放行；确认接口同事务重算 `can_resume`，交错收敛为 `paused`。
4. 重复报告由 `status!='pending'` 拒绝（`reports.py:12-13`），已报告批次禁止改决定与改确认（`regressions.py:81-82,112-113`）。

### 错误语义与契约一致性

- `422/invalid_import`（请求体结构）、`409/workflow_conflict`（门禁与理由空白）、`404/not_found`、统一 `{code,message,issues}` 与 `架构设计.md:179-189` 的 T03 后裁定一致；回归接口未新增任何契约外字段或路径。
- 需-02/03/05/06/08/09/12 与验收 2/3/4/7 的后端行为均能对应到上述实现与测试证据，未见与需求正文冲突的自拟口径。

## 三、判定

**修正性接受**。

- 阻断项：无；
- 重要项 1 项（重-1：回归逐题非法令牌 404 断言缺失，须补测试后方可视为 T04 测试门禁完整）；
- 普通建议 4 项（建-1 至建-4），可并入 T05/T06 处理，不阻断。

**能否开始 T05：可以开始**。后端接口、状态机与报告契约已稳定且与已审架构一致；重-1 的补测与建-3 的双项目断言建议与 T05 既定的编码一致性向量断言（`HANDOFF.md:48`）一并在 T05 周期内落地。本报告结论仅适用于本次回读的当前文件内容；因未能独立回算哈希与运行测试，若复算发现清单与磁盘不符，本判定随清单失效。

批次：20260927-T04-QODER-001 ｜ 状态：已完成 ｜ 判定：修正性接受
