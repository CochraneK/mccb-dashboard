# MCCB 工作台（已迁移 → CochraneK/psy-exp）

本仓库的分析层（refresh.py 框架 + 报告模板 + 100% 合成演示站）已于 2026-10-28 并入
[CochraneK/psy-exp](https://github.com/CochraneK/psy-exp)，作为其**结果处理阶段**；本仓库改为重定向 stub，不再维护。

| 内容 | 新位置 |
|---|---|
| 演示站（GitHub Pages） | https://cochranek.github.io/psy-exp/demo/ |
| 报告中心（浏览器内渲染 + CSV 导出） | https://cochranek.github.io/psy-exp/research-reports.html |
| 分析框架（清洗版 `refresh.py` + 模板提取器 + `chart.min.js`） | `psy-exp/analysis/` |
| 队列参考 T / 官方分注入 / 34 列 refresh.py 兼容 CSV 导出 | `psy-exp/mccb-norms.js` / `psy-exp/mccb-official-scores.js` / `psy-exp/mccb-dashboard-export.js` |
| 往返验证（CSV → refresh.py + JS/scipy Kruskal-Wallis 交叉验证） | `psy-exp/tests/verify-export-csv.cjs` |

说明：

- 旧 Pages URL（https://cochranek.github.io/mccb-dashboard/）5 秒自动重定向到新演示站。
- 本仓库全部历史内容保留在 git 历史中（迁移前 HEAD：`1e8b833b8e7cb68af98c5357653a47beb425f7cd`），可回溯任何旧版本。
- 隐私边界不变：真实参与者数据从未进入本仓库；演示数据 100% 合成（seed `20261028`），队列参考 T 为研究口径、非 MCCB 官方常模。
