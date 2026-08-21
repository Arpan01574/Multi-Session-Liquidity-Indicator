<div align="center">

# Multi-Session Liquidity Indicator

**A Pine Script v6 indicator for session-based liquidity, market structure, and bias analysis on TradingView**

[![Pine Script](https://img.shields.io/badge/Pine%20Script-v6-131722?style=flat-square&logo=tradingview&logoColor=white)](https://www.tradingview.com/pine-script-docs/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/Arpan01574/-Multi-Session-Liquidity-Indicator?style=flat-square&color=orange)](https://github.com/Arpan01574/-Multi-Session-Liquidity-Indicator/stargazers)

[Overview](#overview) · [Features](#features) · [The Bias Engine](#the-bias-engine) · [Installation](#installation) · [Settings](#settings-reference) · [Disclaimer](#disclaimer)

</div>

---

## Overview

**Multi-Session Liquidity Indicator** — published on TradingView as *"Arpan's Trading Sessions"* — is an overlay indicator that maps how price behaves across the five major global trading sessions: **Sydney, Tokyo, Shanghai, London, and New York**.

On top of session range boxes and liquidity lines, it runs a rule-based **bias engine** that reads range, volatility, and volume-flow data to classify every completed session into a market phase — trend, expansion, accumulation, distribution, manipulation, or consolidation — and reports the result through a live on-chart dashboard.

It's built for traders working with session-liquidity / market-structure concepts (premium-discount, liquidity sweeps, session highs and lows) who want that read automated instead of marked up by hand.

## Preview

*Add a screenshot or short GIF of the indicator running on a live chart, e.g. `assets/preview.png`, and embed it here:*

![Indicator preview](assets/preview.png)

## Features

### Session Range Boxes
- Auto-drawn high/low boxes for **Sydney, Tokyo, Shanghai, London, and New York**, redrawn live as each session develops.
- Optional **merge mode** combines Sydney + Tokyo + Shanghai into a single "Asian" box.
- Per-box labels, adjustable background opacity, and solid / dashed / dotted borders.

### Session Liquidity Lines
- **Midlines** — a dashed equilibrium (premium/discount) line at the 50% level of each session's range, with an optional live price label.
- **High/low lines** — solid lines marking each session's extremes, with optional price labels.
- **Carry-forward** — extends the previous session's high/low lines until the next session begins, so liquidity levels stay visible between sessions.

### Macro Liquidity Levels
- **PDH / PDL** — Previous Day High / Low.
- **PWH / PWL** — Previous Week High / Low.
- Plotted as extended reference lines with live price labels.

### Bias Engine
A rule-based classifier ([details below](#the-bias-engine)) that scores every completed session on four dimensions:

| Dimension | What it captures |
|---|---|
| **Market Phase** | Trend, Expansion, Accumulation, Distribution, Manipulation, Consolidation, or Dead |
| **Liquidity** | Stop-hunt / liquidity-trap detection above or below the session range |
| **Volatility** | Candle range vs. `ATR(14)` — Dead, Normal, High, Extreme |
| **Range** | Session range vs. its own rolling average — Tight, Normal, Large, Extreme |

### Live Multi-Session Dashboard
- A table (top-right) summarizing **Asian / Europe / USA** sessions across all four Bias Engine dimensions, color-coded and updated every bar.
- Holds the last completed session's read-out on screen until the next session takes over.

Example output:

| Session | Market Phase | Liquidity | Volatility | Range |
|---|---|---|---|---|
| Asian | BULL TREND | NONE | HIGH | LARGE |
| Europe | EXPANSION ↑ | TRAP LOW | EXTREME | EXTREME |
| USA | CONSOLIDATION | NONE | NORMAL | NORMAL |

### Historical Bias Labels
- Prints the resolved market phase above every past session box, so you can scroll back through history and see how the engine read prior sessions.

### UI & Customization
- Full control over box opacity, borders, labels, and which sessions/lines are shown — 25+ inputs across five organized setting groups.

## The Bias Engine

Each completed session is scored by `get_ultra_bias()`, which combines several independent layers of logic:

**1. Smart-money flow.** A custom money-flow oscillator (`admf`) built from the close-to-close change over true range, weighted by `volume × hlc3`, smoothed with an `RMA(14)`, and normalized against a 20-period average volume. It's paired with a classic Accumulation/Distribution Line (9/15 SMA crossover) for momentum confirmation, and a 20-bar lookback that flags bullish/bearish divergence between price and flow.

**2. Volatility layer** — candle range compared against `ATR(14)`:

| Label | Condition |
|---|---|
| `DEAD` | range < 0.4 × ATR |
| `NORMAL` | default |
| `HIGH` | range > 1.2 × ATR |
| `EXTREME` | range > 2 × ATR |

**3. Range layer** — the completed session's total range compared against its own rolling average (`Range Average Length` input, default 12 sessions):

| Label | Condition |
|---|---|
| `TIGHT` | ratio < 0.5 |
| `NORMAL` | ratio < 1.0 |
| `LARGE` | ratio < 1.3 |
| `EXTREME` | ratio ≥ 1.3 |

**4. Liquidity events.** A `TRAP HIGH` / `TRAP LOW` flag fires when price wicks through a session's high or low and closes back inside it by more than half that candle's range — a stop-hunt / liquidity-grab signature.

These layers feed a priority-ordered decision tree that resolves to a single phase per session:

```
DEAD → MANIPULATION (↓/↑) → EXPANSION (↑/↓) → DISTRIBUTION → BULL/BEAR TREND → ACCUMULATION → CONSOLIDATION
```

- **Expansion** needs a directional breakout *and* aligned smart-money flow *and* ADL momentum *and* no opposing divergence.
- **Manipulation** needs a liquidity trap *and* flow already leaning against the trapped side.
- **Distribution** needs range exhaustion, plus either bearish divergence or negative flow.
- **Accumulation** needs a below-average range *and* positive flow *and* bullish ADL momentum with no breakout.
- Anything that clears none of the above resolves to **Consolidation**.

## Session Times

All sessions use Pine Script's timezone-aware `time()` call, so they auto-adjust for daylight saving in their local region — no manual DST handling needed.

| Session | Local Hours | Timezone |
|---|---|---|
| 🇦🇺 Sydney | 10:00 – 16:00 | `Australia/Sydney` |
| 🇯🇵 Tokyo | 09:00 – 15:00 | `Asia/Tokyo` |
| 🇨🇳 Shanghai | 09:30 – 15:00 | `Asia/Shanghai` |
| 🇬🇧 London | 08:00 – 16:30 | `Europe/London` |
| 🇺🇸 New York | 09:30 – 16:00 | `America/New_York` |

## Installation

1. Open any chart on [TradingView](https://www.tradingview.com/) (a free account is enough).
2. Open the **Pine Editor** panel at the bottom of the screen.
3. Clear the default template and paste in the full contents of this indicator's `.pine` file.
4. Click **Add to Chart**.
5. Open the indicator's **⚙ Settings** to toggle sessions, boxes, lines, and dashboard components to your liking.

## Settings Reference

<details>
<summary><strong>Session Boxes</strong></summary>
<br>

| Input | Default |
|---|---|
| Show Sydney Box | `true` |
| Show Tokyo Box | `true` |
| Show Shanghai Box | `true` |
| Merge Asian Box (Syd+Tok+Sha) | `false` |
| Show London Box | `true` |
| Show New York Box | `true` |

</details>

<details>
<summary><strong>Session Lines</strong></summary>
<br>

| Input | Default |
|---|---|
| Show Asian Session Midline | `true` |
| Show Europe Session Midline | `true` |
| Show USA Session Midline | `true` |
| Show Asian Session High/Low | `true` |
| Show Europe Session High/Low | `true` |
| Show USA Session High/Low | `true` |

</details>

<details>
<summary><strong>Macro Liquidity</strong></summary>
<br>

| Input | Default |
|---|---|
| Show Previous Daily High/Low (PDH/PDL) | `true` |
| Show Previous Weekly High/Low (PWH/PWL) | `true` |

</details>

<details>
<summary><strong>UI & Aesthetics</strong></summary>
<br>

| Input | Default |
|---|---|
| Background Opacity (0–100) | `97` |
| Show Box Borders | `true` |
| Border Style (`Solid` / `Dashed` / `Dotted`) | `Solid` |

</details>

<details>
<summary><strong>Advanced Features</strong></summary>
<br>

| Input | Default |
|---|---|
| Show Box Labels | `true` |
| Show Smart Range Dashboard | `true` |
| Show Historical Bias Labels | `true` |
| Range Average Length (Days) | `12` |
| Show Premium/Discount Midline | `true` |
| Show Midline Price Label | `true` |
| Show Session High/Low Lines | `true` |
| Show Price on H/L Lines | `true` |
| Carry Forward Previous H/L | `true` |

</details>

## Technical Notes

- Pine Script **v6**, single-file overlay indicator.
- The session engine (boxes, lines, dashboard) runs entirely on real-time bar data — no `request.security()` calls, so no higher-timeframe repainting risk there.
- The only `request.security()` calls are `PDH/PDL/PWH/PWL`, which pull already-closed `[1]` values with `lookahead_on` — the standard safe pattern for referencing a prior, fixed higher-timeframe bar.
- Drawing limits: `max_boxes_count = 5000`, `max_lines_count = 500`, `max_labels_count = 500`, `max_bars_back = 5000`.

## Disclaimer

This indicator is provided for **educational and informational purposes only** and does not constitute financial advice. It is a technical-analysis tool built on historical price, volume, and range data — it does not predict future price movement. Always backtest thoroughly and use sound risk management before trading with real capital.

## License

Licensed under the [MIT License](LICENSE).

## Author

Built by **Arpan** — [GitHub @Arpan01574](https://github.com/Arpan01574)

Suggestions and pull requests are welcome — feel free to open an issue.

---

<div align="center">

⭐ If this is useful to you, consider starring the repo!

</div>
