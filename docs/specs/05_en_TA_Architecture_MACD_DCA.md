# 🏗️ TECHNICAL ARCHITECT - TECHNICAL ARCHITECTURE & DESIGN
## MACD DCA Strategy for TradingView (Pine Script v5)

**Version**: 1.0  
**Date**: December 29, 2025  
**Author**: Technical Architect  
**Status**: ✅ Approved – Ready for development  

---

## 📌 EXECUTIVE SUMMARY

This document contains:
- ✅ Overall system architecture (single-file Pine Script)
- ✅ Justified technology choices (Pine Script v5, TradingView)
- ✅ Applied design patterns
- ✅ Code structure & module organization
- ✅ Performance, scalability, security, maintainability
- ✅ Technical risks & mitigation

---

## 1️⃣ OVERALL ARCHITECTURE

### System Overview

```
┌────────────────────────────────────────────────────────┐
│         TradingView Pine Script v5 Strategy            │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌──────────────────────────────────────────────────┐ │
│  │  CONFIGURATION LAYER                           │ │
│  │  ├─ Inputs (investment, MACD params)           │ │
│  │  └─ Indicators setup                           │ │
│  └──────────────────────────────────────────────────┘ │
│                         ↓                             │
│  ┌──────────────────────────────────────────────────┐ │
│  │  SIGNAL DETECTION LAYER                        │ │
│  │  ├─ MACD & Signal calculation (ta.ema)         │ │
│  │  ├─ Crossover detection (ta.crossover)         │ │
│  │  └─ Signal routing                             │ │
│  └──────────────────────────────────────────────────┘ │
│                         ↓                             │
│  ┌──────────────────────────────────────────────────┐ │
│  │  TRADE STATE MANAGEMENT LAYER                  │ │
│  │  ├─ Position open/close state machine          │ │
│  │  ├─ Entry/exit point tracking                  │ │
│  │  └─ Trade lifecycle management                 │ │
│  └──────────────────────────────────────────────────┘ │
│                         ↓                             │
│  ┌──────────────────────────────────────────────────┐ │
│  │  DATA ACCUMULATION LAYER                       │ │
│  │  ├─ OHLCV data collection                      │ │
│  │  ├─ Aggregation (sum, min, max)                │ │
│  │  └─ Running totals                             │ │
│  └──────────────────────────────────────────────────┘ │
│                         ↓                             │
│  ┌──────────────────────────────────────────────────┐ │
│  │  CALCULATION & ANALYTICS LAYER                 │ │
│  │  ├─ KPI calculation (18 metrics)               │ │
│  │  ├─ Statistics (win rate, drawdown, etc.)      │ │
│  │  └─ Global aggregation                         │ │
│  └──────────────────────────────────────────────────┘ │
│                         ↓                             │
│  ┌──────────────────────────────────────────────────┐ │
│  │  PERSISTENCE & STORAGE LAYER                   │ │
│  │  ├─ Trade object creation                      │ │
│  │  ├─ Array storage (in-memory)                  │ │
│  │  └─ State persistence (var keyword)            │ │
│  └──────────────────────────────────────────────────┘ │
│                         ↓                             │
│  ┌──────────────────────────────────────────────────┐ │
│  │  PRESENTATION LAYER                            │ │
│  │  ├─ Table rendering (18 columns)               │ │
│  │  ├─ Summary box display (14 metrics)           │ │
│  │  └─ Color formatting (WIN/LOSS)                │ │
│  └──────────────────────────────────────────────────┘ │
│                                                        │
└────────────────────────────────────────────────────────┘
```

### Architectural Principles
- **Single-file**: One .pine file, no external dependencies
- **Separation of Concerns**: 7 layers, each responsible for a single function
- **Stateless but not Stateful**: State persisted using the `var` keyword
- **Event-driven**: Reactive to MACD signals
- **Transparent**: Every calculation is visible, traceable, and reproducible

---

## 2️⃣ TECHNOLOGY CHOICES

### Primary Technology: Pine Script v5

**Justification**:
- ✅ **TradingView Native**: Direct execution on the platform
- ✅ **Supported**: Pine v5 = latest version, long-term support
- ✅ **Performance**: Optimized compiled execution, fast build (< 2 sec)
- ✅ **Data Access**: Native `time[]`, `close`, `high`, `low`, `volume`
- ✅ **Indicators**: `ta.*` library (EMA, crossover)
- ✅ **UI Widgets**: `table`, `box`, `label` for display
- ✅ **Type System**: Custom types (`Trade`), type safety

**Rejected Alternatives**:
- ❌ Python: Not native to TradingView
- ❌ JavaScript: Not compatible with Pine
- ❌ C++: Overkill, unnecessary compilation

### Target Version: Pine Script v5 Only
- ✅ v5 = latest, all features available
- ❌ v4 = deprecated features
- ❌ v3 = too old, limited features

### Infrastructure: TradingView Cloud
- ✅ Chart rendering
- ✅ Data access (historical, real-time)
- ✅ Execution sandbox
- ✅ User interface (Settings, table display)

---

## 3️⃣ DESIGN PATTERNS

### Pattern 1: State Machine
**Applied to**: Position management (open/closed)

```
State Diagram:
  ┌─────────────────────────────────────────────────┐
  │                                                 │
  v                                                 │
[CLOSED] ──(crossover_up)--> [OPEN] ──(crossover_down)--> [CLOSED]
  │                                                 │
  │                        └─────────────────────────┘
  │
  └─ Initialize trade
```

**Benefits**:
- Clear state transitions
- No overlapping positions
- Easy to debug & audit

### Pattern 2: Data Accumulation
**Applied to**: OHLCV collection during trade

```
Pattern:
  Open position
    ├─ Bar 1: accumulate H1, L1, V1
    ├─ Bar 2: accumulate H2, L2, V2
    ├─ Bar 3: accumulate H3, L3, V3
    └─ Close: max(H1-H3), min(L1-L3), sum(V1-V3)
```

**Benefits**:
- Efficient single-pass calculation
- No re-scanning needed
- Memory efficient (running totals)

### Pattern 3: Object Composition
**Applied to**: Trade type with all metrics

```
type Trade
  ├─ Metadata (dates, indices)
  ├─ Investment data (amount, avg_price, qty)
  ├─ Exit data (price, amount_sold, P&L)
  ├─ Statistics (H/L, volume, movement)
  └─ Analytics (win/loss, ratio, etc.)
```

**Benefits**:
- Cohesive data structure
- Easy to iterate over trades
- Type-safe

### Pattern 4: Layered Architecture
**Applied to**: 7 layers (Configuration → Presentation)

**Benefits**:
- Testable
- Maintainable
- Clear dependency flow

---

## 4️⃣ CODE STRUCTURE

### File Organization

```
// ===== SECTION 1: HEADER & METADATA =====
//@version=5
indicator("MACD DCA Strategy", overlay=false, max_bars_back=200)

// ===== SECTION 2: DOCUMENTATION =====
// Full description, usage, version history, author

// ===== SECTION 3: TYPES & CONSTANTS =====
type Trade
    [18 fields as specified]

const float EPSILON = 0.0001  // Division safety

// ===== SECTION 4: INPUTS & CONFIGURATION =====
investment_per_candle = input.float(...)
macd_fast = input.int(...)
[etc.]

// ===== SECTION 5: GLOBAL VARIABLES (State) =====
var bool position_open = false
var int entry_bar_index = na
var float high_period_temp = 0.0
[etc.]

// ===== SECTION 6: HELPER FUNCTIONS =====
f_barindex_to_datestring(bar_index) => [...]
f_calculate_kpis(trade) => [...]
[etc.]

// ===== SECTION 7: MAIN LOGIC (On-bar) =====
// Execute on every bar

// Step 1: Calculate MACD
macd = ta.ema(...)
signal = ta.ema(...)
crossover_up = ta.crossover(...)
crossover_down = ta.crossunder(...)

// Step 2: State machine
if not position_open and crossover_up
    [open position]
else if position_open and crossover_down
    [close position]

// Step 3: Accumulation (if open)
if position_open
    [accumulate OHLCV]

// ===== SECTION 8: DISPLAY =====
// Update table and summary

// ===== END OF SCRIPT =====
```

### Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| **Variables** | snake_case | `buy_candle_count` |
| **Constants** | UPPER_SNAKE | `EPSILON = 0.0001` |
| **Functions** | f_description | `f_barindex_to_datestring()` |
| **Types** | PascalCase | `type Trade` |
| **Booleans** | is_, has_, should_ | `position_open` |

---

## 5️⃣ PERFORMANCE & SCALABILITY

### Algorithmic Complexity

| Operation | Complexity | Justification |
|-----------|-----------|---------------|
| MACD Calculation | O(n) where n = period | `ta.ema()` is O(n), run once per bar |
| Crossover Detection | O(1) | Simple boolean check |
| Accumulation | O(1) per bar | Running totals |
| KPI Calculation | O(1) | Fixed 18 calculations |
| Array insertion | O(1) amortized | `array.push()` |
| Table rendering | O(m) where m = trades | Loop over trades, once per bar |
| Stats calculation | O(m) | Loop over trades, once per bar |

**Total per bar**: O(max(n, m)) = O(n) typically  
**Overall**: O(N × n) where N = bars, n = MACD period  
→ Linear time, acceptable

### Technical Limits

| Metric | Limit | Justification |
|--------|-------|---------------|
| Max array size (trades) | ~10,000 | TradingView memory limit |
| Max bars processed | ~5,000 | Performance (< 2 sec build) |
| Max table rows displayed | Screen-limited | Scrollable solution |
| MACD period (fastest) | Min 1 | Practical: usually ≥ 5 |
| Build time | < 2 seconds | UX requirement |

**Scalability**:
- ✅ Handles 5000+ bars
- ✅ Handles 10,000 trades
- ✅ Handles any timeframe
- ✅ Handles any instrument

---

## 6️⃣ SECURITY & ROBUSTNESS

### Input Validation

```
investment_per_candle:
  ├─ Min: > 0.01
  ├─ Max: 10,000
  └─ Action: Reject invalid values

macd_fast:
  ├─ Min: 1
  ├─ Max: 50
  ├─ Constraint: < macd_slow
  └─ Action: Warn if invalid

macd_slow:
  ├─ Min: 1
  ├─ Max: 100
  └─ Action: Reject if ≥ 200
```

### Division by Zero Protection

```
EVERYWHERE division occurs:
  if denominator > 0.0001
    result = numerator / denominator
  else
    result = 0.0 (or n/a)
```

### Edge Case Handling

| Edge Case | Handler | Result |
|-----------|---------|--------|
| Volume = 0 | if vol > 0 | Skip, no error |
| Price = 0 | if price > 0 | Skip |
| Empty array | if array.size() > 0 | Empty table |
| Position open at end | Do nothing | No auto-close |
| Duplicate signals | State check | Ignore 2nd signal |

### Data Integrity

- ✅ No data modification (read-only)
- ✅ Immutable trades (no update after creation)
- ✅ Type safety (Trade type enforced)
- ✅ Chronological order (trades added in order)

---

## 7️⃣ MAINTAINABILITY & DOCUMENTATION

### Code Comments Strategy

```
// ===== SECTION NAME =====          // Major sections

// Subsection description           // Subsections

variable = value  // Inline explanation (if not obvious)

if condition
    // Complex logic explanation
    [multi-line logic]
```

### Function Documentation

```
// ==== FUNCTION: f_calculate_kpis ====
// Purpose: Calculate all 18 KPIs for a trade
// Inputs:
//   - buy_candle_count (int): # bars with buy
//   - investment_per_candle (float): € per candle
//   - sum_mid_prices (float): Σ (H+L)/2
// Outputs:
//   - Returns a Trade object with all KPIs
// Formula: See README for detailed formulas

f_calculate_kpis() =>
    [implementation]
```

### Version History

```
V1.0 (2025-12-29):
  ✅ Initial release
  ✅ 12 core logic features
  ✅ MVP status
  ✅ 45–75 min development

V1.1 (Future):
  🔄 Bug fixes based on feedback
  🔄 Performance optimization
  🔄 Documentation improvements

V2.0 (Future):
  ✨ Custom MACD inputs
  ✨ Commission/slippage
  ✨ CSV export
  ✨ Additional filters
```

---

## 8️⃣ TESTING STRATEGY

### Unit Test Approach (Manual)

```
Test MACD Calculation:
  ├─ Input: Close prices [100, 101, 102, 103, 102, 101, 100]
  ├─ Expected: MACD values match TradingView indicator
  └─ Method: Visual comparison on chart

Test KPI Calculation:
  ├─ Input: Example from PO (3 buys, specific prices)
  ├─ Expected: All 18 KPIs match Excel calculation
  └─ Method: Manual Excel validation

Test Edge Cases:
  ├─ Volume = 0: Should not crash
  ├─ Empty array: Table should show "No trades"
  ├─ Position open at end: Should not auto-close
  └─ Divide by 0: Should use default value

Test Integration:
  ├─ 5+ instruments (BTC, EURUSD, AAPL, etc.)
  ├─ 5+ timeframes (1m, 5m, 1h, 4h, 1d)
  └─ 20+ historical datasets
```

### Performance Testing

```
Metric: Build Time
  ├─ Small dataset (500 bars): < 500 ms
  ├─ Medium dataset (2000 bars): < 1 sec
  ├─ Large dataset (5000 bars): < 2 sec
  └─ Target: All < 2 sec

Metric: Memory Usage
  ├─ 100 trades: < 1 MB
  ├─ 1000 trades: < 10 MB
  ├─ 10000 trades: < 100 MB
  └─ Target: Within TradingView limits

Metric: Display Speed
  ├─ Table render: < 100 ms
  ├─ Stats recalc: < 100 ms
  └─ Total: < 200 ms per bar
```

---

## 9️⃣ TECHNICAL RISKS & MITIGATION

### Risk 1: Incorrect P&L Calculation
**Probability**: Medium | **Impact**: Very High

| Aspect | Detail |
|--------|--------|
| **Cause** | Formula error or rounding |
| **Mitigation** | ✅ Excel validation against 20 trades |
| **Fallback** | ✅ Pair code review |
| **Detection** | ✅ Automated unit tests |

### Risk 2: Performance Lag
**Probability**: Low | **Impact**: Medium

| Aspect | Detail |
|--------|--------|
| **Cause** | Too many iterations, oversized arrays |
| **Mitigation** | ✅ Profiling, loop optimization |
| **Fallback** | ✅ Limit array size if needed |
| **Detection** | ✅ Build time < 2 sec |

### Risk 3: Crash on Edge Cases
**Probability**: Medium | **Impact**: High

| Aspect | Detail |
|--------|--------|
| **Cause** | Division by zero, null arrays, edge TFs |
| **Mitigation** | ✅ Explicit edge case handling |
| **Fallback** | ✅ Graceful degradation (default = 0) |
| **Detection** | ✅ 20+ edge case tests |

### Risk 4: TradingView Update Incompatibility
**Probability**: Rare | **Impact**: Very High

| Aspect | Detail |
|--------|--------|
| **Cause** | API change, deprecation |
| **Mitigation** | ✅ Monitor TradingView changelog |
| **Fallback** | ✅ Version compatibility check |
| **Detection** | ✅ Test after each TradingView update |

### Risk 5: User Error (Bad Configuration)
**Probability**: High | **Impact**: Low

| Aspect | Detail |
|--------|--------|
| **Cause** | User enters invalid parameters |
| **Mitigation** | ✅ Input validation, defaults |
| **Fallback** | ✅ Warnings, clear tooltips |
| **Detection** | ✅ Testing with novice users |

---

## 🔟 TECHNICAL ROADMAP

### Phase 0: MVP Build (1 day)
| Task | Owner | Duration |
|------|-------|----------|
| Setup Pine project | Dev | 30 min |
| Implement F1–F6 (core) | Dev | 2 hours |
| Basic testing | Dev | 30 min |
| **Total** | | **3 hours** |

### Phase 1: Stabilization (3 days)
| Task | Owner | Duration |
|------|-------|----------|
| Implement F7–F11 (display) | Dev | 2 hours |
| Edge case handling (F12) | Dev | 1 hour |
| Performance optimization | Dev | 1 hour |
| QA testing (20+ datasets) | QA | 4 hours |
| Documentation | Tech Writer | 1 hour |
| **Total** | | **9 hours** |

### Phase 2: Launch (3 days)
| Task | Owner | Duration |
|------|-------|----------|
| Final polishing | Dev | 1 hour |
| Publish to TradingView | PM | 30 min |
| Documentation release | Tech Writer | 30 min |
| Support monitoring | Support | 8 hours |
| **Total** | | **10 hours** |

---

## ✅ TECHNICAL VALIDATION

| Aspect | Criteria | Status |
|--------|----------|--------|
| **Architecture** | Layered, clear separation | ✅ |
| **Technology** | Pine v5, TradingView native | ✅ |
| **Patterns** | State machine, composition | ✅ |
| **Performance** | O(n), < 2 sec build | ✅ |
| **Scalability** | 5000+ bars, 10k trades | ✅ |
| **Security** | Input validation, edge cases | ✅ |
| **Maintainability** | Documented, organized | ✅ |
| **Testing** | Unit + integration planned | ✅ |
| **Risk Mitigation** | 5 major risks handled | ✅ |

**Status**: 🚀 **ARCHITECTURE APPROVED – READY FOR DEVELOPMENT**

---

**Author**: Technical Architect  
**Date**: December 29, 2025  
**Version**: 1.0 Final  
**Approval**: Technical Lead + Product Owner
