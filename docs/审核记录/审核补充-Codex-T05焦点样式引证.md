# Codex 核查补充：T05 键盘焦点样式引证

- 日期：2026-09-28（北京时间）
- 关联审核：`审核意见-Qoder-T05最终核验.md` 的“未变更受审文件抽查”。

## 核查结论

Qoder 最终报告称 `web/src/style-extra.css` 当前仅 3 行且没有 `focus-visible`；生产方现场回读发现这条观察不符合当前文件：文件共 4 行，第 4 行定义 `button:focus-visible,.file-label:focus-within` 的 2 像素轮廓及 3 像素偏移。

## 可复核证据

命令：`rg -n -i "focus-visible|outline|focus" web/src/style.css web/src/style-extra.css web/src/App.tsx`

实际匹配：

```text
web/src/style-extra.css:4:button:focus-visible,.file-label:focus-within{outline:2px solid #b65b3d;outline-offset:3px}
```

`Get-Content web/src/style-extra.css` 同次回读输出四行。文件 SHA-256 `7300b40726b41242491b0b10d060d793549d88030fc7d06176aca5d962568050` 与两份 Qoder 任务钉版值一致。因此该差异是 Qoder 最终报告的行数／内容读取误判；第一轮报告关于焦点样式的引用有效，无需更改 CSS。键盘浏览器实测范围仍按 `docs/T05浏览器演练记录.md:38` 限定在回归阶段控件。
