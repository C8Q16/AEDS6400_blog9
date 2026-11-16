# Blog 9 · Predicting Stock Returns with Gradient Boosting

This project revisits a classroom stock-picking experiment and tests a simple modification to the classroom XGBoost model. I wanted to answer two questions. Does a shallower model predict future returns more accurately? Do the stocks it ranks highest earn better returns in a backtest?

The project starts with the classroom game, then uses historical stock prices to train and evaluate two models. This README describes how to reproduce both parts and render the blog.

## Data sources

| Source | Contents and use |
|---|---|
| `Full_Results copy.xlsx` | The instructor-provided workbook contains two final portfolio values for each of 45 teams. `portfolio1` is the intuition-based portfolio and `portfolio2` is the indicator-based portfolio. The workbook is used to reproduce the classroom game results. |
| Yahoo Finance | The notebook uses `yfinance` to download daily opening, high, low, and closing prices, adjusted closing prices, and volume for the stocks listed in the classroom script. It also downloads SPY, an ETF that tracks the S&P 500. |
| `UPenn_XGBoostStockPrediction copy.R` | The instructor's R script provides the stock list, feature formulas, prediction target, and original model settings. It is a reference for the Python notebook. |

The notebook requests prices from **January 1, 2022**, through **November 13, 2026**. Its `END` value is `2026-11-14` because `yfinance` excludes the end date. The notebook prints any tickers for which it could not retrieve usable price data. If your results differ, check that list first. A fixed date range helps keep runs comparable, although Yahoo Finance may revise historical data or return a different set of tickers.

## How the analysis works

### 1. Reproduce the classroom game

Each team built two portfolios, one based on intuition and one based on stock indicators. I used a paired t-test because the two portfolio values belong to the same team. The test looks at the difference within each team, while the boxplot shows how final values were distributed across teams. This experiment provides the motivation for testing whether stock indicators can help a prediction model.

### 2. Build the prediction dataset

The Python notebook follows the classroom formulas for features such as momentum, recent volatility, short-term price changes, and daily returns relative to SPY. The model also uses price and volume columns included by the classroom script.

On a signal date \(t\), the model uses data available through that day's close. The prediction target is the return from the **close on trading day \(t+1\)** to the **close on trading day \(t+20\)**, calculated using adjusted closing prices:

```text
target = adjusted_close[t + 20] / adjusted_close[t + 1] - 1
```

The test period starts in January 2024. A row goes into the training set only if its forward return was fully known before then. For example, a signal from late December 2023 is excluded from training if the closing price needed to calculate its target falls in January 2024.

### 3. Compare the forecasts

I kept the classroom XGBoost settings and changed one hyperparameter. The original model has a maximum tree depth of **5**, and the modified model has a maximum depth of **2**. Deeper trees can learn more specific combinations of features, which may fit patterns that happened by chance in the training period. Reducing depth tests whether simpler rules work better on later data. Using the same features, target, and evaluation sample makes the two models easier to compare.

The third forecast is the average forward return in the training set. It assigns that same value to every stock in the test set. The notebook evaluates all three approaches on the same observations using **MAE** and **RMSE**, reported in percentage points.

### 4. Backtest the stock-selection rule

Every 20 trading sessions, each model ranks eligible stocks by predicted return. The portfolio holds the top 10 stocks with equal weights. The two model portfolios, the equal-weight portfolio of eligible stocks, and SPY all use the same entry and exit dates. The notebook compounds the holding-period returns to show the growth of an initial dollar.

The Python backtest uses equal weights and fixed trading-session intervals, so it can produce different results from the instructor's R backtest. It assumes zero transaction costs and zero returns between holding windows. The stock universe is the fixed list from the classroom script, and the notebook evaluates stocks with available forward prices. The results describe this sample and do not establish how the strategy would perform after costs or with a point-in-time stock universe.

## How to reproduce the results

1. Open `blog9.ipynb` in Google Colab.
2. Upload `Full_Results copy.xlsx` through the **Files** panel. The classroom-game cell expects it at `/content/Full_Results copy.xlsx`. If you are running locally, change that path to the workbook's local path.
3. Run **Cell 1** to install `yfinance`, `xgboost`, and `openpyxl`. The notebook also uses `pandas`, `numpy`, `matplotlib`, `scipy`, and `scikit-learn`, which are available in the standard Colab environment.
4. Run **Cell 0** to summarize the classroom game and save its boxplot.
5. Run **Cells 2 through 4** in order. Cell 2 downloads prices and constructs the dataset. Cell 3 trains the models and displays prediction errors. Cell 4 runs the backtest and saves the forecast metrics and strategy returns.
6. Download the generated files from Colab.

| Output | What it contains |
|---|---|
| `classroom_game.png` | Boxplot of final portfolio values in the classroom game |
| `prediction_metrics.csv` | Sample-out forecast errors for both XGBoost models and the historical-mean benchmark |
| `backtest_returns.csv` | Holding-period returns for both model portfolios, the equal-weight stock universe, and SPY |
| `figure1.png` | Cumulative portfolio performance over the backtest |

## Render the blog

Place `blog9.qmd`, `blog9.ipynb`, `classroom_game.png`, `figure1.png`, and `prediction_metrics.csv` in the same directory. With Quarto installed, run this command from that directory:

```bash
quarto render blog9.qmd
```

Open `blog9.html` to check the figures and download links. When publishing, include both PNG files, the notebook, and `prediction_metrics.csv` at the same relative paths. The QMD uses the saved results, so rendering the webpage does not rerun the analysis.

## What to check in the results

In the original run, reducing the tree depth lowered MAE from **6.849** to **6.645** percentage points. The historical mean still had the lowest MAE at **6.426**. The shallower model's top-10 portfolio had the highest cumulative return, **77.86%** over 36 holding periods before trading costs. These metrics answer different questions. Forecast error measures performance across all evaluated stocks, while the portfolio result depends on the stocks selected at the top of the ranking.