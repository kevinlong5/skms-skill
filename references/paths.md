# Data Path Reference

Complete database of data paths used in SKMS research workflows.

---

## CSMAR Database Paths

### Stock Returns

| Path | Content | Frequency |
|------|---------|-----------|
| `<DATA_ROOT>/CSMAR/股票收益率/` | Stock returns data | Daily/Monthly |
| `<DATA_ROOT>/CSMAR/股票收益率/个股日回报率*` | Individual stock daily returns | Daily |
| `<DATA_ROOT>/CSMAR/股票收益率/月回报率/` | Monthly returns | Monthly |
| `<DATA_ROOT>/CSMAR/股票收益率/综合市场日回报率*` | Market returns | Daily |

**Common variables**: `stkcd`, `date`, `rstk` (stock return), `rmkt` (market return), `msize` (market cap), `traden` (trading volume)

### Company Information

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/公司基本信息/上市公司基本信息年度表` | Company annual info |

**Common variables**: `Stkcd`, `year`, `ShortName`, `IndustryCode`, `PROVINCE`, `EstablishDate`, `Plate`, `Stock`

### Financial Statements

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/财务报表/最新报表/` | Latest financial statements (Acc) |

**Common variables**: 
- `A001000000` (Total Assets, TA)
- `A002000000` (Total Liabilities, DEBT)
- `B002000000` (Net Income, NI)
- `B001100000` (Sales Revenue, SALE)
- `C001000000` (Operating Cash Flow, OCF)
- `A001212000` (Fixed Assets, PPE)

### Financial Ratios

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/财务指标分析/相对价值指标/` | Valuation metrics |
| `<DATA_ROOT>/CSMAR/财务指标分析/盈利能力/` | Profitability metrics |
| `<DATA_ROOT>/CSMAR/财务指标分析/每股指标/` | Per-share metrics |

**Common variables**:
- `F100801A` (Market Value, MV)
- `F100901A` (Tobin's Q, TQ)
- `F101001A` (Book-to-Market, BM)
- `F050103B` (ROA)
- `F050503B` (ROE)
- `eps1`, `eps2` (EPS measures)

### Earnings Forecasts

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/业绩预告/业绩预警Wind/` | Earnings warnings |
| `<DATA_ROOT>/CSMAR/业绩预告/业绩快报Wind/` | Earnings quick reports |
| `<DATA_ROOT>/CSMAR/业绩预告/预约披露日期表/` | Scheduled disclosure dates |

### Shareholder Data

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/股东/` | Shareholder data |
| `<DATA_ROOT>/CSMAR/股东/股权性质/` | Equity nature |
| `<DATA_ROOT>/CSMAR/股东/股权激励/` | Equity incentives |
| `<DATA_ROOT>/CSMAR/股东/员工持股计划/` | Employee stock ownership |
| `<DATA_ROOT>/CSMAR/股东/上市公司控制人文件/` | Ultimate controllers |

### Capital Market Supervision

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/资本市场监管/问询函信息表/` | Inquiry letters |
| `<DATA_ROOT>/CSMAR/资本市场监管/监管函信息表/` | Regulatory letters |

### Share Changes

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/增减持/二级市场股份增减持情况文件/` | Secondary market share changes |
| `<DATA_ROOT>/CSMAR/增减持/董监高及相关人员持股变动情况文件/` | Executive share changes |

### ST Status

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/ST/` | Special treatment status |

### Microstructure

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/微观交易结构/买卖价差/` | Bid-ask spread |
| `<DATA_ROOT>/CSMAR/微观交易结构/买卖不平衡/` | Buy-sell imbalance |
| `<DATA_ROOT>/CSMAR/个股日交易衍生指标/` | Trading derivatives |
| `<DATA_ROOT>/CSMAR/个股日交易衍生指标/个股月换手率/` | Monthly turnover |

### Related Party Transactions

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/关联交易/关联交易情况文件/` | Related party transactions |

### R&D and Patents

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/研发/研发投入情况表/` | R&D expenditure |
| `<DATA_ROOT>/CSMAR/环境研究/绿色专利/green patent/` | Green patents |

### Institutional Investors

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/机构投资者/机构持股分类统计/` | Institutional ownership |

### Other CSMAR

| Path | Content |
|------|---------|
| `<DATA_ROOT>/CSMAR/内部控制/内控审计报告信息表/` | Internal control audit |
| `<DATA_ROOT>/CSMAR/政府补助/政府补助2003-2024/` | Government subsidies |
| `<DATA_ROOT>/CSMAR/cite visit/调研机构分类明细表/` | Site visits |
| `<DATA_ROOT>/CSMAR/增发配股/增发/` | SEO (Seasoned Equity Offering) |
| `<DATA_ROOT>/CSMAR/IPO情况/招股及上市基本情况表/` | IPO information |

---

## Project-Specific and Custom Paths

Use placeholders for project-level or manually maintained data. Replace these with local names that match the user's workspace.

| Path | Content |
|------|---------|
| `<PROJECT_ROOT>/analysis/data/` | Project-specific working data |
| `<PROJECT_ROOT>/analysis/code/` | Project Stata code |
| `<PROJECT_ROOT>/analysis/output/result/` | Current result tables and datasets |
| `<PROJECT_ROOT>/analysis/output/figures/` | Figures |
| `<PROJECT_ROOT>/analysis/output/logs/` | Stata logs and diagnostics |
| `<CUSTOM_DATA_ROOT>/ownership/` | Ownership or SOE indicators |
| `<CUSTOM_DATA_ROOT>/firm_location/` | Firm location or regional data |
| `<CUSTOM_DATA_ROOT>/analyst/` | Analyst forecasts |
| `<CUSTOM_DATA_ROOT>/governance/` | Governance or executive data |

---

## Wind Database Paths

| Path | Content |
|------|---------|
| `<DATA_ROOT>/Wind/并购重组/` | M&A transactions |
| `<DATA_ROOT>/Wind/线上销售数据/` | Online sales |

---

## Quick Reference: Most Used Paths

| Priority | Path | Typical Use |
|----------|------|-------------|
| 1 | `<DATA_ROOT>/CSMAR/股票收益率/` | Returns, event study |
| 2 | `<DATA_ROOT>/CSMAR/公司基本信息/上市公司基本信息年度表` | Company info, industry |
| 3 | `<DATA_ROOT>/CSMAR/财务报表/最新报表/` | Financial variables |
| 4 | `<DATA_ROOT>/CSMAR/财务指标分析/相对价值指标/` | MV, BM |
| 5 | `<DATA_ROOT>/CSMAR/业绩预告/预约披露日期表/` | Event dates |
| 6 | `<CUSTOM_DATA_ROOT>/ownership/` | SOE or ownership indicators |

---

## Path Variables Template

Define common paths as global macros for reusable code:

```stata
// Define paths
global CSMAR "<DATA_ROOT>/CSMAR"
global RETURNS "$CSMAR/股票收益率"
global BASIC "$CSMAR/公司基本信息/上市公司基本信息年度表"
global FINANCIAL "$CSMAR/财务报表/最新报表"
global RATIOS "$CSMAR/财务指标分析"
global CUSTOM "<CUSTOM_DATA_ROOT>"

// Usage
use "$RETURNS/stk_day", clear
use "$BASIC/STKBasic_year", clear
```
