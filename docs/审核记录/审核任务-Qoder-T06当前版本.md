# Qoder 独立审核任务：T06 当前版本

## 审核目标

请独立审查 AI 应用评测脚手架 T06 的当前实现和验收证据，判断其是否符合需求与架构、恢复核验是否能避免错误放行、浏览器 E2E 与请求记录是否自证。请给出“接受 / 修正性接受 / 驳回”之一，并逐项列出证据与文件行号。不要只复述生产方验收结论。

用户已授权在 Qoder 限免期按需不限轮次复核，生产方会逐条修正并重新提交直至审核条件关闭。生产方不得自审；Qoder 仅承担独立审查，不替用户裁定两 agent 之间的重大事实争议。

## 项目范围与红线

- 项目：`C:\Users\YUAN\Desktop\个人项目\AI应用评测脚手架`。
- 先读 `AGENTS.md`、`README.md`、`HANDOFF.md`、`需求文档.md`、`架构设计.md`、`开发计划.md`、`docs/验收记录.md` 和 `docs/审核意见-Qoder-T06第一轮.md`、`docs/审核意见-Qoder-T06第二轮.md`、`docs/审核意见-Qoder-T06第三轮.md`。
- 评测样例均为自造夹具；不能据此宣称五类真实样例或业务效果已验收。
- 金融项目只作只读技术参考；本轮没有对其进行修改，也不能把它的业务依赖或接口复制成本项目契约。
- 接口、状态码和字段以本项目定稿需求及架构为准。提交发现不符合时列明来源，不自行提出新的对外契约。
- 审核只读；不要改写、移动或删除任何输入文件。请将审核意见作为最终回复文本返回，不要创建文件。
- 每项阻断或建议须绑定本页列出的具体 SHA-256；不要把对旧哈希的历史报告套用到当前版本。

## 真实验收证据

- 后端：Python 3.13 `pytest -q -rs`，`60 passed, 1 warning`；警告为 Starlette TestClient 的 `httpx` 弃用提示。
- 前端单元测试：`npm --prefix web test`，3 项通过。
- 前端生产构建：`npm --prefix web run build`，退出码 0；Vite 报告 JS 分块 647.02 kB，超过 500 kB 建议阈值。
- Playwright 端到端：`npm --prefix web run test:e2e`，1 项通过；测试启动隔离 FastAPI 服务并上传两份自造 JSON，运行期采集器直接更新本任务清单内的请求记录和截图，结束时清理本轮 SQLite 主文件及 `-wal`、`-shm`、`-journal` 辅助文件，并逐项断言清理结果。
- 浏览器请求记录：`docs/T06浏览器请求记录.json`，47 个请求、47 个响应、0 个非本机请求，HTTP 状态为 43 个 200、4 个 201；记录含复现命令与生成者，浏览器报告截图为 `docs/T06浏览器报告.png`。
- 恢复测试：停服复制 SQLite 文件到隔离路径，检查 `integrity_check`、`foreign_key_check`、当前基线资格，并从恢复副本继续处理回归批次。
- 突变验证：临时移除 `app/service/reports.py` 的 `require_qualified` 调用，资格异常集成测试观察到报告错误地返回 201；按原字节恢复，源文件 SHA-256 回到 `98676a5130b0f0acc9d6bbf887eeb72df06ece4757ab7f45e5b2c8477ac68808`，测试复绿。
- 数据库只读核验工具：`scripts/核验数据库.py`；本轮实跑当前基线通过，另验证缺失路径返回码 2 且不创建文件。
- Playwright 隔离库清理：运行前后 `data/build-temp/` 内符合 `playwright-e2e-*` 的文件均为 4 个，没有本轮新残留；E2E 中另外创建三种侧文件样例并断言主库及侧文件均已删除。
- 前端构建：若本机 Windows 临时目录拒绝 esbuild 清理，令 `TMP` 与 `TEMP` 指向项目忽略目录 `data/build-temp/` 后执行 `npm --prefix web run build`，退出码 0；JS 647.02 kB，仍有 Vite 500 kB 优化提示。

以上均为生产方提供的命令输出和落盘证据，请结合源文件核查其覆盖边界，不把请求记录或截图当作业务真实样例。

## 审查文件固定哈希

| 文件 | SHA-256 | 字节数 |
| --- | --- | ---: |
| `AGENTS.md` | `81ff076a592e506decd974caf462c58ba248fcc0e490af10088c7e2f216caf03` | 1127 |
| `.gitignore` | `3cb34d0138d54b2b7d695bb09f947a95956cd7be76b80a79c450aaed7ab08b83` | 475 |
| `需求文档.md` | `819e23bf2de43f0d1ec3f0c2821a474fe84027326847affcb3f6739d6b0cba11` | 25785 |
| `架构设计.md` | `38d9464f98086cf715afedf7900dbddf4f7f355754cc67fca37bfebd4820cd0f` | 26209 |
| `开发计划.md` | `030a3866a5cb142bff67a266bc3daf0ca857df5598f6727eb6db97c1c8623a14` | 23347 |
| `README.md` | `651511d5dd32da0a83c05ae2aef9204146b19836921045ca353f885fd0379968` | 4984 |
| `app/service/reports.py` | `98676a5130b0f0acc9d6bbf887eeb72df06ece4757ab7f45e5b2c8477ac68808` | 3652 |
| `app/service/qualification.py` | `92a17b45f483446a13c36f46ace4bf2191a25714c832bca7d839d8de874f3696` | 4386 |
| `tests/test_full_flow.py` | `2882148855d67973866350397087d8a0e83511c77d23b15b2c3189d39713ccfb` | 5705 |
| `tests/test_restore.py` | `634bbc7119052ff43e223399c670d289b8435613f249d5137593f0704dcbd011` | 2531 |
| `scripts/核验数据库.py` | `424d6ce3105d541a8667f99ea13325df47116a2ff50469a7ac5500d7838e6a3a` | 1910 |
| `tests/fixtures/演练规则误报基线.json` | `7df2e32a05b418fde8ab0b5730f1038b6ccb902e6b1b45679963e83e6013321a` | 598 |
| `tests/fixtures/演练规则误报回归.json` | `7bbc00bf3093e3db6b0e016a6af4f3b0c285a1844d39aa883e53568d2120a598` | 622 |
| `web/package.json` | `66eb75cb94ed03d441ec52b9bd2e4f50b44585ec7f2284acca59a64707be8bcb` | 623 |
| `web/package-lock.json` | `b30b224fe76816ec237f217ab7b432f66ddd242fcc9b41c1efa8a050e09a0770` | 111978 |
| `web/playwright.config.ts` | `c7681b8a24b8a5bfb47a6cb5431fcc8aab7da62e9216fe3d9d004a4fe78f2174` | 1154 |
| `web/e2e/workflow.spec.ts` | `5d2a48a4361ee34fa14949db727d6b5a8218fd2391a5e1a747d0f23a78824886` | 6160 |
| `docs/验收记录.md` | `bf3c0810c0e375e81d9147331965c5aac7984f1f239b7f844edb1363698435f2` | 7165 |
| `docs/T06浏览器请求记录.json` | `0da0274978c9789dd57d55e302c9db7d7abe33d613d09965bf92629d860a18ca` | 14659 |
| `docs/T06浏览器报告.png` | `fd94edecb82b3216182ea3aafa86a1471b89e7a709f9c8384d4675779ce387d7` | 173196 |
| `docs/审核意见-Qoder-T06第一轮.md` | `3d431403d753396ed9ba5785a71a8446ee08f128c2673e3060874ff26d76b035` | 3308 |
| `docs/审核意见-Qoder-T06第二轮.md` | `b3973f80ad25c52d0c9cec78b619084048fdacca3b9ee3c5d6b21ac02d21caef` | 2160 |
| `docs/审核意见-Qoder-T06第三轮.md` | `4737d083aa4d7d9ff41187f108c20239d4c36573afd8ea74fced67d39decc773` | 1594 |

## 验收标准

1. 独立核对需求、架构、计划、服务实现、测试和恢复工具，找出会造成错误报告、数据损失、错误放行或虚假验收的具体缺口。
2. 检查恢复工具的只读性、路径缺失处理、外键和当前基线资格边界；判断测试是否对应真实 SQLite 状态。
3. 检查浏览器测试的文件来源、用户操作、断言、请求采集方式与日志一致性；判断“0 个非本机请求”是否被证据充分支持。
4. 检查 README 命令与锁定依赖、配置和实际测试流程是否一致。
5. 审核意见按“接受 / 修正性接受 / 驳回”分类，每一项给文件路径、行号、严重程度和修正建议。若没有缺陷，也请写“无发现”并给出关键依据。
6. 明确项目方五类真实样例尚缺，不能将其视作生产方待修代码缺陷或已验收业务效果。

## 回复格式

```text
审核判定：接受 / 修正性接受 / 驳回
审查版本：上表 23 项固定哈希
重要问题：无，或逐项列出严重程度、文件:行、事实与建议
一般问题：无，或逐项列出
接受依据：列出核对过的关键路径及证据
验收边界：明确仍未取得的项目方真实样例
审核身份与工具边界：Qoder；只读输入，未运行命令／计算哈希（若事实如此）
```
