# T03 首轮审核处理

原报告 `审核意见-Qoder-T03初版.md` 判修正性接受。本表只记录生产方修正与本机验证，不给自己判独立通过。

| 意见 | 处理及理由 | 可复核位置 |
| --- | --- | --- |
| 重-1 失效暂停回归零覆盖 | 建立真实接纳事件与未报告回归批次，失效后断批次状态为 `paused` | `tests/test_baselines.py` 的 `test_失效暂停未报告回归而重复和未知失效返回约定错误` |
| 重-2 重复失效 409、未知 404 无测试 | 同一用例分别断状态码与业务 `code` | 同上 |
| 重-3 失效后重评再接纳全链无测试 | 旧事件失效、指针清空，再导入新候选、逐题确认、接纳并核新事件资格 | `tests/test_baselines.py` 的 `test_失效后可重新评测整批接纳并恢复指针` |
| 重-4 畸形请求体按 409 与结构错误口径冲突 | 代理裁定：缺字段、字段类型错、额外字段统一 `422/invalid_import`；字段齐备但理由空白或流程受阻为 `409/workflow_conflict`。已写回架构第八节，代码与测试同步 | `架构设计.md`“T03 审核后的请求校验裁定”；`app/service/baselines.py`；`tests/test_baselines.py` 的畸形请求用例 |
| 轻-1 pending 规则记录损坏可 500 | 捕获无效 JSON 并返回 `409/workflow_conflict`，不把损坏记录用于人工决定 | `app/service/baselines.py` 与候选规则损坏用例 |

2026-09-27 本机 Python 3.13 实跑 `tests/test_baselines.py tests/test_qualification.py tests/test_case_ids.py`：`21 passed, 1 warning`。完整全量将于二轮复核前后复跑并记录。Qoder 首轮在静态计数中推测可能有 `1 skipped`，此为无 `web/dist` 情况下的条件推断；当前实际构建产物存在，先前全量实跑输出为 `42 passed, 1 warning`、无跳过。
