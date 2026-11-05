# Trade Whisperer smart-candle indicator: evidence report

Date: 2026-10-12  
Source: the 13 JPEG attachments in `<workspace>/Downloads/attachments (3)`  
Scope: static visual analysis plus public descriptions by Trade Whisperer. No private Pine source, compiled indicator, alerts, settings, or live-bar recording was supplied.

## Bottom line

The screenshots consistently show a proprietary **candle-colour state machine**, not a conventional buy/sell arrow indicator. The most defensible interpretation is:

| Colour | Documented meaning | Behavioral interpretation |
|---|---|---|
| Blue | Potential start of a bullish trend | Bullish transition candidate |
| Green | Bullish continuation | Bull regime remains active |
| Pink | Potential start of a bearish trend | Bearish transition candidate |
| Red | Bearish continuation | Bear regime remains active |
| Dark green | Cancellation/consolidation | Bearish transition failed or weak bullish/neutral state |
| Dark red | Cancellation/consolidation | Bullish transition failed or weak bearish/neutral state |
| Yellow-green | Optional trim / momentum stall | Extended bullish move is losing momentum |

The exact formula cannot be recovered from screenshots. The likely design is an adaptive multi-factor score feeding a hysteretic state machine. Momentum, volatility-normalized price movement, trend, Bollinger position, volume, higher-timeframe context, and possibly a broad-market regime are plausible inputs. Those inputs and all weights/thresholds remain inferred, not observed.

The companion file `smart_candles_reconstruction.pine` is therefore a **clean-room behavioral approximation**. It is not the author's code and is not expected to reproduce every historical colour.

## Attachment integrity and safety

- Gmail supplied exactly 13 attachments, all with `.jpg` names.
- The Downloads copies are byte-for-byte identical to the extracted Gmail bundle.
- The ZIP contains only those 13 JPEG entries; no executable, script, macro, shortcut, or nested archive was found.
- ZIP SHA-256: `e1b219be4ba25c43a60de8259b00c23040e3b2528158da6c5293c38f018885dd`
- Static inspection does not establish that every possible image decoder is vulnerability-free. Open the files with a patched image viewer/browser, as with any untrusted media.

## Attachment inventory

| File | Size | Visible chart | Main evidence |
|---|---:|---|---|
| `1000111321.jpg` | 1440×3120 | MU, 5 minute | Blue/pink transition candles, continuation colours, Bollinger envelope, moving averages, volume |
| `1000111323.jpg` | 1440×3120 | RKLB, daily | Pink near swing highs and blue near swing lows |
| `1000111325.jpg` | 1440×3120 | CAT, weekly | Alternating bearish and bullish transition states across large swings |
| `1000111327.jpg` | 1440×3120 | LULU, monthly | Pink near a major top followed by a long red continuation state |
| `1000111329.jpg` | 1440×3120 | BE, weekly | Blue near lows/pullbacks and pink near highs; dark consolidation colours |
| `1000111331.jpg` | 1440×3120 | SPCE/SPCX label, daily | Pink at the prior top, blue around the major bottom, then green continuation |
| `1000111333.jpg` | 1802×1055 | SIVE, weekly | Pink high, blue low, red continuation; also contains Fibonacci drawings |
| `1000111334.jpg` | 1440×3120 | MU, 5 minute | Repeated pink/blue transitions at intraday swings |
| `1000111336.jpg` | 1440×3120 | MU, 4 hour | Same state logic on a higher timeframe; background session/regime bands |
| `1000111338.jpg` | 1440×3120 | NOK, weekly | Pink highs and blue lows across a long sample |
| `1000111340.jpg` | 1440×3120 | META, 15 minute | Transition colours plus yellow-green trim/stall candles during consolidation |
| `1000111342.jpg` | 2048×1185 | DELL, weekly | Candle states, Bollinger Bands, moving averages, and visible-range volume profile |
| `1000111343.jpg` | 1802×1027 | AMD, weekly | Multiple blue transitions and green continuation with the same overlays |

The [full contact sheet](./contact-sheet.jpg) shows all attachments. The [chart-focused contact sheet](./chart-contact-sheet.jpg) removes most phone and social-media chrome.

## Direct observations

These conclusions come from repeated visible behavior in the screenshots:

1. A blue candle can mark a possible bullish regime change; subsequent green candles represent continuation. Pink and red form the bearish equivalent.
2. Transition colours cluster around local turns, but they do not identify exact tops or bottoms consistently.
3. Signals appear across intraday, daily, weekly, and monthly charts, suggesting timeframe-independent calculations or separate timeframe-specific calibrations.
4. The same chart can alternate blue and pink in choppy areas. Dark colours occur around weak or cancelled moves.
5. The blue/pink classification is not merely the candle's close-versus-open direction. Public version notes explicitly describe blue signals being allowed on red price bars.
6. The transition timing is not consistent with a simple slow-moving-average crossover. Several coloured transitions appear before the slower visible averages could cross.
7. Bollinger Bands, moving averages, volume bars, visible-range volume profile, Fibonacci levels, boxes/arrows, and background bands are separable overlays. They should not all be attributed to the proprietary candle-colour model.

## Public documentation corroboration

Trade Whisperer's public explainer, [How to read our proprietary smart algorithm candles](https://www.patreon.com/TradeWhisperer/posts/how-to-read-our-111495567), assigns the colour meanings listed above and describes these sequences:

- Blue followed by green confirms a bullish start.
- Pink followed by red confirms a bearish start.
- Blue followed by pink can collapse into dark red; pink followed by blue can collapse into dark green.
- Alternating blue and pink indicates indecision or an “exchange of hands.”
- Weekly/monthly momentum should outweigh daily/lower-timeframe signals.
- A middle Bollinger Band reclaim/loss adds confirmation.
- The yellow-green trim state means a sharp rise is stalling.

The public [New Daily Model V1.14](https://www.patreon.com/TradeWhisperer/posts/new-daily-model-145501749) release says the model was tested on many symbols and historical periods, but it does not disclose a formula. Later public release titles mention more blue signals on red days, colour-blind support, and a low-candle intraday mode. These are author claims and product documentation, not independent performance validation.

## Reconstructed architecture

The smallest design that explains the observed behavior is:

```text
OHLCV + optional market context
        │
        ├─ volatility-normalized momentum
        ├─ fast/slow trend and slope
        ├─ oscillator / acceleration
        ├─ Bollinger basis and band position
        ├─ directional volume anomaly
        └─ optional higher-timeframe or benchmark regime
                         │
               adaptive composite score
                         │
             entry / hold / cancel thresholds
                         │
     blue → green      dark states      pink → red
      bull start       indecision       bear start
                         │
                  yellow trim state
```

The important mechanism is **hysteresis**: entry into a regime requires stronger evidence than remaining in it. This reduces rapid flipping, while explicit cancellation colours preserve information about failed transitions.

### Prototype choices

The Pine reconstruction uses:

- ATR-normalized one-bar movement;
- fast/slow EMA separation;
- RSI displacement from neutral;
- normalized Bollinger position;
- volume anomaly signed by price direction;
- optional SPY or user-selected benchmark regime;
- a rolling score-deviation threshold so sensitivity adapts to the instrument/timeframe;
- a confirmed-bar-only default to avoid presenting an unfinished bar as final;
- a state machine for transition, continuation, cancellation, consolidation, and trim colours.

These are engineering choices consistent with the evidence, not recovered proprietary parameters.

## What remains unknown

- Exact indicators, lookback lengths, weights, transforms, thresholds, and any symbol/timeframe-specific calibration.
- Whether “data-learning feedback loops” means genuine online learning, periodic offline fitting, or marketing language for adaptive rolling statistics.
- How macro events, earnings, sector data, or broad-index state enter the calculation.
- Whether historical colours repaint before or after the candle close. Static screenshots cannot test this.
- Whether displayed examples include discretionary filtering, cherry-picking, or later model versions.
- Real precision, recall, drawdown, expectancy, slippage, or out-of-sample performance. The supplied images are not a backtest.

## How to validate the reconstruction

1. Add both the original indicator and the reconstruction to the same symbol, timeframe, session, and adjusted-price setting.
2. Record only **closed bars**, including every transition and cancellation, for at least 200–500 bars per timeframe.
3. Export a table with OHLCV, original colour, reconstructed colour, timestamp, and whether the colour changed after close.
4. Fit/calibrate on one set of symbols and dates; evaluate on untouched symbols and later dates.
5. Score transition timing separately from exact colour accuracy. A one-bar-late green regime may still be useful even if it misses the original blue candle.
6. Test around earnings, gaps, low-volume periods, and market regime changes. Include fees and slippage in any trading evaluation.

Until that comparison exists, treat the prototype as an explainable research tool—not a verified trading edge or financial advice.
