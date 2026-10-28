# MCCB 认知测评工作台（分析代码 + 框架）

把 MCCB（MATRICS Consensus Cognitive Battery）软件导出的 `.xls / .xlsx` 数据，一键生成一组**自包含、零联网**的 HTML 工作台：1 个总览页 + 每个数据文件一份独立报告 + 1 个个人报告页。替换 / 追加 `data/*.xls` 后重跑即可刷新，浏览器只负责渲染，不需要网络。

> Self-contained HTML dashboard & report generator for MCCB cognitive battery exports. Drop in your `.xls`/`.xlsx` exports, run one script, get offline-viewable interactive dashboards with no server and no CDN.

## ⚠️ 数据说明（重要）

**本仓库不包含任何真实测评数据。**
- `demo/` 下是 **100% 程序合成的演示数据与演示站**（固定随机种子可复现），仅用于展示与验证功能形态。
- 真实数据请自行放入 `data/` 目录（该目录已被 `.gitignore` 屏蔽，不会被提交）。
- 生成的 HTML 会把数据完整内联进文件（`const DATA`），**等价于数据文件本身**——请勿把真实数据生成的 HTML 提交到公开仓库或公网发布；对外分享请使用 `demo/` 或自行脱敏。

## 快速开始

```bash
pip install -r requirements.txt
# 把 MCCB 软件导出的 .xls / .xlsx 放进 data/（也可以放在仓库根目录）
python refresh.py
# 用浏览器打开 mccb_dashboard.html
```

Windows 用户可双击 `刷新工作台.bat`（自动定位 Python → 重跑 → 打开页面）。

**先试试演示站（无需装任何东西）**：直接打开 `demo/mccb_dashboard.html`，或运行 `python demo/_make_demo.py` 重新生成演示数据与演示站。

## 入口与深链

- 总览 `mccb_dashboard.html`：KPI 卡（总场次 / 受试者 / 项目数 / 测验完成数 / 数据跨度 / 近 30 天）· 日/周/月/年趋势（双轴）· 性别 / 年龄 / 核心测验完成度 / 项目场次 / 累计曲线 · 认知剖面雷达（7 域 + 2 综合）· 项目综合分对比 · 各域综合分分布 · 测验×综合分 Pearson 相关热图 · 纵向干预前后对比 · 年龄×综合分散点 · 诊断分组 × 认知域对比（Kruskal-Wallis H/p/η² + 星号标注）· 性别 × 认知域对比 · 显著性表 · 明细表（搜索 + 筛选，可深链单人）· 项目文件清单（可深链单项目报告）。右上角「仅统计正式项目」开关剔除测试文件。
- 项目报告 `reports/report_<文件名>.html`：该文件的 KPI / 月度趋势 / 7 域雷达 / 测验完成度 / 性别 / 明细，并带「本项目 vs 健康对照」「组内性别」两套 KW 显著性（文件名含 `DIAG_MAP` 中健康对照组映射者为参照组，自动跳过 vs 对照）。
- 个人报告 `person.html?person=<文件名>|<受试者ID>`：单人 KPI / 雷达（叠加同组均值虚线）· 与同组差值 · 认知损害筛查（临床 T 分三档）· 同组对标基准表 · 纵向波段明细。

## 数据格式（按表头名读取，不靠列索引）

| 表头 | 用途 |
|---|---|
| `Respondent ID` | 受试者 ID（可带访视后缀，如 `...V08` / `...-2`，自动剥离得到稳定主 ID） |
| `Name` / `Gender` | 姓名 / 性别（Gender 兼容 Male/Female 等） |
| `Start Date`（或 `Test Date`） | 有效日期 |
| `Start Age`（或 `Test Age`） | 年龄 |
| `Assessment` | 波次（基线/随访序号，数值型） |
| `Rater` | 施测者（仅展示） |
| 9 项核心测验列（`TMT`、`BACS SC`、`HVLT-R`、`WMS-III SS`、`NAB Mazes`、`BVMT-R`、`Fluency`、`MSCEIT ME`、`CPT-IP`） | 非空即记一次完成（工作量口径） |
| `<测验> AGT`（兼容 `AGET` 后缀） | 9 项测验的年龄 T 分 |
| `<综合分> AGT`（兼容 `AGET` 后缀） | 9 个综合分（Age-scaled T 分，常模均值≈50 / SD≈10）：SoP 处理速度 / AV 注意警觉 / WM 工作记忆 / Vrbl Lrng 言语学习 / Vis Lrng 视觉学习 / RPS 推理问题 / SC 社会认知 / Overall Comp 综合 / Neurocog Comp 神经认知 |

兼容 74 列 / 75 列两种导出格式（列偏移按表头名对齐，不靠列索引）。**身份键 `pkey = 文件名|主ID`**，跨访视稳定，分组 / 计数 / 深链都用它。

## 诊断分组（DIAG_MAP）

数据本身通常**没有诊断列**，分组按**数据文件名**推断：`refresh.py` 顶部 `DIAG_MAP`（键 = 文件名 stem 小写，值 = 组名）。把你自己各队列 / 项目文件名映射到对应组即可；组名任意，展示顺序跟随 `DIAG_ORDER`。仓库自带映射是 `demo/` 演示文件的示例。
另：`TEST_FILE_HINTS` 命中的文件名视为测试 / 练习数据，可被「仅统计正式项目」开关剔除，且不参与 formal 口径的 KW 检验。

## 验证链（每次改完必跑，全绿才算完成）

```bash
python refresh.py          # 1) 重新生成，人工核对 records/files/core-tests-done 计数
python _verify_js.py       # 2) 白屏守卫：所有生成 HTML 的末段 <script> 过 node --check，期望 BAD 0
node _smoke_person.js      # 3) person.html 运行时冒烟（node:vm 沙箱 + stub DOM/Chart），期望 SMOKE_OK
```

## 关键设计（改代码前必读）

- **CORS**：浏览器读不了本地 `.xls` → 刷新必须在 Python 端重新生成 HTML，浏览器只渲染内联的 `const DATA`。
- **安全内联**：Chart.js 内联时 `</script` → `<\/script`；JSON payload `</` → `<\/`，防提前闭合标签。
- **KW 检验**：scipy 可用时输出 H/p/η²（epsilon-squared 钳 0）；scipy 缺失静默降级（图表不显示星号）。
- **白屏 ≠ 文件可达**：自包含 HTML 交付前必须跑 `_verify_js.py`。

## 隐私与合规

- 真实受试者数据（含姓名、ID、年龄、诊断分组、认知分数）属**准可识别/敏感个人信息**：建议只在本机或院内受控环境分析展示。
- 需要对外分享时：参考 `demo/` 的合成数据方案（或先做去标识化），并用「真实值集合 vs 发布产物」的集合交集法做泄漏检查（姓名 / ID / 文件名片段零出现）。
