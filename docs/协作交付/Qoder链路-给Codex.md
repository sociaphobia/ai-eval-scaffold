# Qoder 侧调用方式（转给 Codex）

- 建立时间：2026-09-24｜由 Qoder 独立审核方实测并撰写
- 已实测：Qoder CLI CN **1.1.62**，`-p` 非交互出答、`--tools ""` 禁全部工具、`--list-sessions` 可枚举会话 UUID、`-c` 续接能答对上一轮内容
- 未实测（不要当能力用）：`--output-format` 的取值、`remote-control`／`--teleport`／`--remote`、细粒度工具白名单（`--allowed-tools` 等）、Codex 沙箱内跑本 CLI 的写权限

## 0. 前提

CLI 与桌面版**不共用登录态**。桌面版用户已完成 `qoderclicn login`（实测 `Login Method: browser`、`Auth Source: local`）。若你侧报 `Not logged in`，说明登录态没落到位，直接回报，不要自己尝试登录。

## 1. 必须先跑的冒烟测（一条命令，不落任何文件）

```bash
EXE="$HOME/.qoder-cn/bin/qoderclicn/qoderclicn.exe"
env -u QODER_AGENT_SDK_ENTRYPOINT -u QODERCN_CLI \
  "$EXE" -p --tools "" -- "只回复四个字：已连通"
```

期望输出：`已连通`。

- `env -u QODER_AGENT_SDK_ENTRYPOINT` **不可省**：agent 会话里该变量被注入（值 `sdk-ts`），子进程会强制要求 `--print --input-format stream-json --output-format stream-json`，缺一即 `sdk_invalid_args` 崩溃。
- 当前终端可能解析不到 `qoderclicn`（PATH 未刷新），一律用上面的绝对路径。
- **风险点**：CLI 会把会话状态写到 `~/.qoder`（不是项目目录）。若你的沙箱只允许写项目目录，冒烟测可能因写不进会话文件而失败——失败就把报错原文回报，由用户裁定要不要放宽，**不要为此改配置或加 `--dangerously-skip-permissions`**。

## 2. 派活模板（把任务发给一个新的 Qoder 会话）

```bash
EXE="$HOME/.qoder-cn/bin/qoderclicn/qoderclicn.exe"
PROJ="C:/Users/YUAN/Desktop/个人项目/AI应用评测脚手架"
env -u QODER_AGENT_SDK_ENTRYPOINT -u QODERCN_CLI \
  "$EXE" -w "$PROJ" -n "<本轮批次号>" --permission-mode default -p -- \
  "<任务包正文，见第 4 节>"
```

要点：

- `-n <批次号>` 给会话命名，便于事后 `--list-sessions` 找到 UUID 再 `-r` 续接。
- `-w` 锁到项目目录，别用系统 Temp。
- 默认保留人工审批语义（`--permission-mode default`）。**禁止**使用 `bypass_permissions`／`dont_ask`／`--dangerously-skip-permissions`。
- 回复走 stdout，重定向进你自己的文件即可留证。

## 3. 会话寻址与续接

```bash
env -u QODER_AGENT_SDK_ENTRYPOINT -u QODERCN_CLI "$EXE" --list-sessions      # 列本项目会话与 UUID
env -u QODER_AGENT_SDK_ENTRYPOINT -u QODERCN_CLI "$EXE" -c  -p -- "<追问>"    # 续最近会话
env -u QODER_AGENT_SDK_ENTRYPOINT -u QODERCN_CLI "$EXE" -r <id> -p -- "<追问>" # 按 id 续
```

这起的是**独立 Qoder 会话**，不是往用户当前打开的桌面版会话里注消息。要与人机对话界面互斥看待。

## 4. 回执约定（把「送达」变成可验证的事实）

每个任务包必须含六项，缺一项即视为未送达：

1. 批次号：`20260924-PRD-Qoder-处理`
2. 唯一交付文件的绝对路径（只准新建这一个，见第 5 节红线）
3. 验收标记：交付文件末尾必须写
   `批次：20260924-PRD-Qoder-处理 ｜ 状态：已完成 ｜ 逐条判定：<N> 项`
4. 逐条引用审核项编号（B-1／I-1…）与**该条原文要点**——用来证明真读过，而不是照抄结论
5. 客观门禁：发起方回读交付文件，校批次号一致、判定条目数与审核项数一致
6. 验收通过后**续发一次停止消息**：「只回复已完成，不要再调用任何工具」

## 5. 本轮的具体任务（红线照此执行）

- 你是生产方。审核方是 Qoder，报告已交付：`C:\Users\YUAN\Desktop\个人项目\AI应用评测脚手架\审核意见-Qoder.md`
- 审核对象钉版：草案 0.11｜163 行｜sha256 `6b4d310478ae6d7ad5d7d3753aa17148f0fc32bef9af5334f1d405db17654bf1`。行号只对该哈希有效，动手前先复算哈希确认稿件没变；变了先重新核对再判。
- 待办：逐条判定该报告的 **B-1（阻断）＋ I-1～I-5（重要）＋ O-1～O-4（可选）**，只新建 `审核处理-Qoder.md`。
- **红线**：不改 `需求文档.md`、不改 `README.md`。`HANDOFF.md` 只允许回写状态位（补记本轮独立审核的批次、审核方、报告路径；第 83-86 行仍停在「第 3 轮」）。B-1（可信备份归属）与报告第六节列出的 9 项归用户裁定，**不得自拟口径、不得替用户选定处理策略**。I-1／I-2／I-4 属生产方可直接补齐的完整性缺陷，无需等新裁定。
- 完成即停，不自行发起下一轮审核。
