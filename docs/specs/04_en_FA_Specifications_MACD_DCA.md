# 🏛️ FUNCTIONAL ANALYST – DETAILED FUNCTIONAL SPECIFICATIONS
## MACD DCA Strategy for TradingView (Pine Script v5)

**Version**: 1.0  
**Date**: December 29, 2025  
**Author**: Functional Analyst  
**Status**: ✅ Validated – Ready for TA & Dev  

---

## 📌 EXECUTIVE SUMMARY

This document contains:
- ✅ Detailed specifications for EACH feature
- ✅ Data model (Trade structure, global variables)
- ✅ Application flows and workflows
- ✅ Display mockups/wireframes
- ✅ Business rules and logic

---

## 1️⃣ DATA MODEL

### Trade Structure (Type Definition)

```
type Trade
    // Dates
    entry_date : string              // "YYYY-MM-DD"
    exit_date : string               // "YYYY-MM-DD"
    entry_bar_index : int            // bar_index of bullish crossover
    exit_bar_index : int             // bar_index of bearish crossover
    
    // Period & Duration
    candle_count : int               // Number of bars (buy_candle_count)
    
    // Investment & Accumulation
    total_invested : float           // investment_per_candle × candle_count
    sum_mid_prices : float           // Σ (H+L)/2 (for average price calc)
    average_price : float            // total_invested / candle_count
    quantity : float                 // total_invested / average_price
    
    // Exit & P&L
    exit_price : float               // Close[exit_bar]
    amount_sold : float              // quantity × exit_price
    pnl : float                      // amount_sold - total_invested
    pnl_percent : float              // (pnl / total_invested) × 100
    roi : float                      // Same as pnl_percent
    
    // Movement & Volume
    high_period : float              // MAX(High) over entire period
    low_period : float               // MIN(Low) over entire period
    movement_percent : float         // ((high - low) / low) × 100
    volume_total : float             // Σ Volume
    volume_avg : float               // volume_total / candle_count
    
    // Analytics
    gain_per_candle : float          // pnl / candle_count
    win_loss_text : string           // "WIN" or "LOSS"
```

### Global Variables (System State)

```
// Configuration (Inputs)
var float investment_per_candle     // Default 5.0
var int macd_fast                    // Default 12
var int macd_slow                    // Default 26
var int macd_signal                  // Default 9

// MACD Calculation
var float macd                       // EMA(close, fast) - EMA(close, slow)
var float signal                     // EMA(macd, signal)
var float histogram                  // macd - signal
var bool crossover_up                // ta.crossover(macd, signal)
var bool crossover_down              // ta.crossunder(macd, signal)

// Trade State
var bool position_open               // true = trade active, false = closed
var int entry_bar_index              // bar_index of bullish crossover
var int exit_bar_index               // bar_index of bearish crossover

// Accumulation (while trade open)
var int buy_candle_count             // Number of buy bars (0 initially)
var float sum_mid_prices             // Σ (H+L)/2
var float high_period_temp           // MAX(High)
var float low_period_temp            // MIN(Low)
var float volume_total_temp          // Σ Volume

// Storage
var array<Trade> trades              // Array of all closed trades

// Display
var table trades_table               // Pine table for trades
var box summary_box                  // Global summary box
```

---

## 2️⃣ FUNCTIONAL SPECIFICATIONS (F1 → F12)

### F1: CONFIGURATION & INPUTS

**Function**: Allow user to configure strategy without code

**Detailed Specification**:

| Input | Type | Default | Min | Max | Step | Visible |
|------|------|---------|-----|-----|------|---------|
| investment_per_candle | float | 5.0 | 0.01 | 10000 | 0.01 | ✅ |
| macd_fast | int | 12 | 1 | 50 | 1 | ✅ |
| macd_slow | int | 26 | 1 | 100 | 1 | ✅ |
| macd_signal | int | 9 | 1 | 50 | 1 | ✅ |

**Logic**:
- Inputs declared in `indicator()` section
- Groups: "MACD Settings" and "Investment Settings"
- Tooltip: short explanation for each

**Validation**:
- investment_per_candle > 0
- macd_fast < macd_slow
- All int/float values valid

---

### F2: MACD CALCULATION & CROSSOVER DETECTION

**Function**: Detect bullish/bearish MACD signals

**Detailed Pseudocode**:

```
EACH BAR:
  1. Calculate MACD
     macd = ta.ema(close, macd_fast) - ta.ema(close, macd_slow)
  
  2. Calculate Signal
     signal = ta.ema(macd, macd_signal)
  
  3. Calculate Histogram
     histogram = macd - signal
  
  4. Detect crossovers
     crossover_up = ta.crossover(macd, signal)    // bullish
     crossover_down = ta.crossunder(macd, signal) // bearish
  
  5. Store for later access
     (global variables updated)
```

**Output**:
- `macd`: float
- `signal`: float
- `histogram`: float
- `crossover_up`: bool
- `crossover_down`: bool

**Validation**:
- Compare with native TradingView MACD → identical

---

### F3: POSITION STATE MANAGEMENT

**Function**: Track position open/close state

**State Machine Logic**:

```
Initial State: position_open = false

IF position_open == false AND crossover_up == true:
  → position_open = true
  → entry_bar_index = bar_index
  → Initialize accumulators (see F4)

IF position_open == true AND crossover_down == true:
  → position_open = false
  → exit_bar_index = bar_index
  → Calculate KPIs (see F5)
  → Create Trade object and add it (see F6)
  → Reset accumulators
```

**Constraints**:
- Ignore duplicate crossovers (no two bullish in a row)
- No trade if position already open
- Position remains open at end of backtest (no auto-close)

---

### F4: DATA ACCUMULATION DURING POSITION

**Function**: Collect OHLCV data for statistics

**Pseudocode**:

```
IF position_open == true:
  1. Increment counter
     buy_candle_count += 1
  
  2. Calculate mid-price
     mid_price = (high + low) / 2
     sum_mid_prices += mid_price
  
  3. Track High/Low
     high_period_temp = max(high_period_temp, high)
     low_period_temp = min(low_period_temp, low)
  
  4. Accumulate volume
     volume_total_temp += volume
```

**Timing**: On each bar where position_open == true

**Reset**: When position closes
```
buy_candle_count = 0
sum_mid_prices = 0.0
high_period_temp = 0.0
low_period_temp = 0.0
volume_total_temp = 0.0
```

---

### F5: FULL KPI CALCULATION

**Function**: Calculate 18 KPIs per trade

**Exact Formulas**:

| KPI | Formula | Example |
|-----|---------|---------|
| Duration | buy_candle_count | 3 |
| Total invested | investment_per_candle × buy_candle_count | 5 × 3 = 15€ |
| Avg buy price | sum_mid_prices / buy_candle_count | (100+101.5+102.5)/3 = 101.33€ |
| Quantity | total_invested / average_price | 15/101.33 = 0.148 units |
| Exit price | close[exit_bar] | 102€ |
| Amount sold | quantity × exit_price | 0.148 × 102 = 15.10€ |
| P&L € | amount_sold - total_invested | 15.10 - 15 = +0.10€ |
| P&L % | (P&L / total_invested) × 100 | +0.67% |
| ROI | P&L % | +0.67% |
| Period high | MAX(high buy + sell bars) | 105€ |
| Period low | MIN(low buy + sell bars) | 99€ |
| % movement | ((high - low) / low) × 100 | +6.06% |
| Total volume | Σ volume | 4200 |
| Avg volume | volume_total / buy_candle_count | 1400 |
| Gain/bar | P&L / buy_candle_count | +0.033€ |
| Win/Loss | P&L > 0 ? "WIN" : "LOSS" | "WIN" |

**Decimal Rules**:
- € values: 2 decimals
- Prices: 4 decimals
- %: 2 decimals
- Quantities: 6 decimals
- Integers: 0 decimals

---

### F6: TRADE STORAGE

**Function**: Persist each trade in array

**Pseudocode**:

```
WHEN position closes (bearish crossover):
  1. Create new Trade
     trade = Trade.new(
       entry_date: convert entry_bar_index,
       exit_date: convert exit_bar_index,
       candle_count: buy_candle_count,
       total_invested: investment_per_candle × buy_candle_count,
       average_price: sum_mid_prices / buy_candle_count,
       quantity: total_invested / average_price,
       exit_price: close,
       amount_sold: quantity × close,
       pnl: amount_sold - total_invested,
       pnl_percent: (pnl / total_invested) × 100,
       high_period: high_period_temp,
       low_period: low_period_temp,
       movement_percent: ((high-low)/low) × 100,
       volume_total: volume_total_temp,
       volume_avg: volume_total_temp / buy_candle_count,
       gain_per_candle: pnl / buy_candle_count,
       win_loss_text: pnl > 0 ? "WIN" : "LOSS"
     )
  
  2. Add to array
     array.push(trades, trade)
```

**Checks**:
- No array size limit
- Persists between bars (`var`)
- Efficient access via `array.get()`

---

### F7: DATE CONVERSION TIMESTAMPS → YYYY-MM-DD

**Function**: Convert bar_index to readable date

**Pseudocode**:

```
function f_barindex_to_datestring(bar_index) =>
    timestamp = time[bar_index]
    datestring = str.format("{0,date,yyyy-MM-dd}", timestamp / 1000)
    datestring
```

**Details**:
- Input: bar_index (int)
- Output: string "YYYY-MM-DD"
- time in milliseconds
- Divide by 1000 for seconds

**Validation**:
- Test on 5+ timeframes
- Verify bar/date match

---

### F8 → F11: DISPLAY & STATS (see WIREFRAMES section 3)

---

### F12: EDGE CASE HANDLING

**Case 1**: Division by zero
```
IF buy_candle_count == 0:
  average_price = 0
  
IF volume == 0:
  volume_avg = 0
  
IF total_invested == 0:
  pnl_percent = 0
```

**Case 2**: Gain/Loss ratio without LOSS
```
IF count_loss == 0:
  ratio = "N/A"
ELSE:
  ratio = avg_gain / abs(avg_loss)
```

**Case 3**: Open position at end of backtest
```
IF position_open == true AND last_bar:
  → Do nothing
  → Do not auto-close
  → Do not display in table
```

**Case 4**: 0 trades
```
IF array.size(trades) == 0:
  → Display empty table (headers only)
  → Display summary "0 trades"
  → No error
```

**Case 5**: Duplicate crossovers
```
IF position_open == true AND crossover_up == true:
  → Ignore
  
IF position_open == false AND crossover_down == true:
  → Ignore
```

---

## 3️⃣ WIREFRAMES & DISPLAY LAYOUT

### Trade Table Wireframe

```
╔═══════════════════════════════════════════════════════════════════════╗
║ MACD DCA Strategy - Trade Table                                       ║
╠═══════════════════════════════════════════════════════════════════════╣
║ Entry   Exit    Bars   Inv.  Avg Price  Exit Price Qty    Sold  ... ║
║ Date    Date    (int)  (€)   (€)        (€)        (units) (€)      ║
╠═══════════════════════════════════════════════════════════════════════╣
║ 2025-12-01 2025-12-05 5    25.00 100.00 98.50 0.250000 24.63 ... ║ ← WIN
║ 2025-12-10 2025-12-14 4    20.00 102.50 101.00 0.195121 19.70 ... ║ ← LOSS
║ 2025-12-20 2025-12-25 6    30.00 99.50  103.00 0.301508 31.06 ... ║ ← WIN
╚═══════════════════════════════════════════════════════════════════════╝
```

**18 Columns (exact order)**:

| # | Column | Format | Align | Width |
|---|--------|--------|-------|-------|
| 1 | Entry Date | YYYY-MM-DD | left | 11 |
| 2 | Exit Date | YYYY-MM-DD | left | 11 |
| 3 | Bars | int | center | 8 |
| 4 | Invested | 0.00€ | right | 10 |
| 5 | Avg Price | 0.0000€ | right | 11 |
| 6 | Exit Price | 0.0000€ | right | 11 |
| 7 | Qty | 0.000000 | right | 11 |
| 8 | Amount Sold | 0.00€ | right | 12 |
| 9 | Period High | 0.0000€ | right | 11 |
|10 | Period Low | 0.0000€ | right | 11 |
|11 | % Move | 0.00% | right | 8 |
|12 | Total Volume | int | right | 10 |
|13 | Avg Volume | 0 | right | 9 |
|14 | P&L € | 0.00€ | right | 10 |
|15 | P&L % | 0.00% | right | 8 |
|16 | ROI | 0.00% | right | 8 |
|17 | Win/Loss | WIN/LOSS | center | 6 |
|18 | Gain/Bar | 0.00€ | right | 10 |

**Styles**:
- Header bg: #2C3E50
- WIN row bg: #D4EDDA
- LOSS row bg: #F8D7DA
- Font: Monospace
- Position: top-right of chart

---

### Global Summary Wireframe

```
╔════════════════════════════════════════════╗
║     GLOBAL SUMMARY - BACKTEST              ║
╠════════════════════════════════════════════╣
║ Total trades               : 15            ║
║ Winning trades (WIN)       : 9 (60%)       ║
║ Losing trades (LOSS)       : 6 (40%)       ║
║ Win Rate                   : 60.00%        ║
║                                            ║
║ Total P&L (€)              : +487.50       ║
║ Total P&L (%)              : +12.34%       ║
║ Avg P&L per trade (€)      : +32.50        ║
║ Avg P&L per trade (%)      : +0.82%        ║
║                                            ║
║ Avg WIN gain               : +72.50€       ║
║ Avg LOSS                   : -45.00€       ║
║ Gain/Loss Ratio            : 1.61          ║
║                                            ║
║ Total invested             : 3950.00€      ║
║ Total withdrawn            : 4437.50€      ║
╚════════════════════════════════════════════╝
```

---

## 4️⃣ DETAILED APPLICATION FLOW

### Main Flow (Each Bar)

```
BAR N:
┌─────────────────────────────────────┐
│ 1. Retrieve OHLCV data              │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ 2. Calculate MACD & Crossovers      │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ 3. State Machine                    │
│    open / close position            │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ 4. IF position OPEN                 │
│    accumulate data                  │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ 5. IF position CLOSED this bar      │
│    calculate KPIs & store trade     │
└─────────────────────────────────────┘
            ↓
┌─────────────────────────────────────┐
│ 6. Update display                   │
└─────────────────────────────────────┘
```

---

## 5️⃣ VALIDATION MATRIX (Requirements → Specs)

| FA ID | Specification | US ID | Criterion |
|-------|---------------|-------|-----------|
| F1 | Input params | US-1 | 4 visible inputs |
| F2 | MACD calc | US-3 | Exact vs native |
| F3 | Position state | US-4 | Correct state machine |
| F4 | Accumulation | US-6 | buy_candle_count, H/L, Vol |
| F5 | KPI calc | US-7 | 18 KPIs exact |
| F6 | Trade storage | US-8 | Array persistence |
| F7 | Date conversion | US-2 | YYYY-MM-DD |
| F8-F10 | Display | US-9, US-10 | Table + summary |
| F11 | Global stats | US-11 | 14 metrics |
| F12 | Edge cases | US-5 | Robustness |

---

## ✅ FA VALIDATION

**Approved by**: Functional Analyst  
**Next step**: Technical Architecture (TA)  
**Status**: 🚀 **COMPLETE FUNCTIONAL SPECIFICATIONS**

---

**Author**: Functional Analyst  
**Date**: December 29, 2025  
**Version**: 1.0 Final
