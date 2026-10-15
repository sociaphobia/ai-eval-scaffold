# Qoder 对 T06 当前版本的首轮独立审核

## 审核身份与边界

- 审核方：Qoder CN，模型 `Qwen3.8-Flash`。
- 会话：`ee70d5b8-3320-4b06-b737-eebe5cb49001`；退出码 `0`；审核时长约 8 分 29 秒。
- 输入：`审核任务-Qoder-T06当前版本.md` 固定的 20 项 SHA-256 文件清单。
- Qoder 声明：逐项通读 20 项，并回读相关实现与测试交叉验证；只读，没有运行命令，没有重算 SHA-256，也没有改动输入文件。
- 生产方独立门禁：派发前清单与工作区哈希一致（20/20）。报告只对当时冻结的版本成立。

## 审核判定

**接受。重要问题：无。**

Qoder 判断报告门禁链由服务端资格检查、依据摘要核对、依据确认、逐题确认和事务内报告写入组成；资格核验对真实 SQLite 状态 fail-closed；恢复工具只读打开数据库且缺失文件不会新建空库；全流程浏览器 E2E 使用项目自造夹具并核验报告、响应状态和请求 origin；README 的命令及配置与实现相符。自造夹具只支持机制验收，不能证明业务规则效果。

## 低级意见与生产方处理

| 序号 | Qoder 观察 | 证据 | 处理 |
| --- | --- | --- | --- |
| 1 | 验收记录把 URL origin 写成 HTTP `Origin` 头，而 JSON 未记录请求头。 | 原 `docs/验收记录.md:24`；浏览器记录仅存请求 URL、方法及类型。 | 改为“所有请求 URL 的 origin”，限定证据表达；由 E2E 直接生成请求记录。 |
| 2 | 请求 JSON 是从 Playwright 测试附件整理的留档副本，含手工元数据，没有复现命令，存在副本漂移风险。 | `web/e2e/workflow.spec.ts` 使用 `testInfo.attach`；`web/test-results/` 被忽略。 | Playwright 测试现从运行期监听器直接写入 `docs/T06浏览器请求记录.json` 与截图，并写入生成者、夹具和复现命令。 |
| 3 | Playwright 隔离库使用 PID 命名，原配置不会清理，反复运行会留下忽略目录里的 SQLite 文件。 | 原 `web/playwright.config.ts` 数据库路径；`.gitignore` 忽略 `/data/build-temp/`。 | 增加 Playwright 配置元数据传递本轮路径，`test.afterAll` 只删除该轮数据库及 SQLite 辅助文件。实跑前后目录均为 5 个既存文件，没有新增残留。 |
| 4 | 验收记录中的测试数、Chromium 和 TypeScript 版本未由 Qoder 回源实测；源码逐文件计数与声明相符。 | Qoder 未运行命令；测试函数计数合计 60，向量测试 3。 | 不把 Qoder 的静态计数说成独立复跑；生产方运行日志作为可复核证据保留。 |

## Qoder 指出的业务验收边界

项目方真实五类样例（规则通过、失败、误报、漏报、无规则）尚未提供。本轮夹具为自造；误报/漏报比例、规则覆盖率和实际业务效果没有验收，也不能从机制测试推出。

## 复核所用官方资料

首轮审核未联网、未运行命令。生产方处理 Playwright 生命周期和配置元数据时核对了官方文档：

- [Playwright 全局设置与清理](https://playwright.dev/docs/next/test-global-setup-teardown)
- [Playwright 配置 API（metadata）](https://playwright.dev/docs/api/class-testconfig)
- [Playwright Web Server](https://playwright.dev/docs/test-webserver)
