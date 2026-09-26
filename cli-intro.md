# 命令行（CLI）用法：把数据、选股与回测接进你的脚本

`bull-pie-cli` 是**只读**命令行入口：读同一份数据目录与凭据，调同一套计算逻辑，
所以结果与图形界面一致——但它能写进脚本、接进定时任务、导给 Excel 或 Python。

> 下文示例里的 `bull-pie-cli` 指二进制本身：Windows 上是 `bull-pie-cli.exe`，
> macOS / Linux 上是解压出来的 `bull-pie-cli`（当前目录下写 `./bull-pie-cli`）。

> 只是想让 AI 帮你取数？那用 MCP 更省事：见 [mcp-intro.md](mcp-intro.md)。

---

## 一、什么场景用 CLI

| 场景 | 一条命令大概长这样 |
| --- | --- |
| **每天收盘后按公式选股，落到文件** | `bull-pie-cli screen --formula "C>MA(C,20) AND V>MA(V,5)*1.5;" --universe market --sort desc --top 50 --exclude-st --min-price 5 --out hits.json` |
| **批量跑回测参数**（网格搜索） | `for f in specs/*.json; do bull-pie-cli backtest run --spec "$f" --out "out/$(basename "$f")"; done` |
| **把自己算的结果导给 Python / Excel** | `bull-pie-cli bars 600519.SH --start 2026-01-01 --end 2026-09-18 --json > bars.json` |
| **找相似走势的票，接进别的流程** | `bull-pie-cli shape --reference 600519.SH --vol-days 2 --vol-ratio 1.5 --near-low 8 --json` |
| **核对图上标的形态**（哪一根、凭什么算命中） | `bull-pie-cli patterns 600519.SH --start 2026-01-01 --end 2026-09-24 --interval week --json` |
| **先看环境对不对**（排障第一步） | `bull-pie-cli status` |
| **给 AI 客户端当数据源** | `bull-pie-cli mcp`（见 [mcp-intro.md](mcp-intro.md)） |
| **收盘后拉一批价量**（自选 / 指定标的） | `bull-pie-cli quote 600519.SH,000001.SZ --out snap.json` |
| **每天存一份市场快照**（宽度 + 情绪） | `bull-pie-cli breadth --out breadth.json` / `bull-pie-cli sentiment --out sentiment.json` |
| **拿基准 / 板块日线做超额分析** | `bull-pie-cli index 000300.SH` / `bull-pie-cli index-bars 000300.SH --start 2026-01-01 --end 2026-09-18 --out hs300.json` |
| **看板块轮动**（哪类在涨、谁在里面） | `bull-pie-cli sectors --tag industry` / `bull-pie-cli sector 885431.TI --limit 30` |
| **看期货 / 期权** | `bull-pie-cli futures CUZL.SHF --days 60` / `varieties` / `varieties --options` / `options IO2609-C-4500.CFE` |
| **盯竞价**（09:15–09:30 各跑一次） | `bull-pie-cli auction --codes 600519.SH,000001.SZ` |

**不适合**：交互式看盘（那用界面）；任何写操作（CLI 一律只读）。

---

## 二、快速开始

**1. 拿到它**（各平台都有现成的，挑一个来源即可）：

| 平台 | 从哪拿 | 备注 |
| --- | --- | --- |
| Windows | 免安装版解压后，`bull-pie-cli.exe` 与 `bull-pie.exe` 在同一目录；安装版在安装目录下 | 也可以单独下载 `bull-pie-cli-<版本>-windows-x64.zip` |
| macOS | 单独下载 `bull-pie-cli-<版本>-macos-universal.zip`（一份同时支持 Intel 与 Apple 芯片） | 已经装了图形界面的话，`bull-pie.app/Contents/MacOS/bull-pie-cli` 就是同一个二进制 |

**2. 放到顺手的地方**：Windows 把它所在目录加进 PATH；macOS 拷进 PATH
（从浏览器下载的二进制带隔离属性，先解掉，否则可能报「无法验证开发者 / 已损坏」）：

```bash
xattr -d com.apple.quarantine ./bull-pie-cli     # 只有下载来的才需要
sudo cp bull-pie-cli /usr/local/bin/
```

**3. 先跑一次状态检查**（**不联网**，只看本地）：

```bash
bull-pie-cli status
```

```
版本            0.3.0
应用数据目录    C:\Users\你\AppData\Roaming\com.bull-pie.app
本地库          …\com.bull-pie.app\market.db
数据版本        bars:2026-09-18|23021|2026-09-18T13:41:25Z
凭据            已配置（来源：Windows 凭据管理器）
日线缓存        23021 行 · 最新 2026-09-18
代码表          5574 条
全市场日线数据  已导入 · 9290373 行 / 5564 只（2016-09-13 ~ 2026-09-17） · 复权因子 57302 行
最近交易日      2026-09-18
```

`--json` 会给出同样内容的机器可读版本（脚本里用这个）。

> macOS 上「应用数据目录」是 `~/.local/share/com.bull-pie.app`、凭据来源显示「macOS 钥匙串」；
> Windows 是 `%APPDATA%\com.bull-pie.app` + 「Windows 凭据管理器」。两边都不会另建一份库。

**4. 需要联网的两个前提**，都在图形界面里配一次即可：**数据接口凭据**（设置 → API 凭据）、
**全市场日线数据**（设置 → 全市场日线数据，想用 `--universe market` 选股或跑全市场回测时需要）。
只装命令行版的话这两样都建不起来——这正是它是「只读入口」的含义。

---

## 三、命令详解

全局参数对所有命令有效：

| 参数 | 作用 |
| --- | --- |
| `--json` | stdout **只有纯 JSON**，日志与提示全走 stderr，所以可以 `... --json \| jq .` |
| `--out <文件>` | 把结果直接写成 **UTF-8 文件**（推荐脚本用：绕开 Windows 控制台编码，中文不乱码）。支持它的命令：`screen`、`backtest run`、`quote`、`index`、`index-bars`、`sectors`、`sector`、`breadth`、`sentiment`、`auction`、`futures`、`options`、`varieties` |
| `--app-dir <目录>` | 覆盖应用数据目录（默认 `%APPDATA%\com.bull-pie.app`；也可设环境变量 `BULL_PIE_DATA_DIR`） |

### `status` — 本地数据与凭据状态（不联网）

```bash
bull-pie-cli status [--json]
```

排障第一条命令：看库在哪、有多少数据、最近交易日、凭据配没配、全市场日线数据导没导入。

### `search` — 检索标的

```bash
bull-pie-cli search 茅台 [--limit 20] [--json]
```

本地代码表优先，未命中才回源；返回带交易所后缀的 `thscode`（`600519.SH`）——后面所有命令都用这个代码。

### `bars` — 取 K 线

```bash
bull-pie-cli bars 600519.SH --start 2026-01-01 --end 2026-09-18 \
  [--adjust forward|backward|none] [--interval day|week|month] [--json] [--out bars.json]
```

- `--adjust` 默认 `forward`（前复权，展示口径）；`backward` 是后复权（回测口径）；`none` 是不复权。
- **复权价由本机按公司行动事件推导**，不采信上游预计算的复权价（上游那份有已知问题）。
- 本地缓存优先，缺失才回源；窗口很大时会自动按 10 年切片取数。

### `screen` — 公式选股 / 按公式值排名

```bash
# 条件选股：命中即入选（扫本地日线，不回源）
bull-pie-cli screen --formula "C>MA(C,20) AND V>MA(V,5)*1.5;" \
  --universe watchlist|cached|list|market [--codes 600519.SH,000001.SZ] \
  [--within 5] [--no-edge] [--out hits.json]

# 排名模式：把公式当**数值指标**，取前 N（加了 --sort 就不再判真假）
bull-pie-cli screen --formula "(HHV(H,172)-LLV(L,172))/LLV(L,172)*100;" \
  --universe market --sort desc --top 50 --exclude-st --min-price 5 --out top50.json
```

| 参数 | 说明 |
| --- | --- |
| `--formula` | 通达信风格公式源码（最后一条输出线非零即命中） |
| `--universe` | `watchlist` 自选 · `cached` 本地已缓存 · `list` 指定代码 · `market` 全市场日线数据 |
| `--codes a,b` | `--universe list` 时的代码列表 |
| `--within N` | 最近 N 个交易日内命中就算；不传表示不限窗口（只按最新一根判断）。排序模式忽略 |
| `--no-edge` | 不看「由假变真」的上升沿，只要当前满足就算 |
| `--sort desc\|asc` | 切成排名模式（公式当数值指标），配合 `--top` |
| `--top N` | 排名模式取前 N（默认 50，上限 500） |
| `--exclude-st` | 排除 ST / *ST / 退市整理（低价 ST 股容易霸榜） |
| `--min-price N` | 只要最新价 ≥ N 元的标的 |

> 「今年以来」这类窗口：`--universe market` 时公式里的交易日数量要自己给。用
> `status --json` 里的交易日历或界面里的提示（例如今年以来 172 个交易日 → `HHV(H,172)`），
> 别估。

### `shape` — 形态相似度搜索

```bash
bull-pie-cli shape --reference 600519.SH [--window 20] [--start 2026-06-01 --end 2026-08-04] \
  [--vol-days 2] [--vol-ratio 1.5] \
  [--near-low 8] [--limit 10] [--include-st] [--json]
```

按参照标的的形状（归一化价格 + 量能距离）在**本地全市场日线数据**上找相似股票。
默认取最近 `--window` 根；`--start/--end` 成对给出时按指定历史区间比较（最多 120 个交易日）。
可要求近期连续放量、限制在区间低点附近。默认排除 ST。

### `patterns` — 形态识别

```bash
bull-pie-cli patterns <代码> --start <日期> --end <日期> \
  [--interval day|week|month] [--adjust forward|backward|none] [--strong-only] [--limit N] [--json]
```

按软件内置的形态知识库在本地识别：蜡烛图 16 类 + 经典图表 7 类。结果与个股页 K 线上的标注是**同一份**，
可以拿来核对「图上标的那几个到底是怎么算出来的」。`--strong-only` 只列强度为「强」的命中；
代码要带后缀（`600519.SH`）。

> 日线以外也支持周线 / 月线（阈值是相对比例，与周期无关）；分钟级不做形态识别。

### `backtest` — 回测

```bash
bull-pie-cli backtest run --spec spec.json [--out result.json]
bull-pie-cli backtest list [--limit 20] [--json]
```

`--spec` 是界面「回测」页同一套参数对象（`universe/range/signal/execution/costs/capital/benchmark/adjust`），
从界面导出或手写都行。引擎：事件驱动日线、T+1、次日开盘成交、涨跌停与停牌处理、100 股整手、
四项费用与滑点。`--out` 会写出完整结果（含净值曲线与成交明细）；不加则只打印指标摘要。

> `backtest list` 的表格里 ID 只显示前 8 位；**完整 id 用 `--json` 拿**（回测详情、复现、给 AI 复盘都需要完整 id——AI 客户端的 `backtest_get` 就是按 id 取完整结果）。

### `mcp` — 以 MCP 服务器运行

```bash
bull-pie-cli mcp [--app-dir <目录>]
```

stdio 一行一条 JSON-RPC，给 AI 客户端用。配置与工具清单见 [mcp-intro.md](mcp-intro.md)。

### `quote` — 批量行情快照

```bash
bull-pie-cli quote 600519.SH,000001.SZ [--out snap.json] [--json]
```

一次拿到最新价 / 涨跌幅 / 涨跌额 / 今开 / 最高 / 最低 / 昨收 / 成交量 / 成交额。
代码可以逗号分隔，也可以空格分隔；单次最多 300 个（要扫全市场请用 `screen`）。

### `index` / `index-bars` — 指数与板块

```bash
bull-pie-cli index 000300.SH,399001.SZ,885431.TI [--out index.json] [--json]
bull-pie-cli index-bars <代码> --start <日期> --end <日期> [--limit 250] [--out bars.json]
```

- 指数（`000300.SH`）与同花顺板块（`885431.TI`）走的是**另一条上游端点**，所以要和个股的 `bars` 分开：
  **无复权概念**、上游只提供**最近约 5 年**、单次约 1000 根——区间越界会静默返回空（脚本里要判断 `bars` 是否为空）。
- 名称取自本地代码表，指数 / 板块的简称可能只有代码；要看板块中文名用下面的 `sector`。
- 常用指数代码：`000001.SH` 上证指数、`399001.SZ` 深证成指、`000300.SH` 沪深300、`000905.SH` 中证500、
  `399006.SZ` 创业板指、`000688.SH` 科创50。

### `sectors` / `sector` — 板块榜与成分股

```bash
bull-pie-cli sectors [--tag cn_concept|industry|region|tszs] [--out sectors.json]
bull-pie-cli sector <板块代码> [--limit 30] [--out sector.json]
```

- `sectors`：某类板块的涨幅榜 / 跌幅榜 / 成交额榜（默认概念板块）。
- `sector 885431.TI`：该板块的统计条（涨跌家数、平均涨跌幅、成交额、最强最弱）与成分股明细
  （代码 / 名称 / 最新价 / 涨跌幅 / 振幅 / 成交额 / 均价）。宽概念板块上千只，默认只打前 30 行，用 `--limit` 调。
- 上游只返回**当前**成分股，没有历史调入调出；同花顺没有开放资金流接口，板块涨跌是资金动向的代理，不是真实净流入。

### `breadth` / `sentiment` — 市场概览

```bash
bull-pie-cli breadth   [--out breadth.json] [--json]
bull-pie-cli sentiment [--out sentiment.json] [--json]
```

- `breadth`：沪深京成交额、涨跌平家数、平均与中位数涨跌幅、涨跌幅分布、分板块（主板 / 创业板 / 科创板 / 北交所）概览。
- `sentiment`：涨停 / 跌停 / 炸板家数、最高连板、涨停梯队（连板数、封单额、首封时间、换手）、热股榜。
- 都是**当日快照**：非交易日 / 盘前会给最近一个交易日的数据，返回里的 `tradeDate` 写明是哪天。上游没有开放资金流接口，板块涨跌是资金动向在本项目里的代理。

### `auction` — 集合竞价

```bash
bull-pie-cli auction [--codes 600519.SH,000001.SZ] [--out auction.json]
```

不传 `--codes` 就只回「短线风向标基准」（上游按当日挑的样本，带「高开 / 放量」这类标签）；
传了就再加一张竞价快照（竞价价、竞价涨跌幅、量比、相对昨日量、换手、未匹配量、竞价额）。
**09:15–09:30 才是竞价进行时**，其余时间拿到的是当日终态——返回里的 `auctionPhase` / `dataStatus` 照实转述上游取值。

### `futures` / `options` — 期货与期权

```bash
bull-pie-cli futures <合约或主连> [--days 250] [--out futures.json]
bull-pie-cli options <期权合约>   [--days 250] [--out options.json]
bull-pie-cli varieties [--options] [--out varieties.json]
```

- `futures CUZL.SHF`：主连（上游不给主连的到期日，输出里会写明）；`futures IF2609.CFE`：具体合约（带上市 / 到期 / 最后交易日）。
- `options IO2609-C-4500.CFE`：方向（认购 / 认沽）、行权价、行权方式、标的、到期。
  **上游没有期权合约列表接口**，只能按完整代码查；各交易所写法不同：中金所 `IO2609-C-4500.CFE`、上期所 `CU2610C100000.SHF`、大商所 `M2611-C-3000.DCE`、郑商所 `SR611C6000.CZC`（月份 3 位）。
- `varieties`：期货品种目录（91 个，带夜盘 / 保证金 / 主力合约）；`--options` 换成期权品种（88 个）。只打前 40 个，完整名单用 `--json`。
- `--days N` 是**说几根就几根**（默认 250，上限 1200）。

### `help` / `version`

```bash
bull-pie-cli help        # 完整用法
bull-pie-cli version     # 版本号（脚本里判断环境用）
```

---

## 四、写脚本时怎么用

### 输出与编码（最常踩的坑）

- 默认是**给人读**的表格 + 摘要；`--json` 才是纯 JSON（stdout 只有数据，日志走 stderr）。
- **中文乱码**：管道 / 重定向的编码由调用方决定，中文 Windows 的 PowerShell 5.1 按 GBK 解码会乱码
  （macOS / Linux 的终端默认就是 UTF-8，不会有这个问题）。脚本里请用 `--out <文件>`
  （CLI 自己按 UTF-8 落盘），而不是 `>` 重定向。

```bash
# 推荐：结果写文件，再交给 Python / jq 处理
bull-pie-cli screen --formula "C>MA(C,20);" --universe market --top 50 --out hits.json
python -c "import json;print(len(json.load(open('hits.json',encoding='utf-8'))['hits']))"

# 也可以直接管道（stdout 是纯 JSON）
bull-pie-cli bars 600519.SH --start 2026-01-01 --end 2026-09-18 --json | jq '.bars[-1]'
```

### 退出码（脚本里据此判断）

| 退出码 | 含义 | 典型处理 |
| --- | --- | --- |
| `0` | 成功（`status` 只是「报告状态」：缺凭据、全市场日线数据没导入也算成功，把 `未配置` 打在表里） | 继续 |
| `1` | 参数 / 凭据 / 前置条件（例如全市场日线数据没导入、Key 没配） | 看 stderr 的提示，修好再跑；**不要**自动重试 |
| `2` | 上游或网络问题（可重试） | 稍后重试；上游限流有明确提示 |
| `3` | 本地读写故障 | 检查磁盘 / 权限 / 数据目录 |

### 两个常见组合

```bash
# 1) 每日收盘后选股，结果落盘归档
bull-pie-cli screen --formula "C>MA(C,20) AND V>MA(V,5)*1.5;" --universe market \
  --exclude-st --min-price 5 --out "hits-$(date +%F).json"

# 2) 串行跑一批回测参数（避免撞上游限流）
for f in specs/*.json; do
  bull-pie-cli backtest run --spec "$f" --out "out/$(basename "$f")" || exit 1
done
```

定时跑（收盘后每天一次）各平台都有现成做法：

```powershell
# Windows：计划任务 / PowerShell
pwsh -c "& 'C:\Tools\bull-pie\bull-pie-cli.exe' screen --formula 'C>MA(C,20);' --universe market --out 'D:\hits\daily.json'"
```

```bash
# macOS：crontab -e 里加一行（15:35 跑；别用 launchd 的 StartCalendarInterval 也行）
35 15 * * 1-5 /usr/local/bin/bull-pie-cli screen --formula 'C>MA(C,20);' --universe market --out "$HOME/hits/$(date +\%F).json"
```

---

## 五、数据目录与凭据

| 项 | 说明 |
| --- | --- |
| 数据目录 | 与图形界面**同一份**（CLI 不会另建库）：Windows 默认 `%APPDATA%\com.bull-pie.app`、macOS 默认 `~/.local/share/com.bull-pie.app`；优先级 `--app-dir <目录>` > 环境变量 `BULL_PIE_DATA_DIR` > 默认 |
| 凭据 | 与界面共用（系统凭据库：Windows 凭据管理器 / macOS 钥匙串）。CLI 不提供写入凭据的命令，请在界面里配一次 |
| 只读 | CLI 不提供任何写操作：改自选、存公式、改监控、触发推送、导入全市场日线数据都只能在界面里做 |
| 限流 | 上游限流按**进程**算：GUI 与 CLI 同时大批量拉数会互相抢配额，脚本里建议串行、别并发 |

---

## 六、限制与排查

| 现象 | 原因与处理 |
| --- | --- |
| `status` 显示「凭据：未配置」 | 到图形界面「设置 → API 凭据」配一次；或给它设环境变量 `HITHINK_FINANCE_API_KEY` |
| 选股返回空 / 报「范围内没有本地日线」 | 先跑 `status` 看「全市场日线数据」那一行（或到界面「设置 → 全市场日线数据」）；`--universe market` 需要全市场日线数据 |
| 中文乱码 | 用 `--out <文件>`，别用 `>` 重定向（见上） |
| 退出码 2 + 「请求过于频繁」 | 上游限流：稍后重试，或把脚本改成串行、拉长间隔 |
| 输出被截断 | `screen` 有 `--top`、`shape` 有 `--limit`、`futures`/`options` 有 `--days`；`varieties` 默认只打前 40 个（完整名单用 `--json`）；`bars` 返回全部区间（很大时用 `--out`） |
| 期货 / 期权报「标的不存在」 | 期货代码要带后缀（`CUZL.SHF` / `IF2609.CFE`）；期权必须填**完整**合约代码，各交易所写法不同（见上） |
| 市场类命令给的是昨天的数据 | `breadth` / `sentiment` / `auction` 都是当日快照，非交易日或盘前给最近一个交易日，返回里的 `tradeDate` / `benchmarkDate` 写明是哪天 |
| 结果与界面不一致 | 先核对**口径**：复权（`--adjust`）、区间、股票池（`--universe`）、是否排除 ST；两者用的是同一套引擎，差异 99% 来自参数 |

**边界**：只读、只在本机使用；行情与财务数据版权归数据服务方，请遵守其授权条款，**不要二次分发**；
所有结果都是历史数据的计算结果，不构成投资建议。
