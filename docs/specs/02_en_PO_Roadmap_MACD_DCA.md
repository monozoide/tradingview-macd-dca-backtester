# 📊 PRODUCT OWNER - USER STORIES & OPERATIONAL ROADMAP
## MACD DCA Strategy for TradingView (Pine Script v5)

**Version**: 2.0 (PM Vision + Features Integration)  
**Date**: December 29, 2025  
**Author**: Product Owner  
**Status**: ✅ Approved - Ready for Sprint 0  

---

## 📌 EXECUTIVE SUMMARY

This document contains:
- ✅ 12 complete User Stories with detailed acceptance criteria
- ✅ 3 auxiliary stories (UX, performance, tests)
- ✅ Operational roadmap (4 phases, 12 weeks)
- ✅ Dependencies & development order
- ✅ Effort estimation (45–75 minutes MVP, ~15h stable v1)

---

## 🎯 PRIORITIZED PRODUCT BACKLOG

### EPIC 0: CONFIGURATION & INFRASTRUCTURE (F1, F11)

#### US-1: MACD & DCA Parameters Configuration
**Story Points**: 5 | **Effort**: 5 min | **Priority**: CRITICAL | **Dependency**: None

**As a** trader,  
**I want** to easily configure the investment amount per candle and MACD parameters,  
**So that** I can test different strategies without modifying the code.

**Acceptance Criteria**:
- ☑ Input `investment_per_candle` (float, default 5.0 €) visible in Settings
- ☑ Input `macd_fast` (int, default 12) visible in Settings
- ☑ Input `macd_slow` (int, default 26) visible in Settings  
- ☑ Input `macd_signal` (int, default 9) visible in Settings
- ☑ All inputs usable without compilation errors
- ☑ Code documentation explaining the role of each input
- ☑ Default values match MACD standards (12-26-9)

**Development Tasks**:
- Declare the 4 inputs in the `indicator()` section
- Document each input with a clear description
- Validate types (float vs int)
- Test with different values (positive, negative, zero)

**Notes**:
- Inputs must be formatted with short descriptions (<50 chars)
- Group inputs by category in the GUI (MACD vs Investment)

---

#### US-2: Timestamp to Date Conversion (YYYY-MM-DD)
**Story Points**: 8 | **Effort**: 8 min | **Priority**: HIGH | **Dependency**: US-1

**As a** user,  
**I want** to see trade dates in a readable format (YYYY-MM-DD),  
**So that** I can easily identify when each trade occurred.

**Acceptance Criteria**:
- ☑ Dates converted from `time[bar_index]` to YYYY-MM-DD format
- ☑ Exact format: "2025-12-29" (4-digit year, 2-digit month, 2-digit day)
- ☑ Valid conversion for any timeframe
- ☑ No error if bar_index is out of bounds
- ☑ Dates displayed in the table match the correct bars
- ☑ Reusable function with documentation

**Development Tasks**:
- Create helper function `f_timestamp_to_datestring(ts)`
- Use `str.format()` with correct date pattern
- Document expected output format
- Test conversion on 5+ timeframes (1m, 5m, 1h, 4h, 1d)
- Validate accurate conversion (no time offset)

**Notes**:
- Pay attention to timezone (TradingView uses UTC)
- Function must handle edge cases (missing timestamps)

---

### EPIC 1: SIGNAL DETECTION (F2, F3, F12)

#### US-3: MACD Calculation and Crossover Detection
**Story Points**: 10 | **Effort**: 10 min | **Priority**: CRITICAL | **Dependency**: US-1

**As a** strategy,  
**I want** to calculate MACD and detect crossovers with the Signal line,  
**So that** I can identify entry and exit points.

**Acceptance Criteria**:
- ☑ MACD calculation = EMA(close, macd_fast) - EMA(close, macd_slow)
- ☑ Signal calculation = EMA(MACD, macd_signal)
- ☑ Histogram calculation = MACD - Signal
- ☑ Bullish crossover detection: MACD[N-1] < Signal[N-1] AND MACD[N] > Signal[N]
- ☑ Bearish crossover detection: MACD[N-1] > Signal[N-1] AND MACD[N] < Signal[N]
- ☑ Variables stored and accessible (macd, signal, histogram, crossover_up, crossover_down)
- ☑ No calculation errors (division by 0, NaN)
- ☑ Accuracy: values identical to TradingView native MACD indicator

**Development Tasks**:
- Calculate MACD using ta.ema()
- Calculate Signal using ta.ema() on MACD
- Use ta.crossover() and ta.crossunder() for detection
- Compare results with native MACD indicator (same chart)
- Validate on 3+ instruments (BTC, EURUSD, AAPL)

**Notes**:
- Use ta.ema() for precision, not manual EMA
- Crossovers must be precise (no approximation)

---

#### US-4: Position State Management (Open/Closed)
**Story Points**: 8 | **Effort**: 8 min | **Priority**: CRITICAL | **Dependency**: US-3

**As a** strategy,  
**I want** to track the state of a position (open or closed),  
**So that** I know when to accumulate and when to stop calculations.

**Acceptance Criteria**:
- ☑ Variable `position_open` (bool) initialized to false
- ☑ `position_open` becomes true upon bullish crossover detection
- ☑ `position_open` becomes false upon bearish crossover detection
- ☑ No new trade can open while `position_open` == true
- ☑ Duplicate crossovers ignored (no two consecutive bullish signals)
- ☑ State persists across bars (`var` keyword)
- ☑ Robust logic: handles gaps, rapid crossovers, edge cases

**Development Tasks**:
- Declare `var bool position_open = false`
- Implement if/else logic for opening/closing
- Test with dataset containing rapid successive crossovers
- Validate that 2 bullish crossovers = 1 open trade, not 2

**Notes**:
- This logic is the foundation of all DCA, must be bulletproof
- Use `var` for persistence across bars

---

#### US-5: Edge Case Management (Division by 0, Crashes)
**Story Points**: 8 | **Effort**: 8 min | **Priority**: HIGH | **Dependency**: US-3 & US-4

**As a** robust strategy,  
**I want** to handle edge cases without crashing,  
**So that** the script never fails, even with abnormal data.

**Acceptance Criteria**:
- ☑ Division by 0 avoided everywhere (avg_price, avg_volume, ratios)
- ☑ Gain/Loss ratio handled if no LOSS trades (default: 0 or "N/A")
- ☑ Volume = 0 accepted (do not divide by volume if 0)
- ☑ Open position at end of backtest = no automatic close
- ☑ Duplicate crossovers ignored (no two bullish signals in a row)
- ☑ Empty array handled (table shows 0 rows, no error)
- ☑ Price = 0 or negative = ignored or filtered
- ☑ No crash after 1000+ bars (memory/recursion)

**Development Tasks**:
- Add checks `if denominator != 0` before each division
- Test with volume = 0
- Test with open position at end of dataset
- Test with 0 trades
- Test with abnormal prices (negative, 0, very large)

---

### EPIC 2: ACCUMULATION & CALCULATIONS (F4, F5, F6)

#### US-6: Accumulation of OHLCV Data During Position
**Story Points**: 12 | **Effort**: 12 min | **Priority**: CRITICAL | **Dependency**: US-4

**As an** open position,  
**I want** to accumulate OHLCV data for each candle,  
**So that** I can calculate precise statistics for the period.

**Acceptance Criteria**:
- ☑ Count number of buy candles (buy_candle_count)
- ☑ Accumulate sum of (High + Low) / 2 for each candle
- ☑ Track maximum High of the period
- ☑ Track minimum Low of the period
- ☑ Accumulate total volume of each candle
- ☑ Reset ALL accumulators when position closes
- ☑ Accumulated data correct over 100+ candles
- ☑ No memory leak (variables reset properly)

**Development Tasks**:
- Declare temp variables: buy_candle_count, sum_mid_prices, high_period_temp, low_period_temp, volume_total_temp
- Increment/accumulate during position_open
- Calculate mid_price = (high + low) / 2 at each candle
- Use math.max() and math.min() to track H/L
- Test on dataset > 100 candles

---

#### US-7: KPI Calculation for Each Closed Trade
**Story Points**: 15 | **Effort**: 15 min | **Priority**: CRITICAL | **Dependency**: US-6

**As a** closed trade,  
**I want** to calculate all KPIs (P&L, average price, volume, etc.),  
**So that** I can store precise statistics and display them.

**Acceptance Criteria**:
- ☑ Total invested = investment_per_candle × buy_candle_count (NOT +1)
- ☑ Average buy price = sum_mid_prices / buy_candle_count
- ☑ Quantity = total_invested / avg_buy_price
- ☑ Sell price = close[bar where bearish crossover]
- ☑ Amount sold = quantity × sell_price (full number)
- ☑ P&L (€) = amount_sold - total_invested
- ☑ P&L (%) = (P&L / total_invested) × 100
- ☑ Period High/Low = max/min of entire period
- ☑ % movement = ((High - Low) / Low) × 100
- ☑ Average volume = volume_total / buy_candle_count
- ☑ Gain per candle = P&L / buy_candle_count
- ☑ Win/Loss = "WIN" if P&L > 0, otherwise "LOSS"
- ☑ Correct decimals: 2 for €, 4 for price, 2 for %
- ☑ Valid values for 50+ different trades

**Development Tasks**:
- Create Trade structure/type with all fields
- Implement each KPI formula exactly as specified
- Validate with PO example (identical results)
- Test on 5+ instruments with different DCA periods
- Round decimals to correct precision

**Notes**:
- This is the core calculation, must be 100% exact
- Validate each formula against the PO

---

#### US-8: Trade Storage in an Array
**Story Points**: 5 | **Effort**: 5 min | **Priority**: CRITICAL | **Dependency**: US-7

**As a** strategy,  
**I want** to store each completed trade in an array,  
**So that** I can display full history and calculate global stats.

**Acceptance Criteria**:
- ☑ Trade type defined with all fields (minimum 18 fields)
- ☑ Array `trades` created with array.new<Trade>()
- ☑ Each calculated trade added using array.push()
- ☑ No size limit (all trades preserved)
- ☑ Array persists across bars (`var` keyword)
- ☑ Reliable access to trades (array.get(), array.size())
- ☑ No trade duplication

**Development Tasks**:
- Define `type Trade` with 18 fields
- Create `var array<Trade> trades = array.new<Trade>()`
- Implement `array.push(trades, new_trade)` on close
- Test with 100+ trades
- Validate no memory issues with large array

---

### EPIC 3: TABLE DISPLAY (F7, F9, F10)

#### US-9: Creation and Display of Trades Table
**Story Points**: 20 | **Effort**: 20 min | **Priority**: HIGH | **Dependency**: US-7, US-8

**As a** user,  
**I want** to see all my trades in a clear table on the chart,  
**So that** I can quickly analyze each position.

**Acceptance Criteria**:
- ☑ Table displayed top-right of the chart
- ☑ Exactly 18 columns in specified order (see PO)
- ☑ Headers: dark gray (#2C3E50)
- ☑ WIN rows: light green (#D4EDDA) + black text
- ☑ LOSS rows: light red (#F8D7DA) + black text
- ☑ Correct text alignment (left, center, right per column)
- ☑ Correct numeric formats (2 dec €, 4 dec price, 2 dec %, etc.)
- ☑ Dates in YYYY-MM-DD format
- ☑ ALL trades displayed (no max limit)
- ☑ Scrollable table if > 15 trades
- ☑ Monospace font for column alignment
- ☑ No text overflow
- ☑ Readable on all timeframes and instruments

**Exact Columns** (order):
1. Entry date (YYYY-MM-DD)
2. Exit date (YYYY-MM-DD)
3. Duration (candles) - integer, center
4. Amount invested (€) - float 2 dec, right
5. Avg buy price (€) - float 4 dec, right
6. Exit price (€) - float 4 dec, right
7. Quantity (units) - float 6 dec, right
8. Amount sold (€) - float 2 dec, right
9. Period High - float 4 dec, right
10. Period Low - float 4 dec, right
11. % movement (H-L) - float 2 dec + %, right
12. Total volume - integer, right
13. Average volume - float 0 dec, right
14. P&L (€) - float 2 dec, right
15. P&L (%) - float 2 dec + %, right
16. ROI - float 2 dec + %, right
17. Win/Loss - "WIN"/"LOSS", center, color
18. Gain per candle (€) - float 2 dec, right

**Development Tasks**:
- Create table with table.new() 18 columns
- Loop through trades array, add rows
- Apply WIN/LOSS colors
- Format each value (decimals, %, alignment)
- Test scrolling if > 15 trades
- Validate on mobile (if TradingView supports)

---

#### US-10: Display of Global Summary (Stats)
**Story Points**: 10 | **Effort**: 10 min | **Priority**: HIGH | **Dependency**: US-8

**As a** user,  
**I want** to see a global summary of my strategy (total P&L, win rate, etc.),  
**So that** I can quickly assess overall profitability.

**Acceptance Criteria**:
- ☑ Summary box displayed below or next to the table
- ☑ Light blue background (#E8F4F8)
- ☑ All 14 PO metrics displayed:
  - Total number of trades
  - Winning trades (WIN) - count and %
  - Losing trades (LOSS) - count and %
  - Win Rate (%)
  - Total P&L (€)
  - Total P&L (%)
  - Average P&L per trade (€)
  - Average P&L per trade (%)
  - Average gain of WIN trades (€)
  - Average loss of LOSS trades (€)
  - Gain/Loss ratio
  - Total amount invested (€)
  - Total amount withdrawn (€)
- ☑ Format: "Metric: Value" clear and readable
- ☑ Correct number formatting (2 dec €, 2 dec + %)
- ☑ Division by 0 handled (if 0 LOSS trades, ratio = "N/A")
- ☑ Visible and persistent at bottom of chart

**Development Tasks**:
- Calculate all 14 metrics (see US-11)
- Create label or table for summary
- Format readable text
- Test edge cases (0 trades, all WIN, all LOSS)

---

#### US-11: Global Statistics Calculation
**Story Points**: 12 | **Effort**: 12 min | **Priority**: HIGH | **Dependency**: US-8

**As a** system,  
**I want** to calculate all global performance statistics,  
**So that** the user sees full performance at a glance.

**Acceptance Criteria**:
- ☑ Total trades = array.size(trades)
- ☑ WIN trades = COUNT(P&L > 0)
- ☑ LOSS trades = COUNT(P&L ≤ 0)
- ☑ Win Rate (%) = (WIN trades / Total) × 100
- ☑ Total P&L (€) = SUM(P&L)
- ☑ Total P&L (%) = (Total P&L / SUM total_invested) × 100
- ☑ Average P&L per trade = Total P&L / Total trades
- ☑ Average WIN gain = SUM(P&L > 0) / WIN count
- ☑ Average LOSS loss = SUM(P&L ≤ 0) / LOSS count (|value|)
- ☑ Gain/Loss ratio = Avg gain / |Avg loss|
- ☑ Total amount invested = SUM(total_invested)
- ☑ Total amount withdrawn = SUM(amount_sold)
- ☑ All calculations exact for 100+ trades
- ☑ No division by 0

**Development Tasks**:
- Loop through trades array
- Accumulate appropriate values
- Apply each formula exactly
- Test with 50+ trades (WIN/LOSS mix)

---

### EPIC 4: QUALITY & POLISH (F12, AUX1-3)

#### US-12: Script Optimization and Performance
**Story Points**: 8 | **Effort**: 8 min | **Priority**: MEDIUM | **Dependency**: US-1 → US-11

**As a** user,  
**I want** the script to be fast and lag-free,  
**So that** backtesting is smooth even on large datasets.

**Acceptance Criteria**:
- ☑ Build time < 2 seconds (chart > 1000 bars)
- ☑ No infinite loops
- ☑ No memory errors
- ☑ No unnecessary recalculation on each bar
- ☑ Array size < 10k trades (limit memory if needed)
- ☑ Variables reset properly (no memory leak)
- ☑ Optimized compiled code

**Development Tasks**:
- Profile the script (Pine Script analyzer)
- Identify bottlenecks
- Optimize loops and repeated calculations
- Test on 5000+ bars

---

#### US-13: Visual Format and UX Polish
**Story Points**: 5 | **Effort**: 5 min | **Priority**: MEDIUM | **Dependency**: US-9, US-10

**As a** user,  
**I want** the table and summary to be visually clear and attractive,  
**So that** I enjoy using this tool.

**Acceptance Criteria**:
- ☑ Monospace font for tables (perfect alignment)
- ☑ Colors consistent with TradingView theme
- ☑ Consistent padding and spacing
- ☑ No text overflow
- ☑ Dark mode compatible
- ☑ Responsive across resolutions
- ☑ Appropriate icons/symbols (WIN ✓, LOSS ✗)

**Development Tasks**:
- Test in light/dark mode
- Adjust colors if needed
- Add borders/separators
- Test on 1200p, 1920p, 4k screens

---

#### US-14: Documentation and Code Comments
**Story Points**: 5 | **Effort**: 5 min | **Priority**: LOW | **Dependency**: All

**As a** future dev or curious user,  
**I want** to understand the code and how it works,  
**So that** I can use it, extend it, or fix it.

**Acceptance Criteria**:
- ☑ All code blocks have explanatory comments
- ☑ Functions have docstrings (Inputs, Outputs, Logic)
- ☑ Variables have clear names (buy_candle_count not bc)
- ☑ Mathematical formulas explained inline
- ☑ README.md or wiki with usage guide
- ☑ Usage example (screenshot + explanations)
- ☑ Notation of what is hardcoded vs configurable

**Development Tasks**:
- Add // comments every 5 lines
- Write docstrings for main functions
- Create markdown user guide
- Screenshots of final result

---

## 📈 OPERATIONAL ROADMAP (12 weeks)

### PHASE 0: MVP (Week 1 - 1 dev day)
**Objective**: Launch minimal functional version  
**Features**: F1-F8 (core logic) + F2, F3, F4, F5, F6  
**Deliverables**: Compiled script, tested, basic documentation

| Task | Owner | Effort | Timeline |
|------|-------|--------|----------|
| US-1: Inputs | Dev | 5 min | Day 1 |
| US-3: MACD | Dev | 10 min | Day 1 |
| US-4: Position state | Dev | 8 min | Day 1 |
| US-6: Accumulation | Dev | 12 min | Day 1 |
| US-7: KPI calc | Dev | 15 min | Day 1 |
| US-8: Array storage | Dev | 5 min | Day 1 |
| **Total** | | **55 min** | **Day 1** |

**Phase 0 Acceptance Criteria**:
- ✅ Pine v5 script compiles without error
- ✅ MACD crossovers detected correctly
- ✅ At least 5 trades calculated correctly
- ✅ No crashes
- ✅ Exact P&L calculations (validated vs Excel)

**Exit Criteria**: Proceed to Phase 1 if all AC validated

---

### PHASE 1: STABLE V1 (Week 2–3 - 6h dev + 8h QA)
**Objective**: Stabilize, polish, validate with beta testers  
**Features**: Complete F1-F12 + AUX1-3  
**Deliverables**: Stable v1.0, full documentation, demo

| Task | Owner | Effort | Timeline |
|------|-------|--------|----------|
| US-2: Date conversion | Dev | 8 min | Day 2 |
| US-5: Edge cases | Dev | 8 min | Day 2 |
| US-9: Table display | Dev | 20 min | Day 2–3 |
| US-10: Summary stats | Dev | 10 min | Day 3 |
| US-11: Global stats calc | Dev | 12 min | Day 3 |
| US-12: Performance opt | Dev | 8 min | Day 4 |
| US-13: UX Polish | Dev | 5 min | Day 4 |
| US-14: Documentation | Dev + PM | 10 min | Day 4 |
| QA: 20+ dataset tests | QA | 4h | Day 2–4 |
| Beta testing (5 traders) | QA | 4h | Day 4–5 |

**Total Effort**: ~130 min dev + 8h QA

**Phase 1 Acceptance Criteria**:
- ✅ 100% functional requirements covered
- ✅ 20+ datasets tested, exact results
- ✅ 5 beta testers with positive feedback
- ✅ < 2 sec build time
- ✅ Table visible and scrollable
- ✅ Zero crashes on edge cases
- ✅ Complete documentation

---

### PHASE 2: LAUNCH & ADOPTION (Week 4–8 - Marketing)
**Objective**: Promotion, early adopter acquisition  
**Deliverables**: 3 videos, 2 articles, 500+ impressions

| Task | Owner | Effort | Timeline |
|------|-------|--------|----------|
| Announce TradingView | PM | 30 min | Day 8 |
| Record demo video | Marketing | 2h | Day 8–9 |
| Write blog guide | Content | 1.5h | Day 9–10 |
| Share Reddit/Discord | Community | 1h | Day 11 |
| Reach influencers | Growth | 2h | Day 11–12 |
| Community management | Support | Ongoing | Week 4–8 |

**KPI Targets**:
- 500+ downloads
- 200+ active users
- 50+ community comments
- 10+ shares/retweets

---

### PHASE 3: V2 FEATURES (Week 9–12)
**Objective**: Feedback-driven improvements  
**New Features**: Custom MACD params, commissions, exports

| Task | Owner | Effort | Timeline |
|------|-------|--------|----------|
| Custom MACD inputs | Dev | 1h | Week 9 |
| Commission slider | Dev | 1.5h | Week 10 |
| CSV export | Dev | 2h | Week 10–11 |
| Volatility filters | Dev | 2.5h | Week 11–12 |

**Total Effort**: ~7h dev

**Release**: V2.0 end of week 12

---

## 🔗 CRITICAL DEPENDENCIES

```
US-1 (Inputs)
  └─→ US-3 (MACD calc)
       └─→ US-4 (Position state)
            ├─→ US-6 (Accumulation)
            │    └─→ US-7 (KPI calc)
            │         ├─→ US-8 (Storage)
            │         │    ├─→ US-9 (Table)
            │         │    └─→ US-10 (Summary)
            │         └─→ US-11 (Global stats)
            └─→ US-9 (Table)

US-2 (Dates) ──→ US-9 (Table)
US-5 (Edge cases) ──→ US-12 (Performance)
US-12 (Performance) ──→ Phase 1 validation
US-13 (UX) ──→ Launch readiness
US-14 (Docs) ──→ Launch readiness
```

---

## 🎯 GLOBAL ACCEPTANCE CRITERIA (v1)

| Domain | Criteria | Status |
|---------|---------|--------|
| **Functionality** | All 14 US complete | ⏳ |
| **Calculations** | 99.9% P&L accuracy | ⏳ |
| **Display** | 18 columns, colors, scrolling | ⏳ |
| **Performance** | < 2 sec build, 0 crashes | ⏳ |
| **Robustness** | Edge cases handled, div/0 safe | ⏳ |
| **Docs** | Commented code, README, guide | ⏳ |
| **Testing** | 20+ datasets, 5 beta users | ⏳ |

---

## 📝 NOTES & CONSTRAINTS

### Technical Constraints
- Pine Script v5 only (no v3, v4)
- TradingView sandbox environment
- No external API calls
- Array size max ~10k (memory limit)
- Build time < 5 sec max

### Business Constraints
- v1: no commissions/slippage (idealized)
- Only one open position at a time (no overlapping)
- Fixed MACD params (12-26-9) in v1
- Investment per candle must be positive

### Documented Limitations
- No short positions
- No multi-instrument simultaneous trading
- No broker integration (backtesting only)
- Minimum timeframe: 1 minute
- Minimum dataset: 100 bars

---

## ✅ FINAL VALIDATION

**Document approved by**: Product Owner  
**Next step**: Start Phase 0 (development)  
**Phase 0 deadline**: 1 day  
**Phase 1 deadline**: 4 days after MVP  
**Launch deadline**: 14 days after MVP  

**Status**: 🚀 **READY FOR SPRINT ZERO**

Complete roadmap, detailed user stories, clear dependencies, estimated effort.

---

**Author**: Product Owner  
**Date**: December 29, 2025  
**Version**: 2.0 (Final)
