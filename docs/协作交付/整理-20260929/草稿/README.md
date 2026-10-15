# AI 应用评测脚手架

本项目在本机保存 AI 应用输入、评测依据、旧版基线、逐题人工结论和历史报告。评测数据只发往本机服务；规则结果用于提示，最终结论由评测人员确认。

T06 的全流程、停服复制恢复和浏览器验收证据见 [验收记录](docs/验收记录.md)；前端视觉选择与实测见 [前端视觉改版记录](docs/前端视觉改版记录.md)。当前自动化样例均为项目自造数据；项目方五类真实样例尚未提供，业务效果验收仍未完成。

## 文档入口

- [文档索引](docs/文档索引.md)：当前权威入口与历史证据入口的总导航。
- [需求文档](需求文档.md)：首版需求的唯一正文。
- [架构设计](架构设计.md)：系统分层、接口契约和运行边界。
- [开发计划](开发计划.md)：阶段任务和验收映射。
- [项目协作说明](AGENTS.md)：项目范围和协作约束。
- `docs/审核记录/`：历次审核任务、审核意见、审核处理与补充材料。历史证据，逐文件绑定其审核时的冻结哈希，不代表当前状态。
- `docs/协作交付/`：协作任务单、链路记录与 T05/T08 执行报告。
- `HANDOFF.md`：本地协作交接文件，按约定由 Git 忽略。

## 环境要求

- Windows 与 PowerShell 7。
- Python 3.13；依赖锁定在 `requirements.lock`。
- Node.js 满足 Vite 7 要求，即 20.19+ 或 22.12+；版本要求见 [Vite 7 官方迁移说明](https://v7.vite.dev/guide/migration)。
- 可访问 npm 注册表以安装前端依赖。

## 安装与构建

在项目根目录打开 PowerShell：

```powershell
py -3.13 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.lock
npm ci --prefix web
npm --prefix web run build
```

前端构建产物写入 `web/dist/`，由 FastAPI 同源提供。第一次运行浏览器 E2E 时，按锁定的 Playwright 版本安装其 Chromium：

```powershell
npm --prefix web exec -- playwright install chromium
```

## 启动与停止

在项目根目录启动服务。服务只监听本机回环地址：

```powershell
$env:AI_EVAL_DB = (Join-Path (Get-Location) 'data\evaluation.sqlite3')
$env:AI_EVAL_PORT = '8000'
.\.venv\Scripts\python.exe -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

在浏览器打开 `http://127.0.0.1:8000`。需要停止服务时，在运行 Uvicorn 的窗口按 `Ctrl+C`，等窗口显示服务已退出后再备份或恢复数据库。不要把监听地址改成 `0.0.0.0`。

## 测试

```powershell
.\.venv\Scripts\python.exe -m pytest -q
npm --prefix web test
npm --prefix web run test:e2e
npm --prefix web run build
```

浏览器 E2E 会单独启动本机服务，使用 `data/build-temp/` 下的隔离数据库，并在测试结束时清理本轮数据库及 SQLite 辅助文件。每次成功的运行都会用浏览器运行期采集数据更新 `docs/T06浏览器请求记录.json` 和 `docs/T06浏览器报告.png`。可通过 `AI_EVAL_PYTHON` 指定 Python 3.13 可执行文件；默认使用项目 `.venv`。

## 停服备份

1. 在 Uvicorn 窗口按 `Ctrl+C`，确认进程已经退出。
2. 将 `data/evaluation.sqlite3` 复制到受项目方管理的备份位置；备份目录应与工作目录分开，并由项目方按其数据管理要求保护。
3. 记录副本日期和 SHA-256，定期进行隔离恢复演练。

示例命令会提示备份目录，不预设用户的实际备份盘符：

```powershell
$backupDirectory = Read-Host '请输入受管理的备份目录'
New-Item -ItemType Directory -Force -Path $backupDirectory | Out-Null
$backupFile = Join-Path $backupDirectory ("evaluation-{0}.sqlite3" -f (Get-Date -Format 'yyyyMMdd-HHmmss'))
Copy-Item -LiteralPath 'data\evaluation.sqlite3' -Destination $backupFile
Get-FileHash -Algorithm SHA256 -LiteralPath $backupFile
```

当前 `app/repository/sqlite.py` 未开启 WAL，停服后按单一 SQLite 文件复制；若后续实现改用 WAL，不能把主数据库与 `-wal` 状态分开复制。SQLite 官方说明见 [WAL 文档](https://www.sqlite.org/wal.html) 和 [在线备份 API](https://www.sqlite.org/backup.html)。

## 恢复与核验

1. 停止本机服务；把当前数据库另存一份，便于回退。
2. 将可信副本复制到 `data/evaluation.sqlite3`。
3. 在项目根目录对恢复文件运行只读核验；检查底层完整性、外键和每个项目当前基线的业务资格：

```powershell
$env:AI_EVAL_DB = (Join-Path (Get-Location) 'data\evaluation.sqlite3')
.\.venv\Scripts\python.exe .\scripts\核验数据库.py
```

只有输出中的 `"passed": true` 且退出码为 `0`，才启动服务继续评测。该脚本以只读方式打开数据库，不会创建缺失的数据库文件。`PRAGMA integrity_check` 不检查外键，因此核验也执行 `foreign_key_check`；业务基线再由资格核验函数逐项检查。SQLite 对应说明见 [PRAGMA 文档](https://www.sqlite.org/pragma.html)。

核验未通过时保持服务停止，不生成正式报告。备份不可用时，依照需求在脚手架内重新评测候选旧版回答，整批合格后重新接纳基线。

## 数据边界

项目和演练数据默认保存在 `data/evaluation.sqlite3`。不要把真实项目数据复制到 issue、公共仓库或非项目方批准的外部服务。`tests/fixtures/` 中含“演练”字样的 JSON 均为自造样例，只用于机制验证。

## 文档整理状态

本节记录 2026-09-29 的文档整理**草稿**状态，本文仍是 `docs/协作交付/整理-20260929/草稿/README.md` 内的候选，尚未落地到项目根目录：

- 计划把根目录与 `docs/` 下的历史审核文件归入 `docs/审核记录/`，协作调度记录归入 `docs/协作交付/`；移动只改文件位置与本文档入口，不改任何审核正文。
- 上节“文档入口”里指向 `docs/文档索引.md` 与 `docs/审核记录/`、`docs/协作交付/` 的条目描述的是落地后的位置；草稿阶段这些目录与索引文件还不存在于项目实际路径。
- 移动映射与引用调整清单见 `docs/协作交付/整理-20260929/整理报告.md` 与同目录 `整理映射.json`。
- 本次整理为 Qoder 生产草稿：未运行任何测试、命令、构建或服务，未重算哈希，不表述为整理已完成或独立审核已通过。落地动作与落地后的核验记录由协调器在核对通过后另行新增。
