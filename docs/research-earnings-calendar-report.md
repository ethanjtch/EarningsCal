# 美股财报日期日历订阅方案调研报告

> 项目：earnings-calendar（自托管美股财报日历 ICS 订阅）
> 调研日期：2026-08-11 ｜ 范围：现成订阅服务 + 数据源/API 两类方案
> 结论速览：**没有主流免费服务提供稳定、支持自选列表的 iCal/WebCal 财报订阅，自托管是合理路径；当前 Nasdaq 单源可用但需加备源；仓库最值得立即修复的是 ICS UID 随机导致订阅端重复事件的问题。**

---

## 一、现成财报日历订阅服务对比

| 服务 | 覆盖 | 免费/付费 | 自选列表 | iCal/WebCal 订阅 | 更新 | 点评 |
| --- | --- | --- | --- | --- | --- | --- |
| Nasdaq 官网日历 | 美股全市场（Nasdaq+NYSE 等），实测单日 240 家 | 免费 | 无（网页） | **无**，但其数据源 API 可自抓（见下） | 实时 | 网页浏览为主，无订阅形态 |
| Yahoo Finance | 全球 | 免费 | 无 | **无**；底层 API 已收紧（实测需登录态） | 实时 | 网页日历，不适合自动化 |
| TradingView | 全球（财报+经济日历） | 免费（高级功能付费） | 无 | **无官方 iCal**；社区有爬取脚本（scraper.tradingview.com），有 ToS/稳定性风险 | 实时 | 页面体验好，不可订阅 |
| MarketBeat | 美股 | 免费看日历，部分功能需 All Access 订阅 | 无 | **无 iCal**，强项是邮件提醒 | 实时 | 提醒走邮件，非日历订阅 |
| EarningsWhispers | 美股 | 免费 | 无 | **无 iCal** | 实时 | 网页日历 |
| Wall Street Horizon | 全球 11,000+ 公司、40+ 事件类型，机构级最准 | 付费（询价，机构级） | 支持（企业平台） | 面向机构交付，无面向个人的订阅链接 | 实时 | 数据质量天花板，价格门槛高 |
| Benzinga | 美股 | Pro/API 均付费 | Pro 支持 | **无公开 iCal**；API 有 earnings calendar | 实时 | 付费为主，API 需询价 |
| Investing.com / Zacks / Seeking Alpha / CNBC / stockanalysis.com | 美股为主 | 免费 | 无 | **均无 iCal 订阅** | 实时 | 全部是网页形态 |

**小结**：现成服务里「免费 + 稳定 iCal 订阅 + 支持自选列表」三者兼备的产品不存在。免费服务要么只做网页（Nasdaq/Yahoo/Investing.com），要么订阅形态缺失（MarketBeat 走邮件）；支持自定义列表且数据质量高的（Wall Street Horizon、Benzinga）都是付费机构产品。这正是 earnings-calendar 自托管方案存在的理由，**不需要被某个现成服务替代**。

---

## 二、数据源 / API 方案对比

| 数据源 | 覆盖 | 关键字段（日期/盘前盘后/确认） | 免费档限额 | 付费 | 稳定性 | 接入难度 |
| --- | --- | --- | --- | --- | --- | --- |
| **Nasdaq**（当前在用）`api.nasdaq.com/api/calendar/earnings` | 美股全市场 | ✅ 日期、`time`（pre-market/after-hours）、fiscalQuarterEnding、epsForecast、noOfEsts、marketCap、lastYearEPS | 无 key、无公开限额；逐日请求 31 次/天 | 免费 | ⚠️ 非官方公开 API，无 SLA，可能加反爬/改版；**2026-08-11 实测正常** | 低（已接入） |
| **Yahoo Finance** `query1/2.finance.yahoo.com/v1/finance/calendar/earnings` | 全球 | ✅ 日期、EPS 预估/实际 | 免费但需 cookie+crumb | 免费 | ❌ **实测 404/429，已要求登录态**，近年持续收紧 | 中 |
| **Financial Modeling Prep (FMP)** | 全球 70k+（美股全） | ✅ 日期、`time`（bmo/amc）、EPS/Revenue 预估；**from/to 范围一次返回** | 免费 250 次/天（注册 key） | Starter $22/月、Premium $59/月 | 稳定、文档好 | 低（1 次请求覆盖整个窗口） |
| **Alpha Vantage** `EARNINGS_CALENDAR` | 美股 | ⚠️ 日期、EPS 预估，**无盘前/盘后字段**；一次返回未来 3/6/12 个月 | 免费 25 次/天、5 次/分 | 免费 key 够用 | 稳定 | 低，但字段短板 |
| **Finnhub** `/calendar/earnings?from&to` | 美股（免费档） | ✅ 日期、`hour`（盘前/盘后）、EPS/Revenue 预估、quarter/year | 免费 60 次/分、**1 个月窗口**、实时更新 | All-In-One $3,500/月 | 稳定、免费档个人使用 | 低（1 次范围请求） |
| **Polygon.io** | 美股 | ❌ **无财报日历端点**（历史财务/行情为主） | 免费 5 次/分 | $29/月起 | 稳定 | 不适用 |
| **Tiingo** | 美股 | ❌ 无财报日历端点（行情+基本面报表+新闻） | 免费档有限 | $10/月起 | 稳定 | 不适用 |
| **Benzinga API** | 美股 | ✅ 日期、时间、EPS/Revenue，实时 | 无稳定免费档 | 付费（询价） | 稳定 | 中，机构向 |
| EODHD / Twelve Data / Intrinio | 美股/全球 | 有 earnings calendar（EODHD）或依赖附加包 | 免费档有限 | 付费 | — | 中 |

**小结**：适合自托管做备源/主源的有三个免费选项：**FMP（范围查询、字段全、免费 250 次/天）、Finnhub（实时、范围查询、免费 60 次/分，但免费档窗口只有 1 个月）、Alpha Vantage（字段缺盘前/盘后，25 次/天够用但信息量低）**。Yahoo 已不可靠，Polygon/Tiingo 无此端点。

---

## 三、实测验证（2026-08-11）

1. **Nasdaq API 正常**：`date=2026-08-11` 返回 240 行，字段含 `time: "time-pre-market" / "time-after-hours"`、`fiscalQuarterEnding`、`epsForecast`、`marketCap`、`lastYearEPS`。仓库全链路 `pnpm fetch`（31 天）0 失败，`pnpm gen` 生成 1,613 个事件。
2. **Yahoo 端点已收紧**：`query1` 无 cookie 返回 404，`query2` 返回 429——免费匿名调用已不可行，不建议作为备源。
3. **发现 ICS UID 随机问题（重要）**：连续两次 `pnpm gen`，两次输出的全部 `UID:` 完全不同（`src/generate/ics.js:6` 的 `buildEvent` 未指定 `uid`，`ics` 库每次运行随机生成）。**后果**：订阅端（尤其 Outlook/Google）每次刷新都把整份日历当成「全新事件」，容易产生重复事件、重复提醒、事件闪烁。**修复成本极低**：在 `buildEvent` 中加确定性 UID，例如 `` uid: `earnings-${entry.symbol}-${entry.date}@earnings.ethanfun.xyz` ``。
4. 事件格式确认：全天事件 `DTSTART;VALUE=DATE:20260810`、`METHOD:PUBLISH`、`X-WR-CALNAME`、盘前/盘后已进 SUMMARY——与主流日历软件兼容性良好。

---

## 四、推荐方案

1. **主源保留 Nasdaq**：免费、字段全（含盘前/盘后）、实测稳定，且已接入。不切换。
2. **新增备源 FMP（首选）**：免费 250 次/天，`earning_calendar?from&to` 一次请求覆盖整个 ±30 天窗口，字段含 bmo/amc，文档稳定。fetch 失败时自动 fallback，消除「Nasdaq 单源」的最大风险。
3. **Finnhub 作第二备源（可选）**：实时性好，但免费档窗口只有 1 个月（约等于仓库 31 天窗口的上限，略紧），适合作为 FMP 之外的再一层保险。
4. **不建议**：Alpha Vantage（无盘前/盘后字段）、Yahoo（登录态门槛）、Polygon/Tiingo（无财报日历端点）、付费切换（Wall Street Horizon/Benzinga 对本场景性价比低）。

---

## 五、可落地改进建议（按优先级）

### P0 · 立即执行（改动小、收益大）

1. **ICS UID 确定性化**：`src/generate/ics.js` 的 `buildEvent` 增加 `uid: earnings-<symbol>-<date>@earnings.ethanfun.xyz`。10 行内改动，消除订阅端重复事件风险。这是本次调研发现的最值得修的问题。
2. **README 同步现状**：仓库实际生成 9 个日历（all / nasdaq100 / sp500 / dow30 / customstock / selected / megacap / largecap / midcap / smallcap），README 仍只写 6 个；补充「订阅刷新频率」说明（Google 最长 24h 延迟）。

### P1 · 本周可做

3. **备源接入**：新增 `FINNHUB_API_KEY` / `FMP_API_KEY` 可选环境变量（`.env.example` 同步），`src/fetch/earnings.js` 中 Nasdaq 请求失败时按 FMP → Finnhub 顺序 fallback（FMP 一次范围请求即可写满 31 天缓存，改动集中在 fetch 层，process/gen 不用动）。
4. **数据新鲜度可见**：workflow 在 `docs/` 写入 `last-fetched.json`（或 index.html 显示「数据更新于 …」），让订阅者一眼看到数据是否新鲜。

### P2 · 后续

5. **失败告警**：当前 fetch 失败静默保留缓存（`src/fetch/earnings.js:44`），连续失败用户无感知。可在 workflow 中检测「连续 N 天 saved=0」时创建 GitHub Issue 提醒。
6. **更新频率评估**：当前每日 04:34/16:34 UTC（美东 00:34/12:34 EDT）两次，覆盖盘前确认与盘后变更，**建议保持**；若想省 Actions 用量可降为每日 1 次（~13:00 UTC），但财报季日期变更频繁，2 次更稳。
7. **窗口 ±30 天评估**：财报日期通常提前 1–2 周确认，30 天窗口合理，无需扩大；若用户需要更长提前量可做成可选配置项。

---

## 六、ICS 与主流日历软件兼容性注意点

- **当前输出已达标**：全天事件（`VALUE=DATE`）、`METHOD:PUBLISH`、`X-WR-CALNAME` 日历名、事件内带 stocks:// 与 Yahoo 链接。三大家均可订阅。
- **Google Calendar**：支持 https 订阅；**刷新最长约 24 小时**（打开应用时即时刷新），无推送。建议在页面上提示「日期变更最迟次日生效，可手动刷新」。另外 Google 订阅日历的提醒（VALARM）基本不生效。
- **Apple Calendar（iOS/macOS）**：支持 `webcal://` 与 `https://`；订阅轮询刷新（应用运行时约数分钟级），全天事件显示良好，提醒支持较好。**当前页面提供 https 链接是正确选择**（webcal 在企业网络/部分客户端兼容性差）。
- **Outlook（网页/桌面）**：支持公开 https ICS；桌面版刷新间隔可配置（15 分钟～每日）；**对 UID 最敏感，随机 UID 最容易在 Outlook 产生重复事件**——这也是 P0 修复 UID 的直接理由。
- **通用**：UID 稳定性 > 一切；SUMMARY 中不要放频繁变化的内容（当前 symbol+公司名+盘前/盘后是稳定的，OK）；全天事件不要引入时区字段，避免跨时区漂移。

---

## 七、最省事的配置方式

### 给最终用户（不碰代码）

1. 打开 https://earnings.ethanfun.xyz/，选择日历（指数成分/市值分档/自选列表）。
2. 复制订阅链接（https）。
3. 添加订阅：Apple（设置 → 日历 → 账户 → 添加订阅日历）/ Google（日历设置 → 添加日历 → 从网址添加）/ Outlook（添加日历 → 来自互联网）。10 秒完成，之后自动更新。

### 给自托管者（Fork 即用，约 10 分钟）

1. Fork 仓库 → Settings → Pages → Source 选 **GitHub Actions**。
2. Settings → Secrets and variables → Actions → Variables 添加：`SHOULD_GEN_SELECTED`、`SHOULD_GEN_ALL`、`CUSTOM_STOCKS`（自选列表，逗号分隔）。
3. 等待定时任务（每日 04:34/16:34 UTC）或手动 Run workflow。完毕。
4. 可选增强：添加 `FMP_API_KEY` 或 `FINNHUB_API_KEY` 变量启用备源自动切换（P1 落地后）。

---

*调研方式：官方文档核对 + 端点实测（Nasdaq/Yahoo）+ 仓库全链路运行验证（fetch/gen ×2 对比 UID）。*
