# A股情绪日报生成器（商业瞰 BVIEW 版）

自动抓取东方财富公开行情数据，生成单页 **HTML** 格式的 A 股市场情绪日报，覆盖涨跌停、炸板、主力资金流、题材热点、龙虎榜等板块，统一遵循 **BVIEW 品牌视觉规范**（品牌红 `#E60012`）。

> 本仓库同时是 WorkBuddy 的 `a-share-sentiment-daily` 技能源码，`SKILL.md` 为该技能的标准说明。

## ✨ 功能特性

- **五大市场指数概览**：涨跌幅、成交额一览
- **涨跌停复盘**：涨停/跌停数量、首板占比、最高连板、连板分布、题材涨停榜
- **炸板数据**：今日炸板数 + 近 15 日炸板趋势（蓝色虚线）
- **涨跌停 15 日趋势图**：原生 SVG 双折线（红=涨停 / 绿=跌停 / 蓝=炸板），通栏 1200×280 画布
- **板块资金流排行**：行业/概念主力净流入 Top（红绿双色）
- **题材热点 & 龙虎榜**：当日主线题材与机构席位动向
- **一键「下载图片」**：`html2canvas` 整页截图导出 PNG
- **BVIEW 品牌 VI**：Logo、配色、字体统一

## 📡 数据源

全部来自东方财富公开接口，无需鉴权：

| 数据 | 接口 |
|------|------|
| 涨停池 / 跌停池 | `push2ex.eastmoney.com/getTopicZTPool` / `getTopicDTPool` |
| 炸板池 | `push2ex.eastmoney.com/getTopicZBPool` |
| 板块 / 个股资金流 | `push2.eastmoney.com` 主力净流入 |
| 龙虎榜 / 题材 | 东方财富 / 同花顺公开端点 |

> ⚠️ 部分接口在 IPv6 环境下会断连，已内置 `ipv4_first` 模块在任意网络请求前强制走 IPv4。

## 📁 目录结构

```
a-share-sentiment-daily/
├── README.md
├── LICENSE
├── .gitignore
├── requirements.txt
├── SKILL.md                    # WorkBuddy 技能说明
├── scripts/
│   ├── run_daily_report.py     # 主入口：编排抓取 + 生成
│   ├── fetch_replay.py         # 涨停 / 跌停 / 炸板池抓取
│   ├── fetch_limit_history.py  # 近 15 日涨跌停 + 炸板历史
│   ├── fetch_flow_top20.py     # 板块资金流 Top20
│   ├── fetch_data_0807.py      # 早期数据抓取（requests 版，保留备查）
│   ├── build_bview_final.py    # HTML 报告渲染（卡片 / SVG 图表）
│   ├── upgrade_trend_svg_1200.py # 趋势图 1200×280 升级脚本
│   └── ipv4_first.py           # 强制 IPv4 网络修复模块（依赖）
├── references/
│   └── layout-spec.md          # 版面 / 卡片规格说明
├── assets/
│   ├── bview-logo.png          # 品牌 Logo
│   └── standard-report-*.html  # 历史标准报告样例
└── examples/
    └── A股市场情绪日报_BVIEW_*.html  # 最新真实产出样例
```

## 🚀 安装与运行

```bash
# 1. 安装依赖（仅需 requests，其余为标准库）
pip install -r requirements.txt

# 2. 运行（指定交易日 YYYY-MM-DD 与输出目录）
python scripts/run_daily_report.py 2026-10-09 ./output

# 3. 产物
# output/A股市场情绪日报_BVIEW_2026-10-09.html
```

也可单独调用渲染（需先有 `market_data_<日期>.json`）：

```bash
REPORT_OUT_DIR=./output python scripts/build_bview_final.py 2026-10-09
```

## 🎨 品牌规范

- **品牌主色**：红 `#E60012`
- **炸板高亮**：蓝 `#2E6FE0`
- **涨 / 跌**：红 `#E60012` / 绿 `#0a8a3a`（遵循 A 股习惯：红涨绿跌）
- 字体：系统无衬线 + 等宽数字，标题加粗

## ⚠️ 免责声明

本工具仅用于数据聚合与可视化展示，**所有数据来自第三方公开接口，不构成任何投资建议**。市场有风险，决策需谨慎。报告内容的准确性、完整性以数据源为准。

## 🤖 自动化（GitHub Actions）

仓库内置 `.github/workflows/daily-report.yml`，可在每个交易日收盘后**自动抓取数据并生成当日报告**，无需人工干预。

- **触发方式**
  - 定时：`cron 0 8 * * 1-5`（UTC），即北京时间每个交易日 16:00 自动运行（收盘后留 1 小时数据缓冲）；
  - 手动：在 Actions 页面点 `Run workflow`，可指定任意 `trade_date`（YYYY-MM-DD）。
- **运行逻辑**
  1. 检出仓库、安装 Python 3.11 + `requests`；
  2. 解析交易日（周末自动跳过；手动指定优先）；
  3. 调用 `scripts/run_daily_report.py` 生成报告到 `reports/`；
  4. 上传产物（Artifacts，保留 30 天）+ 自动 `commit/push` 回仓库。
- **产物**：`reports/A股市场情绪日报_BVIEW_<YYYYMMDD>.html`（与 `examples/` 同版式）。

> ⚠️ 注意：东方财富部分接口在境外 IP（GitHub 默认 `ubuntu-latest`  runner）可能受限或返回空。若定时运行抓不到数据，可改为**自托管 Runner**（位于大陆网络）或本地定时执行。生成器内置 `ipv4_first` 强制 IPv4，已缓解 IPv6 断连问题。

## 📄 开源协议

本项目以 **Apache License 2.0** 发布。版权所有 © 2026 商业瞰 BVIEW。

- 允许商用、修改、再分发，但需保留版权与许可证声明；
- 修改文件须标注变更；
- 详见根目录 `LICENSE` 全文。
<img width="1920" height="10587" alt="A股市场情绪日报_BVIEW_20261012" src="https://github.com/user-attachments/assets/52327a15-1c56-42ec-9f2d-7f8fb06739ac" />
交流方式：675880130@qq.com；微信号：laucamber
