# GitHub Pages 报告历史展示

## Pages 设置

| 设置项 | 值 |
|--------|---|
| Source | Deploy from a branch |
| Branch | `gh-pages` |
| Folder | `/` (root) |

在仓库 **Settings → Pages** 中按上表配置即可。

## 站点地址

```
https://mushinouta.github.io/daily_stock_analysis/
```

## 目录结构

每次 `每日股票分析` workflow 运行后，会在 `gh-pages` 分支创建/更新以下文件：

```
/                          ← 根目录总索引（所有日期倒序列出）
  index.html
  YYYYMMDD/                ← 每天一个目录
    index.html             ← 当日报告目录页（含返回总目录链接）
    report.html            ← 股票分析报告（HTML）
    report.md              ← 股票分析报告（原始 Markdown）
    market_review.html     ← 大盘复盘（HTML）
    market_review.md       ← 大盘复盘（原始 Markdown）
```

示例路径：

- `/20260306/` — 2026-03-06 当日目录
- `/20260306/report.html` — 2026-03-06 股票分析报告
- `/20260306/market_review.html` — 2026-03-06 大盘复盘

## 历史保留策略

- 每天运行后新增一个以 `YYYYMMDD` 命名的目录，**历史目录不会被删除**。
- 根目录 `index.html` 在每次发布时自动重建，按日期**倒序**列出所有已存在的报告。
- 若当次 workflow 中 artifact 不存在或分析未生成报告，`deploy-pages-history` job 会跳过本次发布并在日志中输出原因，不影响整体 workflow 状态。

## 工作流说明

`deploy-pages-history` job 在 `analyze` job 完成后运行（`needs: [analyze]`，`if: always()`），执行以下步骤：

1. 下载当次 run 的 artifact（`analysis-reports-<run_number>`）到 `_artifact/`
2. 推断报告日期（`report_YYYYMMDD.md` 中最新的 8 位日期）
3. 使用 `pandoc` 将两份 Markdown 转换为独立 HTML
4. 生成当日目录页 `YYYYMMDD/index.html`
5. Checkout `gh-pages` 分支，将当日目录写入其中
6. 扫描所有 8 位数字目录，重建根目录 `index.html`（倒序）
7. Commit 并 push 到 `gh-pages`
