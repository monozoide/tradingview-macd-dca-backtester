# 📋 BUSINESS ANALYST - SPECIFICATIONS & GAP ANALYSIS
## MACD DCA Strategy for TradingView (Pine Script v5)

**Version**: 1.0  
**Date**: December 29, 2025  
**Author**: Business Analyst  
**Status**: ✅ Approved – Ready for FA traceability  

---

## 📌 EXECUTIVE SUMMARY

This document formalizes:
- ✅ Detailed business requirements (18 sections)
- ✅ Gap analysis: current state vs. future state
- ✅ Requirements traceability matrix (needs → features)
- ✅ Precise business rules and constraints
- ✅ Business processes (trading workflow + backtesting)

---

## 1️⃣ BUSINESS CONTEXT & PROBLEM STATEMENT

### Business Problem
**Statement**: Active traders on TradingView do not have an automated and transparent backtesting tool to validate the profitability of a strategy combining the MACD oscillator with progressive accumulation (DCA). Existing solutions are either complex to code, black-box (proprietary), or paid.

**Business Impact**:
- Retail traders test strategies “blindly” without precise data
- Inability to validate a strategy in 2 minutes
- Lack of confidence in investment decisions
- Unexploited market opportunity (200k+ potential traders)

### Business Use Case
**Primary Actor**: DCA Trader (progressive accumulator)  
**Objective**: Validate whether a MACD + DCA strategy was historically profitable  
**Scope**: Standalone backtester, not live trading

**Main Scenario**:
1. Trader opens a TradingView chart (BTC/USD, 1h timeframe)
2. Applies the “MACD DCA Backtest” script from the library
3. Configures: `investment_per_candle = $100`, default MACD
4. Reviews the trades table + global summary
5. Decision: “Yes, I invest” or “No, too risky”

---

## 2️⃣ DETAILED BUSINESS REQUIREMENTS (EB-1 to EB-18)

### EB-1: P&L Calculation Accuracy
**Requirement**: Calculated P&L must be accurate to the decimal and reproducible on any dataset

| Aspect | Detail |
|------|--------|
| **Criterion** | Exact P&L (€) for 100% of trades |
| **Justification** | Critical trust: trader must trust the results |
| **Measurement** | Manual vs. system comparison on 20 trades |
| **Acceptance** | ≤ €0.01 maximum deviation |
| **Business Dependency** | EB-8 (Accumulation), EB-9 (KPI Calculation) |

---

### EB-2: Full Transparency
**Requirement**: Every calculation, step, and value must be inspectable and verifiable by the user

| Aspect | Detail |
|------|--------|
| **Criterion** | 100% of trades displayed in detail |
| **Justification** | Black-box = no trust. Transparency = trust |
| **Measurement** | User can manually verify each trade |
| **Acceptance** | 18-column table, all trades visible |
| **Business Dependency** | EB-12 (Table Display) |

---

### EB-3: Ease of Use
**Requirement**: A trader without technical skills must be able to backtest in < 2 minutes

| Aspect | Detail |
|------|--------|
| **Criterion** | Onboarding < 2 minutes, 0 technical errors |
| **Justification** | Frictionless UX = rapid adoption |
| **Measurement** | Time to first backtest, number of user errors |
| **Acceptance** | ≥ 8/10 traders succeed on first attempt |
| **Business Dependency** | EB-1, EB-11 (Intuitive Inputs) |

---

### EB-4: Multi-Instrument Compatibility
**Requirement**: The backtester works on ALL TradingView instruments (crypto, stocks, forex, commodities)

| Aspect | Detail |
|------|--------|
| **Criterion** | No adaptation required, works on 100% of instruments |
| **Justification** | Heterogeneous market = versatile need |
| **Measurement** | Tests on 5+ different instruments |
| **Acceptance** | Same behavior regardless of instrument |
| **Business Dependency** | EB-7 (Universal MACD), EB-8 (Universal DCA) |

---

### EB-5: Multi-Timeframe Compatibility
**Requirement**: The backtester works on all timeframes (1m, 5m, 1h, 4h, 1d, 1w, 1M)

| Aspect | Detail |
|------|--------|
| **Criterion** | No adaptation required, works on 100% of timeframes |
| **Justification** | Traders use different timeframes |
| **Measurement** | Tests on 5+ timeframes |
| **Acceptance** | Consistent results regardless of timeframe |
| **Business Dependency** | EB-7, EB-8 |

---

### EB-6: Robustness & Zero Crashes
**Requirement**: The script must never crash, even with abnormal data

| Aspect | Detail |
|------|--------|
| **Criterion** | 99.9% uptime, handling all edge cases |
| **Justification** | A crash = loss of trust |
| **Measurement** | 0 crashes over 1000+ bars |
| **Acceptance** | No reported crashes, graceful handling |
| **Business Dependency** | EB-18 (Edge Cases) |

---

### EB-7: Reliable MACD Crossover Detection
**Requirement**: Detected MACD/Signal crossovers must be identical to the native TradingView MACD indicator

| Aspect | Detail |
|------|--------|
| **Criterion** | 100% crossover detection accuracy |
| **Justification** | Exact = reproducible, verifiable |
| **Measurement** | Comparison with native TradingView MACD |
| **Acceptance** | Identical crossovers, 0 false positives |
| **Business Dependency** | EB-4, EB-5, EB-15 (Trade Logic) |

---

### EB-8: Correct DCA Logic
**Requirement**: Exact progressive accumulation: 1 buy per bar from bullish signal up to the day before bearish signal

| Aspect | Detail |
|------|--------|
| **Criterion** | DCA: `buy_candle_count = N` (NOT N+1) |
| **Justification** | Common error = invalidates everything |
| **Measurement** | Validation on PO example (3 buys) |
| **Acceptance** | 100% of trades have correct buy_candle_count |
| **Business Dependency** | EB-15 (Open/Close Logic) |

---

### EB-9: Accurate KPI Calculation
**Requirement**: 18 KPIs calculated exactly according to the specification formulas

| Aspect | Detail |
|------|--------|
| **Criterion** | All formulas implemented exactly |
| **Justification** | Analytics foundation |
| **Measurement** | Excel validation on 20 trades |
| **Acceptance** | ≤ €0.01 deviation on P&L, ≤ 0.1% on % |
| **Business Dependency** | EB-1, EB-2 |

---

### EB-10: Correct Date Handling
**Requirement**: Trade dates displayed as YYYY-MM-DD and correspond to the correct bars

| Aspect | Detail |
|------|--------|
| **Criterion** | 100% correct dates, standard format |
| **Justification** | Traceability, auditability |
| **Measurement** | Date comparison vs. chart |
| **Acceptance** | All dates matching |

---

### EB-11: Intuitive & Configurable Inputs
**Requirement**: 4 inputs (investment, macd_fast, macd_slow, macd_signal) available in TradingView Settings

| Aspect | Detail |
|------|--------|
| **Criterion** | User can change parameters without coding |
| **Justification** | Strategy A/B testing |
| **Measurement** | Changing parameters changes results |
| **Acceptance** | Input GUI functional, values applied |

---

### EB-12: Complete Table Display
**Requirement**: 18-column table displayed on chart, scrollable, ALL trades visible

| Aspect | Detail |
|------|--------|
| **Criterion** | 100% of trades displayed (no limit) |
| **Justification** | Transparency + full analysis |
| **Measurement** | Table scrollable if > 15 trades |
| **Acceptance** | All trades visible, readable |

---

### EB-13: Relevant Global Statistics
**Requirement**: 14 summary metrics (total P&L, win rate, avg gain/loss, ratio, etc.)

| Aspect | Detail |
|------|--------|
| **Criterion** | Complete, decision-oriented summary |
| **Justification** | Fast macro view |
| **Measurement** | All 14 metrics displayed |
| **Acceptance** | Accurate, readable stats |

---

### EB-14: Acceptable Performance
**Requirement**: Script compiles/runs in < 2 seconds even on 5000+ bars

| Aspect | Detail |
|------|--------|
| **Criterion** | < 2s build time |
| **Justification** | Smooth UX = adoption |
| **Measurement** | Build time measurement |
| **Acceptance** | < 1.5s ideal, < 2s acceptable |

---

### EB-15: Correct Business Logic (Open/Close)
**Requirement**: Only one trade open at a time, no overlap, correct bullish → bearish signal sequence

| Aspect | Detail |
|------|--------|
| **Criterion** | Correct state machine logic |
| **Justification** | Prevents invalid overlapping trades |
| **Measurement** | No overlapping trades, correct order |
| **Acceptance** | `position_open` boolean = single source of truth |

---

### EB-16: Correct Numeric Formats
**Requirement**: Appropriate decimals: 2 for €, 4 for price, 2 for %, 6 for quantities

| Aspect | Detail |
|------|--------|
| **Criterion** | Exact formatting as specified |
| **Justification** | Professionalism, clarity |
| **Measurement** | Visual check of 18 columns |
| **Acceptance** | 100% correct formats |

---

### EB-17: Appropriate Visual Colors
**Requirement**: WIN = green, LOSS = red, consistent with trading conventions

| Aspect | Detail |
|------|--------|
| **Criterion** | Standard color codes |
| **Justification** | Intuitive UX |
| **Measurement** | Table visuals |
| **Acceptance** | Clear, distinct colors |

---

### EB-18: Robust Edge Case Handling
**Requirement**: No crashes/errors for: volume=0, price=0, open position at end of backtest, 0 trades, divide by 0

| Aspect | Detail |
|------|--------|
| **Criterion** | Graceful handling of all edge cases |
| **Justification** | Production robustness |
| **Measurement** | Edge case tests, 0 crashes |
| **Acceptance** | All edge cases handled |

---

## 3️⃣ GAP ANALYSIS (Current vs. Future State)

### Current State (Before)
| Aspect | Current Situation |
|------|------------------|
| **MACD+DCA Tool** | Does not exist on TradingView |
| **DCA Backtesting** | Manual (Excel) or custom code |
| **Transparency** | Black-box if available |
| **Cost** | Paid (third-party) or custom development |
| **Adoption Time** | 1–2 hours (if skilled) or impossible |
| **Errors** | Manual calculations = error-prone |
| **Access** | Experts/devs only |

### Future State (After)
| Aspect | Future Situation |
|------|------------------|
| **MACD+DCA Tool** | Free, production-ready script |
| **DCA Backtesting** | Automated, in 2 minutes |
| **Transparency** | 18 visible columns, 100% traceable |
| **Cost** | Free |
| **Adoption Time** | < 2 minutes |
| **Errors** | 0 (automated, validated calculations) |
| **Access** | All TradingView traders |

### Identified Gaps
| Gap | Cause | Solution |
|-----|-------|----------|
| **No tool** | Underserved market | Develop Pine script |
| **Manual calculations** | No automation | Automated DCA logic |
| **Black-box** | Lack of transparency | Full 18-KPI table |
| **Accessibility** | Too technical | Simple inputs, no coding |

---

## 4️⃣ REQUIREMENTS TRACEABILITY MATRIX

### Business Requirements → Functional Requirements

| EB ID | Business Requirement | FA ID | Functional Specification | Acceptance Criterion |
|------|----------------------|-------|--------------------------|----------------------|
| EB-1 | Accurate P&L | F5 | Precise KPI calculation | ≤ €0.01 deviation |
| EB-2 | Transparency | F5, F9, F10 | Table + stats | 18 columns visible |
| EB-3 | Simplicity | F1 | Intuitive inputs | < 2 min setup |
| EB-4 | Multi-instrument | F2, F7 | Universal MACD | Crypto + stocks tested |
| EB-5 | Multi-timeframe | F2, F7 | Universal MACD | 5+ TF tested |
| EB-6 | Robustness | F12 | Edge cases | 0 crashes |
| EB-7 | Reliable MACD | F2 | MACD + crossover calc | Matches TradingView MACD |
| EB-8 | Correct DCA | F3, F4 | Buy logic N (not N+1) | 100% trades = N buys |
| EB-9 | Accurate KPIs | F5 | 18 exact formulas | Excel validation |
| EB-10 | Correct dates | F7 | YYYY-MM-DD conversion | All dates valid |
| EB-11 | Custom inputs | F1 | 4 parameter inputs | TradingView GUI |
| EB-12 | Full table | F9 | 18-column display | All trades visible |
| EB-13 | Global stats | F10 | 14 summary metrics | Visible summary box |
| EB-14 | Performance | F12 | < 2s script build | Timed |
| EB-15 | Trade logic | F3, F4, F8 | State, DCA, accumulation | 1 trade at a time |
| EB-16 | Formatting | F5, F9 | Precise decimals | €2, price4, qty6 |
| EB-17 | Colors | F9 | WIN=green, LOSS=red | Visually clear |
| EB-18 | Edge cases | F12 | Robust handling | volume=0, div/0 |

---

## 5️⃣ BUSINESS RULES

### BR-1: DCA Count Formula
**Rule**: `buy_candle_count = bar_index_exit - bar_index_entry - 1`  
**Example**: Entry bar 100, exit bar 104 → 104-100-1 = 3 buys (bars 101, 102, 103)  
**Reason**: Exclude exit bar (no buy), count from the day after entry

### BR-2: P&L Calculation
**Rule**: `P&L = (quantity × exit_price) - total_invested`  
**Constraint**: Exit price = Close[bearish_signal_bar], no estimation  
**Validation**: P&L > 0 = WIN, ≤ 0 = LOSS

### BR-3: Average Buy Price
**Rule**: `avg_price = SUM[(H+L)/2 for buy bars] / buy_candle_count`  
**Note**: Simple average, not weighted  
**Reason**: Fixed €100 investment each bar regardless of price

### BR-4: Position State Machine
**State CLOSED** → Detect bullish MACD crossover → **State OPEN**  
**State OPEN** → Detect bearish MACD crossover → Close trade, return to **State CLOSED**  
**Constraint**: One state at a time, no multi-position

### BR-5: Trade Display
**Visible if**: Position closed (completed trade)  
**Not visible if**: Position open (ongoing trade)  
**Reason**: No partial trades in table

### BR-6: Volume & High/Low
**Period**: Includes ALL bars from bullish signal (inclusive) to bearish signal (inclusive)  
**High Formula**: MAX(High of buy bars + sell bar)  
**Low Formula**: MIN(Low of buy bars + sell bar)

### BR-7: Zero Commission (V1)
**Sold amount** = `quantity × exit_price` WITHOUT fees  
**Reason**: Idealized V1 backtester; V2 will add commission parameter  
**Documentation**: Clear disclaimer “no fees v1”

### BR-8: Win Rate %
**Calculation**: `(count(P&L > 0) / total_trades) × 100`  
**Edge case**: If 0 trades, display “N/A” or “0%”

---

## 6️⃣ BUSINESS PROCESS (Workflow)

### Business Process: “Validate MACD+DCA Strategy”

```
1. DECISION
   Trader decides to test MACD+DCA strategy
   Need answered: Is it profitable? What is the win rate?

2. CONFIGURATION
   Opens TradingView chart (BTC 1h, EURUSD 4h, etc.)
   Applies “MACD DCA Backtest” script
   Configures inputs: investment_per_candle = $100, MACD (12-26-9)

3. BACKTEST EXECUTION
   Script iterates through history bar by bar
   ├─ Detects MACD crossovers
   ├─ Opens/closes trades
   ├─ Accumulates DCA
   ├─ Calculates KPIs
   └─ Stores in arrays

4. RESULTS VISUALIZATION
   ├─ Table: all trades in detail (18 columns)
   ├─ Summary: global stats (14 metrics)
   └─ Colors: WIN green, LOSS red

5. ANALYSIS & DECISION
   Trader reviews results
   Verifies transparency (manual verification possible)
   Decides: Invest or not? Adjust parameters?

6. ITERATION (Optional)
   Change investment_per_candle = $200
   Change MACD = (10-25-8)
   Rerun backtest (< 2 sec)
   Compare results
   Go to step 4 or End
```

### Key Events
- **Trigger**: User applies script
- **Process**: Historical scan + signal detection
- **Output**: Table + summary
- **Decision**: User choice

---

## 7️⃣ BUSINESS CONSTRAINTS

### Functional Constraints
- ✅ Only one trade open at a time
- ✅ DCA starts the day after bullish crossover
- ✅ Sell on bearish crossover day (NO buy that day)
- ✅ No commissions/slippage v1
- ✅ MACD parameters (12-26-9) fixed v1
- ✅ One instrument at a time

### Technical Constraints
- ✅ Pine Script v5 only
- ✅ TradingView sandbox (no external API)
- ✅ Build < 2 seconds
- ✅ Array size < 10k
- ✅ No multi-threading

### Business / Legal Constraints
- ✅ Clear disclaimer: “backtester only, not live trading”
- ✅ No guarantee of future profits
- ✅ Historical data for analysis, not prediction
- ✅ Open-source (transparency)

---

## ✅ BA VALIDATION

| Section | Complete | Approved |
|--------|----------|----------|
| Business context | ✅ | ✅ |
| EB-1 to EB-18 | ✅ | ✅ |
| Gap analysis | ✅ | ✅ |
| Traceability matrix | ✅ | ✅ |
| Business rules | ✅ | ✅ |
| Business process | ✅ | ✅ |
| Constraints | ✅ | ✅ |

**Status**: 🚀 **SPECIFICATIONS APPROVED**

Ready for Functional Analyst traceability.

---

**Author**: Business Analyst  
**Date**: December 29, 2025  
**Next phase**: Detailed Functional Analyst Specifications
