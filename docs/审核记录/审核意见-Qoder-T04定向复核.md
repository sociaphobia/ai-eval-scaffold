任务和必读文件已全部回读完毕。以下为 T04 定向复核报告。

---

# T04 定向复核报告（重-1 / 建-3 关闭确认）

- 批次：`20260927-T04-QODER-002`
- 审核对象：`tests/test_regressions.py`、`tests/test_reports.py` 新增断言，及首轮报告重-1、建-3 两项的闭环核对；服务代码按任务书固定范围（首轮 `审核清单-Qoder-T04.json` 哈希、Codex 已复算 13 项不符为零）回读确认。
- 审核方式：只读回测，未修改任何项目文件。

## 一、审核限制声明

1. **哈希未独立回算**：本会话工具集仅暴露 `Read`，无 Bash 等执行工具，无法对任务书所列两个测试文件 SHA-256（`f9b26c50…bc3f`、`94b2a961…da3`）独立回算，以 Codex 回算为准。本次报告结论仅适用于当前磁盘回读到的文件内容。
2. **测试未独立运行**：无法执行 `python.exe -m pytest -q tests/test_regressions.py tests/test_reports.py`，生产方 `11 passed, 1 warning` 证据标"未独立运行"，以下均为静态源码判定。
3. 旧报告结论未自动延伸：对服务代码的判断基于本次回读的当前内容，逐行核对了 `case_ids.py`、`regressions.py`、`reports.py`、`batches.py`。

## 二、重-1 闭环核对：接受

**新增断言**：`tests/test_regressions.py:56-66` `test_回归逐题非规范路径令牌返回未找到`，对 `PUT /api/v1/regression-batches/{batch_id}/cases/6aKY5LiA=/decision` 断言 `404` 且 `code == "not_found"`（65-66 行），与基线侧 `test_baselines.py:59-66` 对称，令牌同为填充 Base64URL `6aKY5LiA=`。

**静态验证链条**（本次独立核对，非引用首轮结论）：

- `6aKY5LiA=` 含 `=`，不匹配 `case_ids.py:8` 的 `TOKEN_PATTERN`（`[A-Za-z0-9_-]+\Z`），`decode_case_id`（`case_ids.py:16-17`）先抛 `not_found`；即便绕过正则，`base64.b64decode(..., validate=True)`（19 行）亦会因非法字符抛 `binascii.Error` → `not_found`（22 行）；即便再绕过，回环校验 `encode_case_id(case_id) != token`（23 行）仍拦下。三层防御，任一层都返回 404。
- 该令牌宽松解码恰为"题一"，批次在回归侧存在（`regressions.py:114-115` 的查库会命中）。因此**若突变删除 `regressions.py:108` 的 `decode_case_id` 守卫并改走宽松解码，该 PUT 将返回 200 而非 404，新测试必红**——断言具有实际守卫力，非"恰好也 404"的假覆盖。
- 测试前置路径成立：仅 `version_label` 变化的回归批次导入后经 `rebind_pending_regressions`（`regressions.py:130-134`）保持 `pending`，`save_regression_decision` 的 404 必然来自令牌解码（`regressions.py:108` 先于 112 行的状态检查），归因唯一。

**残留条件**：首轮重-1 要求中"共享编码向量断言（`HANDOFF.md:48`）在 T05 前落实"部分不属本次两个测试文件范围，未在本次核对，须随 T05 既计划项继续跟踪，不构成 T04  reopening 理由。

## 三、建-3 闭环核对：接受

**新增断言**：`tests/test_reports.py:108-128` `test_两个项目的批次和报告列表互不混入`，覆盖任务书三个核对重点之二：

- **基线批次不混入**（113-114 行）：两项目各自 `baseline-batches` 列表精确等于自身批次 ID 的单一元素列表；若过滤缺失，另一项目列表非空即红。
- **回归批次不混入**（119-120 行）：项目二 `regression-batches` 为空列表，项目一精确等于新批次。
- **报告列表不混入**（127-128 行）：项目一 `reports` 精确等于新报告 ID 列表，项目二为空。

**实现侧回读**：`batches.py:15-17` 按 `project_id + kind` 过滤且先校验项目存在（13-14 行）；`reports.py:52-58` 按 `project_id` 过滤。断言与实现语义一致，无误判路径（`准备项目` 经 `tmp_path` 隔离数据库，`test_regressions.py:11`；导入返回 201、改绑收敛 `pending` 均与首轮已审行为一致，本次回读未新设防）。

## 四、其余核对重点与限制

- 任务书三个核对重点（404 令牌、基线/回归批次不混入、报告列表不混入）均已由上述两条新测试覆盖，未发现新增测试自身存在误判或漏断言。
- 首轮建-1（`busy_timeout`）、建-2（边际消息误导）、建-4（归属确认）不在本次复核范围，维持"并入 T05/T06、不阻断"口径。
- 因无法独立运行测试与回算哈希，若 Codex 回算发现文件与任务书哈希不符，本判定随固定范围失效。

## 五、判定

- 重-1：**接受**（守卫断言已落地且经突变推演确认有效）。
- 建-3：**接受**（双项目三类列表断言完整）。
- **T04 可以结项**：首轮唯一重要项已闭环，测试门禁完整。**可以进入 T05**，T05 周期内须完成：共享编码向量断言（`HANDOFF.md:48`）、建-1/建-2 处理安排、以及开发计划既定的 T06 突变验证项不被顺延遗漏。

批次：20260927-T04-QODER-002 ｜ 状态：已完成 ｜ 判定：接受

