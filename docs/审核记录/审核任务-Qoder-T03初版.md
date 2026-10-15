# T03 基线接纳与资格核验独立审核任务

- 批次：`20260927-T03-QODER-001`。用户已要求转由 Qoder 协作。只读审核，不修改文件。
- 目标：逐条审查 T03 是否实现需求中的首版逐题评测、整批接纳、误报理由、基线资格异常整批暂停、错误接纳失效及重新评测路径，并核验 T02 集成没有倒退。
- 权威资料：项目 `AGENTS.md`；`需求文档.md` 第三节、需-01/02/08/09、验收 1/2/7；`架构设计.md` 第三至八节；`开发计划.md` T03；T02 终审 `审核意见-Qoder-T02最终复核.md`。金融项目只读技术参照，不是本轮业务依据。
- 新对外契约：`架构设计.md` 第八节补充 `POST /api/v1/baselines/{baseline_id}/invalidate`，系 Codex 按用户持续授权代理裁定，以满足需求第八节“查实错误接纳后暂停并重新评测接纳”。须独立审查路径、字段、状态、历史保留与原契约兼容，不可将旧哈希的审核自动扩展到此增量。
- 固定对象：架构当前 SHA-256 `35e55f7bbf1bc73b8fd99d0706d125405dbdfc034defb33b91eec90d15566886`；`app/service/baselines.py` 为 `131f027e6d519c370bbf4f37f215ec0327f6f576c78fbfae9603839d2b00ae22`；`app/service/qualification.py` 为 `92a17b45f483446a13c36f46ace4bf2191a25714c832bca7d839d8de874f3696`；`app/service/case_ids.py` 为 `46479c24ae61cb3636301d036bd52a5f7016561959cba8689c3f6bcc2a15cb38`；`app/api/baselines.py` 为 `2621450766b639dd07a034c0edc91bdca04bbe473d94685311d6a769ccb47519`。无哈希工具时标未独立回算。
- 必读代码：上面四个 T03 文件；`app/main.py`、`app/service/imports.py`、`app/repository/sqlite.py`；`tests/test_baselines.py`、`tests/test_qualification.py`、`tests/test_case_ids.py`、`tests/fixtures/case_id_vectors.json`，必要时回读 T02 测试。
- 本机实跑：2026-09-27，Python 3.13 `python.exe -m pytest -q` 得 `42 passed, 1 warning`；Qoder 若不能运行，必须标“未独立运行”。
- 必核：接纳前原始材料与规则事实核对；全合格门禁与误报理由；事件＋指针＋状态原子提交；恢复或材料损坏时业务资格判定而非 500；任何单题失格如何暂停整批但保留其他题排查；失效后旧事件、当前指针和未报告批次状态；重复操作与错误响应；路径令牌编码向量；测试是否有只镜像实现却未检查风险的用例。给每项阻断或重要问题的文件行号、最小复现与影响。
- 报告采用“接受／修正性接受／驳回”，说明 T04 可否开工。末尾必须是 `批次：20260927-T03-QODER-001 ｜ 状态：已完成 ｜ 判定：<判定>`。
