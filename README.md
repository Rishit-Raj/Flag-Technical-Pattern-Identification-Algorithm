# Flag Technical Pattern Identification Algorithm

A Python implementation that automatically detects **Bull Flag** and **Bear Flag** chart formations from financial time series data. Two independent detection methods are provided — a **PIP-based** approach and a **trendline regression** approach — along with built-in backtesting analytics to measure post-pattern returns.

---

## What Are Flag Patterns?

Flag patterns are short-term continuation formations that appear after a strong, sharp price move (the **pole**), followed by a brief consolidation period (the **flag body**), and then a breakout resuming the original trend direction.

### Bull Flag
- Appears after a sharp **upward** move (the pole)
- Price consolidates in a **downward-sloping channel** (flag)
- Confirmed when price breaks **above** the upper consolidation boundary
- Signals continuation of the uptrend

### Bear Flag
- Appears after a sharp **downward** move (the pole)
- Price consolidates in an **upward-sloping channel** (flag)
- Confirmed when price breaks **below** the lower consolidation boundary
- Signals continuation of the downtrend

**Key geometric constraints applied in this implementation:**
- Flag width must be less than 50% of the pole width
- Flag height must be less than 50–75% of the pole height (method-dependent)

---

## Project Structure

```
flags.py                   # Main pattern detection, backtesting, and visualization
perceptually_important.py  # Perceptually Important Points (PIP) extraction algorithm
trendline_automation.py    # Automated trendline fitting via slope optimization
```

---

## Detection Methods

### 1. PIP-Based Detection (`find_flags_pips`)
Uses **Perceptually Important Points (PIPs)** — a technique that iteratively selects the most structurally significant price points by maximizing perpendicular, vertical, or Euclidean distance from a connecting line. Five PIPs are extracted from the flag body and their geometric relationships (alternating peaks/troughs, slope signs, line intersection) are validated to confirm or reject each candidate pattern.

### 2. Trendline Regression Detection (`find_flags_trendline`)
Fits automated **support and resistance trendlines** to the flag body using least-squares regression as an initial estimate, then refines slopes via numerical optimization to ensure the lines are tangent to price extrema (i.e., no price bars violate the trendline). Pattern confirmation is triggered by a breakout of the upper (bull) or lower (bear) trendline.

Both methods share the same `FlagPattern` dataclass for consistent output.

---

## FlagPattern Dataclass

Each detected pattern is stored as a `FlagPattern` object with the following fields:

| Field | Description |
|---|---|
| `base_x / base_y` | Start of the pole (index and price) |
| `tip_x / tip_y` | End of pole / start of flag body |
| `conf_x / conf_y` | Breakout confirmation point |
| `pole_width / pole_height` | Dimensions of the pole |
| `flag_width / flag_height` | Dimensions of the consolidation body |
| `support_slope / support_intercept` | Lower trendline of the flag |
| `resist_slope / resist_intercept` | Upper trendline of the flag |

---

## Backtesting

The main script includes a backtesting framework that measures **log returns** following each confirmed pattern. A `hold_mult` parameter controls the holding period as a multiple of the flag's width. Results are assembled into `pandas` DataFrames for each pattern type (`bull_flags`, `bear_flags`) capturing:

- Flag dimensions (width, height)
- Pole dimensions
- Channel slope
- Forward return over the holding period

Price data is log-transformed before analysis, making returns arithmetic and resolving price-scaling issues across long time series.

---

## Modules

### `perceptually_important.py`
Implements the `find_pips(data, n_pips, dist_measure)` function. Iteratively selects the `n_pips` most structurally important price points using one of three distance measures:
- `1` — Euclidean distance
- `2` — Perpendicular distance
- `3` — Vertical distance

### `trendline_automation.py`
Implements automated trendline fitting:
- `fit_trendlines_single(data)` — fits support/resistance lines to a single close-price series
- `fit_trendlines_high_low(high, low, close)` — fits lines using high/low wicks for tighter bounds
- `optimize_slope(support, pivot, init_slope, y)` — numerical slope optimizer that iteratively refines the trendline until it is tangent to the relevant price extrema

---

## Dependencies

```bash
pip install pandas numpy matplotlib mplfinance
```

---

## Notes

- This project is for **educational purposes** and does **not constitute financial advice**.
- Log-transforming price data before running detection is recommended for stationarity and scale invariance.
- The `order` parameter controls rolling window sensitivity — larger values detect longer-term patterns with fewer false positives.
- Works on any OHLCV time series; tested on BTC/USDT hourly and daily data.

---

## License

This project is open source.
