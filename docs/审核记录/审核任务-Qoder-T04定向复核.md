# T04 定向复核任务

- 批次：`20260927-T04-QODER-002`。用户指定 Qoder 作为独立审核方；只读项目，不修改任何文件。
- 目标：复核 `审核意见-Qoder-T04初版.md` 的重-1 与建-3 是否通过实际新增断言关闭，并判断 T04 能否结项、进入 T05。项目红线遵循 `AGENTS.md`、`需求文档.md` 与 `架构设计.md`；不可把旧报告对旧文件的结论自动延伸到新文件。
- 必读：首轮审核报告、`tests/test_regressions.py`、`tests/test_reports.py`、`app/service/case_ids.py`、`app/service/regressions.py`、`app/service/reports.py`、`app/api/batches.py`。
- 固定范围：服务代码仍为首轮 `审核清单-Qoder-T04.json` 中的哈希，已由 Codex 复算 13 项不符数为零；仅测试文件变化。`tests/test_regressions.py` 当前 SHA-256 `f9b26c50aaab1011ead190fc3bf2979555d862102c6cffa199d35c041e00bc3f`，`tests/test_reports.py` 当前 SHA-256 `94b2a9612d468bb3cbd55d1d9e6005f18e8fb2ebe45961b1582dba976ae29da3`。审核后 Codex 再回算。
- 生产方实跑证据：`python.exe -m pytest -q tests/test_regressions.py tests/test_reports.py` 得 `11 passed, 1 warning`；若 Qoder 没有执行工具，需标记未独立运行。
- 核对重点：填充的 Base64URL 回归路径令牌返回 `404/not_found`；两个项目的基线批次、回归批次与报告列表没有相互混入。若新增测试自身存在误判或漏断言，应以文件行号说明。
- 交付：中文报告，逐项接受／修正性接受／驳回，列出证据和限制，说明 T04 结项及 T05 开始与否。末尾必须为 `批次：20260927-T04-QODER-002 ｜ 状态：已完成 ｜ 判定：<判定>`。
