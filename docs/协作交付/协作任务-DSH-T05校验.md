# T05 后续执行任务包

- 批次：`20260927-T05-DSH-001`；执行方：DSH `deepseek-flash`；协调方：Codex；最终独立复核方：Luna。
- 项目：`C:\Users\YUAN\Desktop\个人项目\AI应用评测脚手架`。只处理本项目，不读写金融项目或用户其他目录。
- 目标：核实 T05 页面当前修订的构建与测试结果，定位审核后变化的版本边界，给出有行号和实测输出的剩余工作清单。Codex 已在隔离自造样例上看到旧回归批次的依据说明带入新批次，已在 `web/src/App.tsx` 的 `regressionId` 切换效果中清空该状态；修订尚未完成构建与浏览器复测。
- 先依次读取 `AGENTS.md`、`README.md`、`HANDOFF.md`；再读 `需求文档.md` 第八节、`架构设计.md` 第八节、`开发计划.md` T04/T05、`审核意见-Qoder-T04定向复核.md`、`审核意见-Luna-T05初版.md`、`审核意见-Luna-T05复核.md`、`docs/T05浏览器演练记录.md`、`web/src/App.tsx`、`web/src/api/http.ts`、`app/service/baselines.py`、`tests/test_baselines.py`。
- 固定输入哈希：`web/src/App.tsx` 为 `39db9378eb29240c02736f373bb3b6e919715a2010f5005b464cf87b32091ec0`；`架构设计.md` 为 `7b16ce21e43afea27bc37b149b5ed02ba544993a5afa1099763558b387bd4d1a`；`app/service/baselines.py` 为 `54df6a80ce58749686fec2c6c22de211698d740a2ec903cc8d0b1443a538af15`；`tests/test_baselines.py` 为 `c760cb7239a228416f6f06e42b28c3008c7586ffecf144276650e69a679a0e3c`。先复算，不符即停止并报告。
- 执行：运行 `npm --prefix web test`、`npm --prefix web run build`，用本机 Python 3.13 运行后端全量测试；记录实际命令、退出码、通过数和警告。只分析代码与测试，不打开可见浏览器，不触发原生选文件窗口，不安装依赖或更改配置。若沙箱阻断命令，保留原始错误，不用绕过权限参数。
- 分析：核对 `regressionId` 切换时依据说明清空是否解决已观察到的残留；核对 T04 定向复核之后 `架构设计.md`、`app/service/baselines.py`、`tests/test_baselines.py` 的变化范围与契约影响，明确哪些旧审核结论不能覆盖当前哈希。不得把生产方的浏览器记录声称为自己实测。
- 只允许新建一个交付文件：`docs/协作交付/T05-DSH-执行报告.md`。业务源码、需求、架构、开发计划、交接、README、夹具一律不改；构建器自动更新被忽略的 `web/dist` 可接受。若发现需修代码，只在报告写出精确改法、受影响文件与测试，不直接修改。
- 验收：报告逐项写事实、文件行号或原始命令输出、哈希、`changed_files`、会话 ID、未验证边界；最后一行必须是 `批次：20260927-T05-DSH-001 ｜ 状态：已完成 ｜ 结论：<结论>`。完成报告并回读后停止，不自行派下一任务。
