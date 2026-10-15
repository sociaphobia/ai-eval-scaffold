# T02 初版独立代码审核任务

- 批次：`20260927-T02-QODER-001`。用户 2026-09-27 09:51 明确要求改由 Qoder 协作。
- 项目工作区：`C:\Users\YUAN\Desktop\个人项目\AI应用评测脚手架`。仅只读审查，不修改任何文件或向外发送材料。
- 目标：独立审查 T02 初版的 SQLite 仓储、严格 UTF-8 JSON 导入、字面规则及项目列表接口，找真实缺陷和与需求／架构契约不一致之处。
- 红线：以 `AGENTS.md`、`需求文档.md`、`架构设计.md` 第三至八节、`开发计划.md` T02 为权威；金融项目只读技术参照，不复制金融业务。审查只针对当前工作区，原报告位于本项目目录。
- 应读代码：`app/main.py`、`app/model/errors.py`、`app/repository/sqlite.py`、`app/service/imports.py`、`app/service/rules.py`、`app/api/projects.py`、`app/api/batches.py`；对应 `tests/test_imports.py`、`tests/test_rules.py`、`tests/test_repository.py`、`tests/test_smoke.py`。
- 生产方当前证据：Python 3.13 全量当前测试 `15 passed, 1 warning`，命令为 `python.exe -m pytest tests/test_imports.py tests/test_rules.py tests/test_repository.py tests/test_smoke.py -q`。审核方若无命令能力，标“未独立运行”，不得虚构。
- 必核：字段额外值、类型与格式版本优先；非标准 JSON、重复／空标识、规则字符串；整批事务与外键；回归配对与依据摘要；无基线 409；错误响应结构；Host／Origin；业务测试是否遗漏会导致错误放行的路径。只对能用文件行号或实测复现证明的问题列阻断／重要；可选改善另列。
- 交付：中文报告，按“接受／修正性接受／驳回”判定；每项问题给文件行号、最小复现与影响，说明是否允许 T03 在修正期间继续；末尾标记 `批次：20260927-T02-QODER-001 ｜ 状态：已完成 ｜ 判定：<判定>`。
