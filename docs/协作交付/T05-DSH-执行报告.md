# T05-DSH 执行报告（批次 20260927-T05-DSH-001）

- 执行方：DSH `deepseek-flash`；协调方：Codex；最终独立复核方：Luna。
- 会话 ID：`session-84be70d4-b78a-4af1-b7ec-f2a2f3ae7abc`（`$env:DSH_SESSION_ID`，同一 ID 见运行日志 `data/dsh-T05.log` 首行）。
- 项目根目录：`C:\Users\YUAN\Desktop\个人项目\AI应用评测脚手架`。仅在本项目内操作。
- 执行时间：2026-09-27 21:16–21:2x（本机时间）。环境：Node.js v24.19.0、Python 3.13.14（项目 `.venv`）、pytest 9.1.1。
- 任务包：`协作任务-DSH-T05校验.md`。

## 零、固定输入哈希复算（门禁）

四份固定输入全部与任务包一致，门禁通过，未停止。

| 文件 | 任务包固定值 | 本轮实测 | 结果 |
| --- | --- | --- | --- |
| `web/src/App.tsx` | `39db9378eb29240c02736f373bb3b6e919715a2010f5005b464cf87b32091ec0` | 同左 | 一致 |
| `架构设计.md` | `7b16ce21e43afea27bc37b149b5ed02ba544993a5afa1099763558b387bd4d1a` | 同左 | 一致 |
| `app/service/baselines.py` | `54df6a80ce58749686fec2c6c22de211698d740a2ec903cc8d0b1443a538af15` | 同左 | 一致 |
| `tests/test_baselines.py` | `c760cb7239a228416f6f06e42b28c3008c7586ffecf144276650e69a679a0e3c` | 同左 | 一致 |

## 一、执行结果：三条指定命令全部因权限失败

### 1.1 `npm --prefix web test` → 退出码 1，0 项通过

原始输出（摘录，未删改）：

```
npm warn Unknown user config "electron_mirror". This will stop working in the next major version of npm.
npm warn Unknown user config "electron_builder_binaries_mirror". This will stop working in the next major version of npm.

> ai-app-evaluation-web@0.1.0 test
> node --test tests/*.test.mjs

✖ tests\caseId.test.mjs (3.1092ms)
ℹ tests 1  ℹ suites 0  ℹ pass 0  ℹ fail 1
ℹ cancelled 0  ℹ skipped 0  ℹ todo 0  ℹ duration_ms 10.985

✖ failing tests:
test at tests\caseId.test.mjs:1:1
✖ tests\caseId.test.mjs (3.1092ms)
  Error: spawn EPERM
      at ChildProcess.spawn (node:internal/child_process:458:11)
      at spawn (node:child_process:813:9)
      at TestContext.<anonymous> (node:internal/test_runner/runner:517:19)
      errno: -4048, code: 'EPERM', syscall: 'spawn'

EXITCODE=1
```

警告 2 条（npm 用户配置 `electron_mirror`、`electron_builder_binaries_mirror` 已废弃）。**没有取得任何用例通过数**；失败发生在 `node --test` 派生测试文件子进程时，不是断言失败。

### 1.2 `npm --prefix web run build` → 退出码 1

原始输出（摘录，未删改）：

```
> ai-app-evaluation-web@0.1.0 build
> tsc -b && vite build

failed to load config from C:\Users\YUAN\Desktop\个人项目\AI应用评测脚手架\web\vite.config.ts
error during build:
Error: spawn EPERM
    at ChildProcess.spawn (node:internal/child_process:458:11)
    at Object.spawn (node:child_process:813:9)
    at ensureServiceIsRunning (...\web\node_modules\esbuild\lib\main.js:2272:29)
    at build (...\web\node_modules\esbuild\lib\main.js:2170:26)
    at bundleConfigFile (...\web\node_modules\vite\dist\node\chunks\config.js:35895:23)

EXITCODE=1
```

可确证的分段事实：`tsc -b && vite build` 中 `vite build` 已开始执行，说明 `tsc -b` 退出码为 0 且未输出诊断；失败点在 Vite 加载 `web/vite.config.ts` 时由 esbuild 派生服务进程。**未生成构建产物**。

### 1.3 后端全量测试（`.\.venv\Scripts\python.exe -m pytest -q`）→ 退出码 1，14 passed / 43 errors

原始汇总行：

```
14 passed, 3 warnings, 43 errors in 8.82s
EXITCODE=1
```

- **通过数：14**。**错误数：43**，全部发生在 **setup 阶段**。
- 43 个错误的异常完全相同：

```
E   PermissionError: [WinError 5] 拒绝访问。:
    'C:\Users\YUAN\AppData\Local\Temp\dsh-0TCJxj\pytest-of-YUAN'
.venv\Lib\site-packages\_pytest\pathlib.py:175: PermissionError
    for x in os.scandir(root):
```

  抛出链：`tmp_path` → `_pytest/tmpdir.py:300 _mk_tmp` → `:287 factory.mktemp` → `:136 mktemp` → `:117 _ensure_relative_to_basetemp` → `:213 getbasetemp` → `_pytest/pathlib.py:420 make_numbered_dir_with_cleanup` → `:401 make_numbered_dir` → `:228 find_suffixes` → `:187 extract_suffixes` → `:175 os.scandir`。会话收尾时同一异常再次出现于 `_pytest/tmpdir.py:337 cleanup_dead_symlinks` → `pathlib.py:354 root.iterdir()`。
- 受影响用例（错误清单，按原输出的类名）：`tests/test_baselines.py` 10 项、`tests/test_imports.py` 13 项、`tests/test_qualification.py` 5 项、`tests/test_regressions.py` 6 项、`tests/test_reports.py` 5 项、`tests/test_repository.py` 3 项、`tests/test_smoke.py` 0 项。凡使用 `tmp_path` 夹具的用例全部在建立临时目录时即报错，未进入测试体。
- 警告 3 条：`fastapi/testclient.py:1` 的 `StarletteDeprecationWarning: Using httpx with starlette.testclient is deprecated; install httpx2 instead`；`_pytest/cacheprovider.py:469` 与 `:423` 的 `PytestCacheWarning: could not create cache path ...\.pytest_cache\v\cache\nodeids`（及 `lastfailed`），两条均为 `[WinError 5] 拒绝访问`。
- 错误信息中的中文在控制台为乱码（`�ܾ����ʡ�`），属本机控制台代码页显示问题，异常类型与 WinError 5 明确。

## 二、归因：权限失败的确切原因（用最小探针分离变量）

### 2.1 前两条命令：确认是「以管道 stdio 派生子进程」被禁止

| 探针 | 命令 | 结果 |
| --- | --- | --- |
| A | `node -e "spawn(process.execPath,['-e','...'],{stdio:'pipe'})"` | `Error: spawn EPERM`，`errno -4048`，退出码 1 |
| B | 同 A，仅把 `stdio` 改为 `inherit` | 子进程输出 `v24.19.0`，`CLOSE 0`，退出码 0 |

结论：阻断精确落在「Node 以**管道** stdio 派生子进程」这一操作上（`node --test` 的测试运行器与 esbuild 的服务进程都使用该方式），与项目源码、依赖版本或断言内容无关。这是本执行环境的已知边界，不是构建失败或测试失败。

### 2.2 第三条命令：**不是沙箱策略，而是 `Path.mkdir(mode=0o700)` 在本环境生成的目录不可访问**

| 探针 | 操作 | 结果 |
| --- | --- | --- |
| C | `.venv` Python：`Path('.').mkdir(mode=0o700)` 后 `os.scandir` / `rmdir` | `MKDIR_700_OK` → **`SCANDIR_700_FAIL PermissionError [WinError 5]`** → **`RMDIR_700_FAIL PermissionError [WinError 5]`** |
| D | 同一 Python：`Path('.').mkdir()`（默认 mode）后 `os.scandir` / 写文件 / 删除 | 全部成功（在 `$env:TEMP` 与项目工作区内均成功） |
| E | PowerShell：`New-Item` 建目录 + `Get-ChildItem` + `Remove-Item` | 全部成功 |

C 与 D 的唯一差别是 `mode=0o700`：**本环境中用该 mode 建出的目录，创建它的进程自身既无法列举也无法删除**。pytest 的临时目录机制固定使用 `mode=0o700`（`_pytest/pathlib.py:175/187/228/401/420`、`_pytest/tmpdir.py:117/136/213/287/300/337`），因此所有 `tmp_path` 用例在 setup 阶段必然失败。

为验证这一点，另做了两次辅助重跑（见 §三 辅助命令），更换 `--basetemp` 到 `$env:TEMP` 下新目录与项目工作区内新目录，均为完全相同的 `PermissionError`，与 §1.3 一致。

**归属未分离**：这两个探针都经同一执行环境，故我无法在本环境内判定该 ACL 现象最终源自执行环境的受限令牌／ACL 处理，还是本机 Python 3.13 在 Windows 上的 `mkdir(mode=...)` 行为。可分离的判据是：在同一台机器、同一 Python 3.13、**不经本执行环境**的普通 shell 中复跑探针 C。C 的三行代码与结果见上表，可直接复现。**无论归属如何，都不涉及本项目源码。**

### 2.3 沙箱状态与权限参数的遵守情况

- 本轮文件沙箱策略为 `workspace-write`。**未使用任何提权或绕过权限的参数**（未使用 `sandbox_permissions`），全部命令按任务包要求原样保留原始错误。
- 依据任务包「若沙箱阻断命令，保留原始错误，不用绕过权限参数」，§1.1 与 §1.2 的失败按原样记录，不作为绿灯，也不替代后续复测。

## 三、辅助复现（非任务包指定命令，**不替代** §1 的结果，不构成任何门禁通过）

为把「命令被环境阻断」与「代码是否有问题」分开，另跑了两条不派生子进程的最小命令：

| 辅助命令 | 结果 | 可覆盖范围 |
| --- | --- | --- |
| `node web/tests/caseId.test.mjs`（进程内直接执行测试文件，不经 `node --test` 派生运行器） | 退出码 0；`ℹ tests 3 / pass 3 / fail 0`；逐项 ✔ `题一`、✔ `中文/题一`、✔ ` a ` | 仅说明 `tests/fixtures/case_id_vectors.json` 的 3 条向量在当前 `web/src/caseId.ts`（`690b49ee…`）下成立。**不等于** `npm --prefix web test` 通过 |
| `node node_modules/typescript/bin/tsc -b`（`web/` 下） | 退出码 0，无诊断输出 | 仅说明类型检查部分通过。**不等于** `npm --prefix web run build` 通过（Vite/esbuild 段仍未跑） |

另两次 pytest 辅助重跑（`--basetemp` 指向 `$env:TEMP` 新目录、指向项目内 `.dsh-pytest-tmp`）均在 setup 阶段得到同样的 `PermissionError`，退出码 1，与 §1.3 结论相同，未提供新增通过数。

## 四、`changed_files`

### 4.1 有意新建（唯一交付文件）

- `docs/协作交付/T05-DSH-执行报告.md`（本报告）
- `docs/协作交付/`（为放置本报告而新建的目录）

### 4.2 非预期新建、**清理未成功**（须由外部处理）

| 路径 | 来源 | 状态 |
| --- | --- | --- |
| `.dsh-pytest-tmp\`（项目根内） | §三 的 pytest `--basetemp` 辅助复跑 | **存在，无法删除** |
| `modeprobe-d01c46\`（项目根内） | §2.2 探针 C | **存在，无法删除** |

两个目录都是在复现 `mkdir(mode=0o700)` 现象时生成的；按 §2.2，创建进程自身也无权列举或删除它们。已尝试并**全部失败**（均为 `Access is denied` / `[WinError 5] 拒绝访问`）：

- `Remove-Item -LiteralPath <dir> -Recurse -Force`
- `cmd /c "rmdir /s /q <dir>"`
- `cmd /c "takeown /f <dir> /r /d y"` → `Successfully processed 0 files; Failed processing 1 files`
- `cmd /c "icacls <dir> /grant yua n:(F)"`、`icacls <dir> /setowner <user>`、`icacls <dir> /reset /t /c` → 均为 `Access is denied`
- 分别对两个目录重复上述全部尝试

我未再次尝试提权删除（遵守任务包不使用绕过权限参数的要求）。**待外部处置**：请由有权权限的会话删除这两个目录；若 §2.2 的 ACL 现象已排除，普通 `rmdir /s /q` 即可。

### 4.3 由本执行环境自动写入（非我指定）

- `data/dsh-T05.log`：DSH 运行日志，首行记录本会话 ID。被 `.gitignore` 的 `*.log` 覆盖。

### 4.4 未被我修改

`web/src/`、`app/`、`tests/`、`需求文档.md`、`架构设计.md`、`开发计划.md`、`HANDOFF.md`、`README.md`、`AGENTS.md`、`docs/` 内既有文件、夹具：**一律未写入**。`web/dist` 的 mtime 为 2026-09-27 21:12:45，早于本次会话开始（约 21:16），**不是本轮产物**，我未对其内容与当前 `App.tsx` 的一致性作任何断言。

## 五、`regressionId` 切换清空依据说明：静态核对

### 5.1 代码事实（行号）

| 行号 | 内容 |
| --- | --- |
| `web/src/App.tsx:89` | `const [basisReason, setBasisReason] = useState("");` |
| `web/src/App.tsx:102` | `const loadRegression = useCallback(async (id) => setRegression(await 接口.回归详情(id)), []);` → 依赖为空数组，引用稳定 |
| `web/src/App.tsx:107` | `useEffect(() => { setBasisReason(""); if (regressionId) loadRegression(regressionId).catch(...) }, [regressionId, loadRegression]);` |
| `web/src/App.tsx:105` | 切换项目：`setRegressionId("")`（同时 `setRegression(null)`） |
| `web/src/App.tsx:113` | 导入新回归批次：`setRegressionId(created.batch_id)` |
| `web/src/App.tsx:158` | 回归批次 tab 点击：`setRegressionId(item.batch_id)` |
| `web/src/App.tsx:167` | 说明文本框 `value={basisReason}`（受控） |
| `web/src/App.tsx:168` | 「确认当前依据」按钮 `disabled={... \|\| !basisReason.trim()}` |

### 5.2 判定

1. **就 `docs/T05浏览器演练记录.md:20-23` 记录的残留而言，修复在覆盖面上成立。** 该残留的触发路径是「已报告批次的说明框有文字 → 导入 input 变化的回归文件 → 新批次仍显示旧文字」，而导入新批次必然经 `:113` 改变 `regressionId`，从而触发 `:107` 清空。所有会切到另一批次的入口（`:113` 导入、`:158` 切 tab、`:105` 换项目）都改变 `regressionId`，无遗漏入口。
2. **不引入「输入被误清」的新问题。** `:107` 的依赖为 `[regressionId, loadRegression]`，而 `loadRegression` 由 `useCallback(…, [])` 定义（`:102`），引用跨渲染稳定；因此该 effect 只在 `regressionId` 真正变化时运行，普通重渲染不会清掉用户正在输入的文字。
3. **附带消除了一个门禁提示被削弱的问题。** 修复前，残留文字会让新批次的「确认当前依据」按钮（`:168` 依赖 `basisReason.trim()`）处于可点状态；清空后新批次重新要求填写理由。该结论由代码推出，**未在浏览器实测**。
4. **无任何自动化测试覆盖该修复。** `web/tests/` 仅有 `caseId.test.mjs`（编码向量），不存在组件／DOM／端到端测试；§1.1 的前端测试命令本轮也未取得绿灯。因此该修复目前**只有静态论证，没有测试或浏览器证据**。

### 5.3 同类残留观察（仅静态分析，未改任何代码）

1. `web/src/App.tsx:118`：`invalidateBaseline` 成功后只调用 `loadProjects/loadLists/loadBaseline`，**未调用 `loadRegression(regressionId)`**；而 `regressionId` 不变，`:107` 的 effect 不会重跑。结果是：已被服务端置为 `paused` 的回归批次，在「检查新版」页仍沿用旧的 `regression.status`，`:159` 的状态标签与 `:161` 的暂停提示都不会出现，与 `:118` 的 notice（「未出报告的回归批次已暂停」）不一致。服务端门禁仍是权威（`架构设计.md:107` 的实现补充裁定、`开发计划.md:107`「按钮禁用仅是提示，服务端门禁仍是权威」），故**不构成错误放行**。
   - 精确改法（未执行）：把 `:118` 的 `Promise.all([loadProjects(), loadLists(projectId), loadBaseline(baselineId)])` 改为在 `regressionId` 非空时一并加入 `loadRegression(regressionId)`。
   - 受影响文件：仅 `web/src/App.tsx`。所需测试：一条浏览器用例（失效当前基线后切到回归页，应看到「已暂停」）或一条组件测试。
2. `web/src/App.tsx:88` 的 `invalidateReason` 只在成功路径 `:118` 清空；`:181` 的 Modal `onCancel` 只做 `setInvalidateOpen(false)`，取消后再次打开会沿用上次文字。同类：`:85` 的 `name` 在 `:180` 的 `onCancel` 时不清空。严重度低（同项目内，无跨项目串味，且 Modal 的确认按钮仍以非空校验为条件）。
   - 精确改法（未执行）：`:181` 改为 `() => { setInvalidateOpen(false); setInvalidateReason(""); }`；`:180` 改为 `() => { setCreateOpen(false); setName(""); }`。
   - 受影响文件：仅 `web/src/App.tsx`。所需测试：组件测试或浏览器用例「取消后重开，输入框为空」。

## 六、版本边界：哪些旧审核结论不能覆盖当前哈希

### 6.1 T05（Luna 两轮）边界：变化面只有 `web/src/App.tsx`

Luna 两轮报告（`审核意见-Luna-T05初版.md:19`、`审核意见-Luna-T05复核.md:14`）把 `web/src/App.tsx` 钉在 `dcb43f60b9d091771ed850f78d9934bdf30a00f8b0298eb9a09c6441d2df32e5`；当前为 `39db9378…`（mtime 2026-09-27 21:03:49，晚于两份报告 20:51:54 / 20:55:43）。

我复算了 Luna 表格中其余全部条目，**逐项与她的记录一致**（下列哈希为我本轮实测）：

| 文件 | 实测 SHA-256 | 与 Luna 表 |
| --- | --- | --- |
| `web/src/api/http.ts` | `92e764e94b4a8d05101d0606aea1cdcaa34ff0e581f8ddd059b817b12bfeab54` | 一致 |
| `web/src/caseId.ts` | `690b49ee53b6387fcc9667ce018da6209b39fc7b1f61b2b5ef7feed20bbe68e2` | 一致 |
| `web/src/main.tsx` | `748dd8def2cc2cf825546e776e6c59bffeca39bdadc4f4b78de38935abfc18f3` | 一致 |
| `web/src/style.css` | `91abb7e93f15abece4538f3d4d66e09c796f7028b2e554218cda71ffd695cb80` | 一致 |
| `web/src/style-extra.css` | `712b986c2127deec58c9d9e6f297e50952ac50370d90504cd1f3071638db2e3b` | 一致 |
| `web/src/vite-env.d.ts` | `65996936fbb042915f7b74a200fcdde7e410f32a669b1ab9597cfaa4b0faddb5` | 一致 |
| `web/tests/caseId.test.mjs` | `77a2cd99a970a044862fbf9cd64e7e54c17ed66e80afe83f6fd12ee126334a77` | 一致 |
| `web/vite.config.ts` | `0f20570cc1839025e84f0346c604cd4d95436195599f0b8b5dac85934d8aa08c` | 一致 |
| `app/main.py` | `c78fb9ccb061dd5e1f6d24356c4245bbcf0ebdb1fe9d8cca0951e531c36f9be5` | 一致 |
| `tests/fixtures/演练基线.json` | `36a8f84c7e8e0bde785893014b09b0fd511413a0f9fdb58772b8be47b379ff03` | 一致 |
| `tests/fixtures/演练回归.json` | `c2adbf7a96ac4e26460b2a2ef5ee630ec5fcb40faaca019ac3aabac2f24a59e5` | 一致 |
| `需求文档.md` | `819e23bf2de43f0d1ec3f0c2821a474fe84027326847affcb3f6739d6b0cba11` | 一致 |
| `架构设计.md` | `7b16ce21e43afea27bc37b149b5ed02ba544993a5afa1099763558b387bd4d1a` | 一致 |
| `开发计划.md` | `75602834ed9cedd4c18b9f7bc5047564bd8ffa239de2b257e253169742e53455` | 一致 |

结论：

- **不能覆盖当前哈希的**：一切依赖 `App.tsx` 行为的判定，即 Luna 报告的主流程、错误／暂停展示分支（她引用 `web/src/App.tsx:19-25,139,148,160-168`）以及由这些分支支撑的 T05 验收项。她两轮的驳回结论对应旧哈希，须对 `39db9378…` 重测。
- **可以继续沿用的**（因为文件逐字节相同，不是推断）：Luna 对这些文件的静态判定，例如 `web/src/style.css` 的 900px／560px 响应式规则与长回答断行、`web/src/api/http.ts` 的相对路径 `/api/v1`（当前 `:42`）、`app/main.py` 的 Host／Origin 校验。这些结论只覆盖其被引用的静态范围，不覆盖运行期行为。
- 上述「可沿用」是**跨报告复用他人结论**的记录方式，不等于我本轮复核了这些行为；本轮未打开浏览器。

### 6.2 T04 边界：三份文件在定向复核之后被改，T04「接受」不覆盖当前哈希

T04 首轮固定清单为 `审核清单-Qoder-T04.json`（mtime 10:50:09）。我实测的对照：

| 文件 | 清单固定值（T04 受审） | 本轮实测 | 结论 |
| --- | --- | --- | --- |
| `架构设计.md` | `22fef528d4ab920e926748bc5a0b32ed0058d6b47a3086b82a5d05e43aec087c` | `7b16ce21…4d1a` | **已变化** |
| `app/service/baselines.py` | `842ae21c6aa5d7a101ecbdec688baf9125404f91895a2408ee6970e8c1a710f3` | `54df6a80…af15` | **已变化** |
| `tests/test_baselines.py` | `59c026f4ac8e656ec49c53bdc6a3b403403853ce2bb1f1908c04247c8cd23807` | `c760cb72…0e3c` | **已变化** |
| `tests/test_regressions.py` | `600b18ff79144dc713ce3e0b91d487d522aec396f38204b10fea03c1701cd544` | `f9b26c50aaab1011ead190fc3bf2979555d862102c6cffa199d35c041e00bc3f` | 变化发生在 T04 **定向**阶段：等于 `审核意见-Qoder-T04定向复核.md:13` 的钉版 `f9b26c50…bc3f` |
| `tests/test_reports.py` | `05608b494a6017e50281b067d441c422286b834872d8af0804837b75defd5159` | `94b2a9612d468bb3cbd55d1d9e6005f18e8fb2ebe45961b1582dba976ae29da3` | 同上：等于该报告钉版 `94b2a961…da3` |
| `app/main.py` | `c78fb9ccb061dd5e1f6d24356c4245bbcf0ebdb1fe9d8cca0951e531c36f9be5` | 同左 | 未变化 |

清单其余 7 项（`app/repository/sqlite.py`、`app/service/imports.py`、`app/service/regressions.py`、`app/service/reports.py`、`app/api/regressions.py`、`app/api/reports.py`、`tests/test_qualification.py`）本轮**未复算**，不作结论。

时间序（文件系统 mtime，同一台机器）：

```
10:50:09  审核清单-Qoder-T04.json                ← T04 首轮固定对象
10:54:19  审核意见-Qoder-T04初版.md
11:15:46  审核任务-Qoder-T04定向复核.md
11:17:26  审核意见-Qoder-T04定向复核.md          ← 判「接受，T04 可以结项」
11:25:28  app/service/baselines.py               ← 三份文件在报告后约 8 分钟同批被改
11:25:31  架构设计.md
11:25:39  tests/test_baselines.py
```

因此：**T04 定向复核的「接受」绑定 `22fef528`／`842ae21c`／`59c026f4`；当前 `7b16ce21`／`54df6a80`／`c760cb72` 不受该结论覆盖。** `审核意见-Qoder-T04定向复核.md:43` 自身也写明「若 Codex 回算发现文件与任务书哈希不符，本判定随固定范围失效」。

### 6.3 三份文件的变化范围：可确证的部分与不可确证的部分

**可确证（按行号位移交叉校验）**

- 审查方对自己受审版本的行号引用是精确的：`审核意见-Qoder-T04定向复核.md:19` 称 `tests/test_regressions.py:56-66` 为填充令牌 404 用例、断言在 `:65-66`；我实测该文件（哈希未变）`:56` 为 `def test_回归逐题非规范路径令牌返回未找到`、`:65-66` 为两条断言，**逐行一致**。这把该方法论校准为可用。
- `app/service/baselines.py`：`审核意见-Qoder-T04初版.md:46,52` 引用其受审版本 `:66`、`:100-102`、`:111`、`:126-128`；当前对应 `:67`（`accept_baseline` 的 `BEGIN IMMEDIATE`）、`:101-103`（`rebind_pending_regressions`）、`:112`（`invalidate_baseline` 的 `BEGIN IMMEDIATE`）→ **净增 1 行**，插入点在 `:66` 之前。当前 `:43` 恰为单行的请求体结构校验（`set(payload) != {"qualified","reason"} or type(payload.get("qualified")) is not bool or not isinstance(payload.get("reason"), str)` → `invalid_import`），与位移量自洽。
- `tests/test_baselines.py`：`审核意见-Qoder-T04初版.md:28` 引用填充令牌用例 `:59-66`、`:52` 引用 `:148-149`；当前分别对应 `:61-68`（`test_非规范路径令牌返回统一未找到`）与 `:150-151` → **净增 2 行**，插入点在 `:59` 之前。
- 三份文件按当前内容共同指向的契约面：`架构设计.md:179`（T03 审核后的请求校验裁定）规定逐题决定与失效接口的请求体缺字段／类型错误／额外字段统一 `422/invalid_import`，字段齐备但规则误报缺理由、失效理由空白或状态不允许为 `409/workflow_conflict`；实现见 `app/service/baselines.py:43`（决定体）与 `:108-111`（失效体、空白理由 409）；测试见 `tests/test_baselines.py:113-128`（`test_畸形决定与失效请求体使用结构错误状态`）。

**不可确证（限制声明，不得当作事实）**

- 仓库**无任何提交**：`git log` 返回 `fatal: your current branch 'master' does not have any commits yet`，`git status --porcelain` 全部条目为未跟踪 `??`。工作区内无旧版本副本，因此 **11:25 那批改动的 diff 无法复原**；「插入点在哪一行、插入了什么、是否还有行内改写」均**未验证**。上述「净增 1／2 行」是行号位移量，不是 diff。
- 我**不能**断言该批改动由谁所做、意图为何、是否已另行审核，也**不能**断言 §6.3 列出的契约面就是全部变化内容。
- `架构设计.md` 自 CodeBuddy 架构 1.0「接受」后至少落盘四次：`9de1a9b5…`（见 `HANDOFF.md:100`）→ `0ae8626b…`（见 `审核任务-CodeBuddy-开发计划0.2定向复核.md:5`，且 `审核意见-CodeBuddy-开发计划0.2定向复核.md:8,31-32` 只核了第八节三行，其报告内引作 `:187/:189/:191`，当前对应 `:193/:195/:191`）→ `22fef528…`（T04 首轮对象，审查方无工具、未独立回算哈希）→ `7b16ce21…`（当前）。**当前架构哈希没有任何独立审核覆盖。**

## 七、剩余工作清单（含行号）

标记：**[阻断]** = 挡住 T05 结项；**[门禁]** = 客观门禁未取得证据；**[裁定]** = 需协调方决定。

1. **[阻断] 浏览器绿灯复测 `basisReason` 修复。** 依据 `docs/T05浏览器演练记录.md:19-23`（红灯场景）与 `:25`（未完成项）。本轮按任务包未打开浏览器，故仍未完成。
2. **[阻断] T05 验收所需的浏览器证据。** `开发计划.md:108` 要求「用浏览器从空项目走到报告」并手动／自动检查窄屏、键盘焦点、长文本、错误与暂停状态；`需求文档.md:153-156` 为验收 1–4；`审核意见-Luna-T05复核.md:44-49` 与 `:59` 逐条列出未关闭项（主流程、错误／暂停、完整键盘路径、窄屏实测、运行期请求）。注意：Luna 的驳回是**证据不足**，不是已证实代码缺陷。
3. **[门禁] 前端两条命令在本执行环境不可复现。** `npm --prefix web test`（退出码 1，`spawn EPERM`，0 项通过）、`npm --prefix web run build`（退出码 1，`tsc -b` 通过后 esbuild `spawn EPERM`），原始输出见 §1.1／§1.2，归因见 §2.1。→ 需在一个允许「以管道 stdio 派生子进程」的环境重跑，才能取得可引用的前端门禁证据。
4. **[门禁] 后端全量测试在本执行环境不可完成。** 退出码 1、`14 passed, 3 warnings, 43 errors`，43 个错误全部为 `tmp_path` 建立时的 `PermissionError [WinError 5]`；换两处 `--basetemp` 均复现；最小探针证明直接原因是 `Path.mkdir(mode=0o700)`（`_pytest/pathlib.py:175` 等）。→ 需在无该 ACL 现象的环境重跑，才能独立核验生产方记录的 `57 passed, 1 warning`。
5. **[裁定] 三份文件（`架构设计.md`、`app/service/baselines.py`、`tests/test_baselines.py`）在 T04 定向复核之后被改且无独立审核覆盖**（见 §6.2／§6.3）。→ 需补一次定向复核，或回退／书面说明该批改动来源。在此之前，T04「接受」不能延伸到当前哈希。
6. **[裁定] 规格回写缺口。** `审核意见-CodeBuddy-开发计划0.2定向复核.md:62-63` 的可选-1／可选-2 要求把前端编码一致性断言与「共享书面编码样例」载体写入 `开发计划.md:108`（T05 自测）与 `:75`（T01 新建清单）；本轮复读这两行仍无对应内容，而实际落实物已存在：`web/tests/caseId.test.mjs`（mtime 11:17:06）、`tests/fixtures/case_id_vectors.json`（09:54:44，内容为 3 条向量）、`docs/用例标识编码样例.md`（09:54:47）。但 `开发计划.md` 哈希仍是已审的 `75602834…`，回写会作废该哈希。→ 由协调方裁定「回写并重新钉哈希」还是「记录以实现物为准的书面偏离」。
7. **[裁定，非阻断] T04 遗留普通建议。** `审核意见-Qoder-T04初版.md:35` 建-1：`app/repository/sqlite.py:71` 未设 `busy_timeout`，并发写会返回 500 而非统一错误结构；`:36` 建-2：`app/service/reports.py:18-23` 的基线查询未过滤 `invalidated`，门禁 message 与真实原因不符（放行判断仍正确）；`:38` 建-4：`app/service/imports.py:133-136` 的「没有可配对标识」提示未被 T04 测试覆盖（归 T02）。
8. **[裁定，非阻断] 计划自测清单不一致。** `开发计划.md:67` 把「T02 编解码」列为必跑，但 `:86` 的 T02 自测清单无该断言（`审核意见-CodeBuddy-开发计划0.2定向复核.md:64`）。
9. **[外部条件] 真实项目方样例缺失。** `需求文档.md:158-161`、`开发计划.md:115`；现有 `tests/fixtures/演练基线.json`（`36a8f84c…`）与 `演练回归.json`（`c2adbf7a…`）均为自造样例，不能作为业务效果验收。
10. **[外部处置] 清理 §4.2 的两个残留目录** `.dsh-pytest-tmp\`、`modeprobe-d01c46\`。
11. **[待办] T06**：`开发计划.md:111-117` 的全流程、恢复演练与突变验证尚未开始。

## 八、未验证边界

1. 本轮**未打开任何浏览器**、未运行前端页面、未触发原生选文件窗口。§五 关于 `App.tsx` 的全部结论均为源码静态阅读，**不构成行为验证**。
2. **未把 `docs/T05浏览器演练记录.md` 的生产方记录当作本轮实测**；该文件 `2049234f…`（我实测）只作为被引用的记录。
3. 生产方记录的 `57 passed, 1 warning` 与「修订后标准前端构建退出码 0」本轮**均未复现**（本轮为 `14 passed + 43 errors` 与构建退出码 1）。
4. §1.1 与 §1.2 的结果**只能说明命令在本环境被阻断，不能说明代码有缺陷**；§三 的两条辅助命令**不能替代**任务包指定的命令，不构成门禁通过。
5. §2.2 的 ACL 现象**最终归属未分离**（执行环境受限令牌／ACL 处理 vs 本机 Python 3.13 Windows `mkdir(mode=...)` 行为）；分离方法已在 §2.2 给出（同一台机器、同一 Python、非本执行环境的普通 shell 重跑探针 C）。
6. §6.3 的「净增 1／2 行」是行号位移推断，**不是 diff**；11:25 批次改动的作者、意图与完整内容均未验证。
7. 未复算的 T04 清单条目见 §6.2 末段；未复算的文件不作任何结论。
8. `web/dist` 未由本轮生成，也未核对其内容与当前 `App.tsx` 是否一致。
9. 我**未修改**任何业务源码、需求、架构、开发计划、HANDOFF、README、夹具（见 §4.4）；§5.3 的两条改法只写在报告里，未落地。
10. 未使用任何提权或绕过权限的参数。

批次：20260927-T05-DSH-001 ｜ 状态：已完成 ｜ 结论：三条指定命令在本执行环境全部因权限失败（前端两条为 Node 以管道 stdio 派生被禁止、`spawn EPERM`；后端为 `tmp_path` 建目录 `PermissionError [WinError 5]`，43 errors / 14 passed），未取得任何门禁绿灯；同时因诊断额外产生 `.dsh-pytest-tmp\` 与 `modeprobe-d01c46\` 两个无法删除的目录需外部清理；`App.tsx:107` 清空 `basisReason` 经静态核对可覆盖已观察的跨批次残留且不引入误清，但无自动化或浏览器证据；T04 定向复核「接受」不覆盖当前 `架构设计.md`、`app/service/baselines.py`、`tests/test_baselines.py` 三份哈希；T05 不得结项。
