# AirPassengers forecasting report
Experimentation & Causal Inference — [SDAIA Academy](https://github.com/SDAIAAcademy/)

Name: Anas Ibrahim Almutairi
## Recommendation

I recommend **AutoETS** as the model to ship for the current monthly passenger forecast. In the eight-origin, 12-month rolling-origin evaluation, it achieved a mean **MASE of 0.952**, compared with **1.313** for the seasonal-naive benchmark. This is an improvement of about **27.5%** in scaled absolute error. It also improved RMSSE (1.003 versus 1.246) and scaled CRPS (0.0499 versus 0.0616).

This recommendation is based on the rolling-origin harness rather than a single holdout. The framework leaderboard's one 12-month validation split ranked AutoARIMA first, but the wider harness result available for the selected seasonal ETS contender is the more useful evidence for the operational choice.

## Data and initial read

The data contain 144 monthly observations from January 1949 through December 1960, with no missing months. Passenger numbers rise strongly over the full period and repeat a clear annual pattern. The seasonal swings become larger as the overall level grows, so uncertainty and variance should be watched carefully when forecasting further ahead.

The additive STL decomposition confirms this reading. Trend strength was **1.00** and seasonal strength was **0.99**. Both components are therefore essential: a useful model must represent a growing level and annual seasonality. The raw-series ACF is also high at lags 12 and 24, which is further evidence of a yearly seasonal structure.

## Benchmark and residual diagnosis

The seasonal-naive benchmark repeats the value from the same month in the previous year. It is a meaningful baseline because it captures the annual calendar pattern, but it does not directly model the upward growth in passenger counts.

Its residuals are not white noise. The Ljung-Box results were:

| Lag | Ljung-Box statistic | p-value |
|---:|---:|---:|
| 12 | 234.494 | 2.32e-43 |
| 24 | 275.036 | 1.71e-44 |

Both p-values are far below conventional significance levels, so the remaining serial structure is not random. In practical terms, the benchmark leaves trend-related structure that an ETS-style model can model.

## Rolling-origin evaluation

Both models were evaluated over eight rolling origins with a 12-month forecast horizon. Lower MASE, RMSSE, and CRPS are better; 80% coverage should ideally be close to 0.80.

| Model | MASE | RMSSE | Scaled CRPS | 80% coverage | Minimum fold MASE | Maximum fold MASE |
|---|---:|---:|---:|---:|---:|---:|
| SeasonalNaive | 1.313310 | 1.245572 | 0.061603 | 0.510417 | 0.411584 | 1.958726 |
| AutoETS | 0.951994 | 1.002880 | 0.049858 | 0.489583 | 0.564803 | 1.554323 |

AutoETS wins on all three error/distribution metrics in this comparison. Its improvement is not identical in every fold, which is visible in its MASE range, but its worst fold is still better than the benchmark's worst fold.

## Intervals and next step

The point forecasts improve with AutoETS, but neither model's nominal 80% interval is calibrated well: observed coverage was about **51.0%** for SeasonalNaive and **49.0%** for AutoETS. These bands are too narrow or otherwise insufficiently uncertain, so they should not be described as fully reliable 80% decision intervals.

The next modelling change should be to fit and validate a **log-transformed or multiplicative seasonal model**. The chart shows that the amplitude of seasonal variation grows with the series level. A transformation intended to stabilize that variance may produce better-calibrated forecast intervals; the same rolling-origin evaluation and coverage check should be repeated before adoption.
