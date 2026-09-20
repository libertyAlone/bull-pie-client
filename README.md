<p align="center">
  <img src="logo.png" alt="牛派 bull-pie" width="148" />
</p>

<h1 align="center">牛派 · A 股行情研究与回测工具</h1>

**牛派**是一款 Windows 桌面软件，适合用来看行情、管理自选和持仓、做条件选股与策略回测。

软件本身可免费使用，数据保存在你的电脑上。行情数据需要使用你自己申请的 API Key，软件不提供或代办数据账号。

> 第一次使用？可以先看带截图的 **[普通用户使用指南](USER-GUIDE.md)**。

## 下载

请到发布页面下载最新版本：

- [GitHub Releases](https://github.com/libertyAlone/bull-pie-client/releases)
- [Gitee Releases](https://gitee.com/libertyAlone/bull-pie-client/releases)（更新可能稍晚，以 GitHub 为准）

| 文件 | 适合谁 |
| --- | --- |
| `bull-pie_<版本>_x64-setup.exe` | **推荐普通用户使用**，双击安装即可 |
| `bull-pie-<版本>-portable.zip` | 免安装版，解压后运行 `bull-pie.exe` |
| `bull-pie_<版本>_x64_en-US.msi` | 适合需要 MSI 安装包的用户 |

请不要下载发布页面里的 `Source code`，那不是可直接运行的软件。

## 使用要求

- Windows 10 或 Windows 11，64 位系统。
- 一个由你自己申请的行情数据 API Key。
- 如果软件打开后白屏或闪退，请安装 [Microsoft Edge WebView2 Runtime](https://developer.microsoft.com/microsoft-edge/webview2/)。

> 软件免费不等于数据服务免费。数据权限、额度和使用范围，请以数据服务方的规则为准。

## 三步上手

1. 安装或解压软件并启动。
2. 打开 **设置 → API 凭据**，填入自己的 API Key，然后点击保存和自检。
3. 打开 **设置 → 数据**，先同步全市场代码表，之后就可以搜索股票名称或代码开始使用。

如果还没有 API Key，可前往同花顺金融数据服务（Fuyao）申请。牛派不会内置、共享或代你申请 Key。

## 可以做什么

- **看市场**：查看大盘、涨跌分布、板块表现、资金情况和市场快讯。
- **看个股**：搜索股票，查看 K 线、指标、财务摘要、公告等信息。
- **管理自选**：把常看的股票加入自选并按分组整理。
- **记录持仓**：补录买卖、分红和出入金，查看当日概览、仓位占比、收益日历、清仓分析、交易笔记及股票盈亏排行；资产分析和收益日历还可以与上证指数做同期对比。
- **条件选股**：使用内置条件或自定义公式筛选股票。
- **策略回测**：用历史数据检查策略过去的表现，并查看收益曲线和交易明细。
- **设置提醒**：满足价格、涨跌幅或公式条件时，发送到已配置的通知渠道。
- **查看更多市场**：查看全球指数、公募基金、期货和期权等信息。
- **使用 AI 助手**：可选接入本地模型或自己的模型服务，用来解释结果和整理思路。

部分功能需要先在 **设置 → 数据** 中导入全市场日线。首次导入需要一点时间，之后通常只需更新新增数据。

## 数据与隐私

- API Key、模型 Key 和通知凭据只保存在你的电脑上，请不要发给别人，也不要贴到反馈截图里。
- 行情缓存、持仓记录、回测记录和软件设置默认保存在：`%APPDATA%\com.bull-pie.app`。
- 设置页可以导出自选、偏好、公式和提醒规则。
- 如果要完整保留持仓流水、历史回测和缓存数据，请备份整个数据目录。
- 使用外部 AI 服务时，界面会提示哪些内容将被发送；不需要 AI 功能可以完全不配置。

## 常见问题

| 问题 | 处理方法 |
| --- | --- |
| 双击后白屏或闪退 | 安装 WebView2 Runtime 后重试 |
| 提示 API Key 无效 | 在设置页重新保存并自检，仍失败时到数据服务方后台重新申请 |
| 搜不到中文名称 | 到 **设置 → 数据** 同步一次全市场代码表 |
| 提示请求过于频繁 | 稍等片刻再试，并把自动刷新频率调低 |
| 自选里有 ETF，但集合竞价没有显示 | 集合竞价数据只覆盖 A 股个股，ETF、指数等标的会自动跳过，不影响其他自选股票 |
| 安装包被系统提示风险 | 安装包暂未做代码签名，请只从上面的正式发布页下载 |
| 回测结果与其他软件不同 | 不同软件的复权方式、交易价格和手续费假设可能不同，请先核对回测设置 |
| 持仓总资产显示为负数 | 如果只记录了买入、没有先记录入金，现金会变成负数。请在第一笔买入之前补录入金，再重新查看持仓 |

## 进阶使用

普通用户不需要阅读下面的内容；如果你会使用命令行或希望让其他工具读取本地数据，可以参考：

- [命令行工具说明](cli-intro.md)
- [MCP 工具说明](mcp-intro.md)

## Star History

如果这个项目帮到了你，欢迎点个 ⭐：

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=libertyAlone/bull-pie-client&type=Date&theme=dark" />
  <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=libertyAlone/bull-pie-client&type=Date" />
  <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=libertyAlone/bull-pie-client&type=Date" />
</picture>

## 问题反馈

使用中遇到问题，可以在仓库的 **Issues** 中反馈。建议附上：

- 软件版本
- 操作步骤
- 报错文字或截图

请务必遮住 API Key、模型 Key、Webhook、Secret 等敏感信息。

## 许可与免责

- 本软件可免费下载使用，源代码不公开，具体许可见 [LICENSE](LICENSE)。
- 软件不连接券商、不自动交易，也不构成投资建议。
- 行情、指标、选股和回测结果仅供研究参考，不能代表未来表现，请自行判断并承担风险。
- 行情及财务数据的权利归数据服务方所有，请遵守相应授权和使用规则。
