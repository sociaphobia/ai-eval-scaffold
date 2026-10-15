# Qoder T02 初版审核处理

- 原报告：`审核意见-Qoder-T02初版.md`，批次 `20260927-T02-QODER-001`，判修正性接受。
- 本文只记录生产方处理与反证，不代替 Qoder 独立复核。

| 审核项 | 处理 | 可复核证据 |
| --- | --- | --- |
| 重-1 默认 404／422 泄漏 `detail` | 为 `RequestValidationError` 与 `StarletteHTTPException` 增加统一 `{code,message,issues}` 映射；405 代码值写回架构第八节 | `app/main.py` 对应异常处理器；`tests/test_imports.py` 的 `test_默认校验与未知路径也使用统一错误格式` 曾先红后绿 |
| 重-2 回归与摘要缺直接测试 | 增加无基线 409、完全配对、部分／无交集整批拒绝、可行动提示、摘要字段与顺序、超长和缺项断言 | `tests/test_imports.py` 对应新增测试；当前四文件合计 `24 passed, 1 warning`，2026-09-27 本机 Python 3.13 实跑 |
| 重-3 断言“原工作区仅 14 例” | 不接受该计数；初版送审时 `test_smoke.py` 含 5 个测试，加上 `test_imports.py` 7、`test_rules.py` 2、`test_repository.py` 1，合计 15，与当时命令输出一致。审查方没有独立运行命令，可能漏数静态页面烟测 | 初版工作区烟测输出在本任务工具记录；当前 `rg -c '^def test_'` 为 5＋15＋2＋2＝24，当前实跑为 24 例通过。原报告自身也标“未独立运行” |
| 可-1 纯空格字面规则 | 按规则仅拒绝空字符串，允许单个空格字面匹配；架构第八节补明确口径 | `app/service/imports.py` 规则 `text` 校验；`tests/test_imports.py` 单空格用例先红后绿 |
| 可-2 事务中途失败 | 增加数据库主键冲突时前两次写入回滚的测试 | `tests/test_repository.py` 新增回滚用例 |
| 可-3 未知项目与坏文件错误先后 | 选定未知项目 404 优先，先查项目后解析文件；架构第八节同步 | `app/service/imports.py`；`tests/test_imports.py` 未知项目加坏文件用例 |
| 可-6 JSON 重复键 | 在解析时拒绝同一对象内重复键，避免前值静默丢失；架构第八节同步 | `app/service/imports.py` `_unique_object`；`tests/test_imports.py` 重复键用例先红后绿 |

待完成：请 Qoder 定向复核上述重要项及本次架构第八节增量。T03 代码尚未写入；不能把 T02 自测通过说成整体功能完成。
