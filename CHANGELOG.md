# 更新日志

此处记录 TradingAgents 的所有重要变更。

格式基于 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)，
本项目遵循 [语义化版本](https://semver.org/spec/v2.0.0.html)。
升级时需要操作的变更列在各版本的"从……升级"下。

## [0.7.1] — 2026-10-27

决策仅按最终收盘价评分，并在长假期间跨期结算；模型传入的未补零日期不再读到运行日期之后提交的申报文件；修复了投资组合、价格、SEC 报表、报告、Azure 和 Claude 设置。

### 从 0.7.0 升级

- Docker 镜像不再包含 `.env.enterprise`。在 Docker 中使用 Azure OpenAI 时，将其密钥放入 `.env`。

### 修复

- 模型传入未补零的日期（如 `2024-7-5`）时按该日期读取。此前它可能被比较为晚于运行日期之后提交的申报文件，SEC EDGAR 随后会返回该文件。(#1488)
- 决策不再按仍在交易的时段 K 线评分；在市场休市超过一周（如国庆长假）前做出的决策，将在恢复交易后结算，而不是一直挂起。(#1548, #1539)
- 回测报告其最后一次结算的失败；同一秒启动的两个回测不再共用一个运行。(#1557, #1549)
- 以券商拼写列出的持仓（`BTCUSDT`、`700.HK`）会匹配到所分析的标的；文件加载时拒绝非有限的投资组合数值。(#1556, #1509)
- 价格保留有效数字：不足一美分的收盘价不再显示为 0.00，货币对保留其点值。(#1567)
- SEC EDGAR 报表每个期间一行，模型在长历史中不再把数字读成错误的季度；不足一百万的数字不再显示为 0。(#1517, #1589)
- 将报告保存到之前用过的文件夹时，会移除上一次运行的小节。(#1566)
- 结束早于开始的日期窗口会退回给模型，而不是返回无数据。(#1595)
- 非年度或季度的报表频率会退回给模型，而不是以该词返回年度数据。(#1593)
- 记忆的最近教训以及轮换保留的条目按决策日期排序，回填更早日期时不再挤掉它们。(#1603)
- 接近 0% 或 100% 的预测市场概率显示其与该值的距离，保留两位有效数字。(#1598)
- Azure OpenAI 收到 `max_tokens` 和已配置的端点。(#1584, #1600)
- Claude 的 effort 设置对带日期的模型 ID（如 `claude-sonnet-4-20250514`）正确判定。(#1585)
- CLI 用与运行相同的规则校验股票代码，因此 `XAUUSD+` 被接受，`..` 在提示处被拒绝。(#1604)
- Docker 镜像不再包含任何文件夹中的 `.env` 文件。
- 修复了因平台原因在 Windows 和 macOS 上失败的测试。(#1506, #1485, #1541)

### 贡献者

[@alanfate3](https://github.com/alanfate3), [@aranlucas](https://github.com/aranlucas), [@BichengWang](https://github.com/BichengWang), [@Bortlesboat](https://github.com/Bortlesboat), [@hoffman988](https://github.com/hoffman988), [@jhill-cmd](https://github.com/jhill-cmd), [@muhib-karim](https://github.com/muhib-karim), [@pei711](https://github.com/pei711), [@peiji1981](https://github.com/peiji1981), [@rudycelekli](https://github.com/rudycelekli), [@Shubham-Padkonde](https://github.com/Shubham-Padkonde), [@stevexyz2003](https://github.com/stevexyz2003), [@SuperMarioYL](https://github.com/SuperMarioYL), [@sytfly](https://github.com/sytfly), [@yuv009](https://github.com/yuv009).

## [0.7.0] — 2026-10-16

可在你的 ChatGPT 套餐上运行；回测从每次决策后的首个收盘价评分；启动时提示有新版本；每个 GitHub 发布版本附带 wheel。

### 新增

- **你的 ChatGPT 套餐。** `chatgpt` 提供方通过 OpenAI 面向开源应用的登录，在 ChatGPT Plus 或 Pro 套餐上运行 TradingAgents，无需 API 密钥。在 CLI 的提供方菜单中选择"Subscription"；无人值守运行时（Docker 和 Python，`llm_provider="chatgpt"`），用 `tradingagents login chatgpt` 登录一次。`tradingagents logout chatgpt` 撤销登录。CLI 列出该套餐的模型，推理强度与 OpenAI 一样生效。(#1194, #1475)
- **启动时的新内容。** 交互式 CLI 会提示新版本，并在能判断 TradingAgents 安装方式时给出升级本安装的命令，以及 Tauric Research 针对当前运行版本的通知。它每天最多在后台检查一次，运行从不等待它；无终端的运行从不检查，`TRADINGAGENTS_NO_UPDATE_CHECK=1` 可关闭。
- **发布版本下载。** 每个 GitHub 发布版本附带源码归档和 wheel，因此可直接从发布页面 `pip install` 安装某版本，无需克隆。

### 从 0.6.0 升级

- 安装 `cryptography` 42 和 `packaging` 24 或更高版本。
- 决策现在从其日期后的首个收盘价评分。记忆日志中已结算的结果保留较早的条目（决策日期的收盘价），因此用 `run_id` 恢复的回测会混用两者；请开始新运行以便同类比较。

### 修复

- 决策不再按其无法据以交易的新闻评分：日期为 *d* 的运行读取 *d* 收盘后发布的新闻，因此记忆日志和回测在 *d* 后的首个收盘价计入，而非 *d* 的收盘价。(#1518)
- 无终端的运行（如定时任务或 CI）在启动前不再联系公告服务器。
- 当主提供方没有推理设置时，其他提供方的模型层级保留其配置的推理强度、思考级别或 Claude effort。(#1440)

### 贡献者

[@LeonWTW](https://github.com/LeonWTW), [@OleksanderSS](https://github.com/OleksanderSS), [@sovorn-c](https://github.com/sovorn-c).

## [0.6.0] — 2026-10-03

HTML 报告；每个模型层级可配不同提供方；分析师工作期间结算所有股票代码的过往决策；Yahoo 新闻源宕机时读取公司新闻。

### 新增

- **HTML 报告。** 保存的报告还包含 `complete_report.html`，即单页报告，各小节列在正文旁，在浏览器、手机和打印时都易读。在提示处回答保存问题时也会询问该页面，并可在浏览器中打开；`--no-html` 或 `save_reports(..., html=False)` 跳过它。该页面自包含：不加载任何内容，也不运行任何内容。(#1419)
- **每个模型层级一个提供方。** 快速和深度模型可运行在不同提供方上，例如管理者用 Claude 而分析师用 OpenAI：设置 `deep_think_provider` 或 `quick_think_provider`（`TRADINGAGENTS_DEEP_THINK_PROVIDER`、`TRADINGAGENTS_QUICK_THINK_PROVIDER`）及该层级的模型。报告头会标明每个层级的提供方。(#1440)
- **过往决策与分析师并行结算。** 运行的 analyst 工作期间，所有已过持有期的已记录决策都会结算，适用于任何股票代码，而不仅是正在分析的那个。若结算失败，运行继续，报告头会说明。(#1428, #1445)
- **Reddit 上的加密货币。** 对加密货币股票代码，情绪分析师读取 r/CryptoCurrency 和 r/CryptoMarkets，以及对应币种的 r/Bitcoin、r/ethereum 或 r/solana，而非股票子版块。(#1461)
- `settle_all_pending()` 不运行分析即结算所有到期决策，供定时任务使用；它和 `settle_pending()` 返回已结算的决策以及无法结算的决策。(#1445)
- 报告头标明本次运行的评级。
- 按名称提供欧元区宏观序列：`ecb_deposit_rate`、`ecb_main_refi_rate`、`euro_hicp`、`euro_core_hicp`、`euro_real_gdp`、`germany_10y`、`france_10y` 和 `eur_usd`。(#1465)

### 模型

- Gemini 调用在 600 秒后超时，与 OpenAI 和 Anthropic 调用一致，卡住的调用不再挂起运行。(#1417)

### 从 0.5.2 升级

- 安装 markdown-it-py 4.0 或更高版本。
- 保存的报告还会写入 `complete_report.html`；传 `--no-html` 或给 `save_reports()` 传 `html=False` 可只保留 Markdown。
- 运行不再在启动前结算其股票代码的决策；Memory Log 步骤在运行期间结算每个股票代码的决策。
- `settle_pending()` 返回带 `settled` 和 `failed` 的 `Settlement`（原为 `None`）。
- 0.5.2 的检查点不会恢复。

### 修复

- Yahoo 新闻源无返回时，公司新闻从 Yahoo 搜索读取。当 Yahoo 没有该标的的文章时，其新闻报告为不可用而非不存在，并尝试配置的下一个新闻供应商。(#1467, #1469)
- 引用他人的评级（如"Consensus rating: Buy"）不再替换投资组合经理自己的评级。(#1466)
- 含空字段的 Yahoo 新闻文章不再丢失该股票代码的新闻。(#1458)
- 模型写入含杂散字符的开始日期会退回给模型更正，而不是让新闻或价格请求失败。(#1476)
- Alpha Vantage 拒绝的请求报告为不可用，而非作为数据返回；API 密钥不会出现在任何 Alpha Vantage 错误中。(#1442)
- Yahoo 基本面按各自货币显示每个金额：伦敦价格以 GBp，市值和每股数字以 GBP，ADR 的收入数字以其报告货币（如台积电用 TWD）。(#1456)
- Jev 筛查使用 `TYPESAFE_BASE_URL` 指定的端点，因此可通过 OpenRouter 运行。(#1416)
- 共享记忆日志的运行对每个决策只结算一次，中断的结算保留已完成的结果。
- 报告头将情绪分析师命名为 `sentiment`，与 CLI 一致。
- `requirements.txt` 说明依赖声明在 `pyproject.toml` 中。

### 贡献者

[@amitbhansali](https://github.com/amitbhansali), [@AyushJain-601](https://github.com/AyushJain-601), [@BichengWang](https://github.com/BichengWang), [@byte7m](https://github.com/byte7m), [@Dangerboi](https://github.com/Dangerboi), [@davidalmeida90](https://github.com/davidalmeida90), [@Jackzigen](https://github.com/Jackzigen), [@jhill-cmd](https://github.com/jhill-cmd), [@Recnelis0](https://github.com/Recnelis0), [@seaaxe](https://github.com/seaaxe), [@vaegev](https://github.com/vaegev).

## [0.5.2] — 2026-09-29

并行分析师；无人值守 CLI 运行；记录生成来源的报告；过往运行只提供其日期已知的信息。

### 新增

- **并行分析师。** 选定的分析师同时运行，因此分析师阶段耗时约等于最慢的一个；CLI 在每个分析师完成时显示其进度和时间，所有报告到齐后开始辩论。(#1255, #433)
- **无人值守 CLI 运行。** 标志回答每次运行的问题，`TRADINGAGENTS_*` 变量回答其余问题，因此定时任务或脚本无需任何询问：`tradingagents --ticker NVDA --date 2026-09-26 --analysts market,news --save --no-show`。无终端时，缺失的答案会在启动前停止运行并指明需设置什么。(#1127, #1133)
- **报告记录其生成来源。** 每份报告开头列出分析日期、版本、提供方、模型、分析师、轮次和数据供应商；状态日志将其保存为 `run_settings`。(#752, #1197)
- **Docker 数据文件夹。** `TRADINGAGENTS_DATA_DIR` 将结果、报告和记忆日志保存在宿主机文件夹中。(#865)
- **分析师工具调用上限。** 经过 `max_tool_rounds` 轮（默认 20，`TRADINGAGENTS_MAX_TOOL_ROUNDS`）后，分析师根据已有内容撰写报告。(#1420)
- `tradingagents backtest` 在每个单元格开始时显示，并显示用于继续的 `--run-id`；`--analysts` 接受 `sentiment`。
- 台湾、韩国、新加坡和欧洲主要交易所的基准。(#1392)
- 支持 Python 3.14；Docker 镜像运行 Python 3.13。

### 模型

- `deepseek-flash` 被视为思考模型，因此结构化输出可用。(#1388)

### 从 0.5.1 升级

- 需要 Python 3.11 或更高版本；安装 pandas 3。
- 最低版本：yfinance 1.7.0、langchain-core 1.6.5、langchain-openai 1.6.6、langchain-anthropic 1.7.4、langchain-google-genai 4.4.0、langgraph 1.2.12、langgraph-checkpoint-sqlite 3.1.1、langchain-aws 1.7.9（`bedrock` extra）。
- 报表默认使用 SEC EDGAR，然后 Yahoo（`fundamental_data = "sec_edgar,yfinance"`）。
- 回溯日期的运行不获取 Yahoo 或 Alpha Vantage 报表或内幕交易（它们不带申报日期），只获取公司当前名称并加以标注。
- 记忆日志为 `tradingagents.memory` 包（原为 `tradingagents.decision_log`）；结算和反思从 `tradingagents.graph` 移入其中，`Reflector` 不再从 `tradingagents.graph` 导出。
- 移除 `process_signal`；评级为 `propagate()` 的第二个值或 `final_state["final_rating"]`。
- 评级仅从其 `Rating:` 标签读取；无标签的自由文本决策为 `REVIEW`。
- 数据层函数接受 `as_of_date=`（原为 `curr_date=`）；`build_instrument_context` 和 `resolve_instrument_context` 接受 `trade_date=`。
- `VendorRateLimitError` 改为 `VendorUnavailableError`。
- 数据工具从运行状态获取标的；工具调用不再携带 `symbol` 或 `ticker`。
- 状态日志：`trader_investment_plan`（原为 `trader_investment_decision`），移除 `judge_decision`，新增 `final_rating` 和 `run_settings`。
- 0.5.1 的检查点，或在不同设置下保存的检查点，不会恢复。

### 修复

- 数据工具提供运行的标的，即使模型传入其他符号。
- 失败的供应商请求报告为不可用，而非缺失或退市的符号；一个供应商的"无数据"不再代表另一个失败的链。(#1386)
- 评级为投资组合经理自己的评级，而非其论点引用的评级 (#1383)；其行在其他输出语言中保留英文标签 (#1435)。
- 重复给出的回测单元格只运行一次 (#1415)，结算在相同日期上比较决策与其基准。
- 无价格的符号快照不再终止运行。
- 并发运行不再丢失记忆日志条目。
- 仅按年初至今申报的 SEC EDGAR 季度数字会被提供，并标注其跨度。
- Yahoo 股息率和负债权益比以百分比显示。(#1414)
- FRED 变更行标明其跨越的日期。(#1397)
- `stream_run` 向工具提供图自身的配置。
- 测试套件不联网、不写入用户文件，且以任意顺序通过；集成测试用 `-m integration` 运行。(#1395)

### 贡献者

[@1Wizzy](https://github.com/1Wizzy), [@BichengWang](https://github.com/BichengWang), [@Chaoqi31](https://github.com/Chaoqi31), [@davidalmeida90](https://github.com/davidalmeida90), [@djconnexion77](https://github.com/djconnexion77), [@fuzing](https://github.com/fuzing), [@hailampy123](https://github.com/hailampy123), [@hesam-shams](https://github.com/hesam-shams), [@HEYALT](https://github.com/HEYALT), [@Jackzigen](https://github.com/Jackzigen), [@kagura-agent](https://github.com/kagura-agent), [@loulanyue](https://github.com/loulanyue), [@mannubaveja007](https://github.com/mannubaveja007), [@olivergpt](https://github.com/olivergpt), [@rita112025-cpu](https://github.com/rita112025-cpu), [@shuyan-code](https://github