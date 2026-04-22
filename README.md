# Pulling Data From Yahoo Finance
This script retrieves historical market data for a list of tickers using yfinance, pulling up to 24 months of daily price data in a single, multi-threaded request. The data is structured with a MultiIndex format—organizing each ticker alongside its corresponding metrics (e.g., Open, High, Low, Close, Volume)—which enables efficient large-scale analysis across multiple tickers in a unified dataset.

We also defined the list of tickers (*tickers*) in the S&P 500 Index and other tickers we will like to track, before passing this variable into the Yahoo Finance download function.

```
# Download stock data for multiple tickers
# The data will have a MultiIndex for columns (Ticker, Metric)
data = yf.download(
    tickers,
    period="25mo",
    interval="1d",
    group_by="ticker",
    auto_adjust=False,
    threads=True
)
```

# Calculate Technical Indicators Across Various Time Frames
This script computes key technical indicators and performance metrics from historical price data, including total returns, RSI, volatility, rolling Sharpe ratio, and MACD. It transforms raw price series into actionable signals, enabling systematic evaluation of trading setups across multiple assets.

```
def calculate_total_pnl(close_prices, period_days):
    """
    Calculates the total percentage profit/loss over a given period as specified by user.
    """
    # Calculate the price at the start of the period
    price_start_of_period = close_prices.shift(period_days)
    # Calculate total PnL%
    total_pnl_percent = ((close_prices / price_start_of_period) - 1) * 100
    return total_pnl_percent


def calculate_rsi(close_prices, period):
    """
    Calculates the RSI (Relative Strength Index) over a given period as specified by user.
    """

    delta = close_prices.diff()
    gain = delta.clip(lower=0)
    loss = -delta.clip(upper=0)

    avg_gain = gain.rolling(window=period, min_periods=period).mean()
    avg_loss = loss.rolling(window=period, min_periods=period).mean()
    rs = avg_gain / avg_loss
    rsi = 100 - (100 / (1 + rs))
    return rsi

def calculate_volatility(close_prices, period, annualization_factor=252):
    daily_returns = close_prices.pct_change()
    volatility = daily_returns.rolling(window=period).std() * (annualization_factor**0.5)
    return volatility

def rolling_sharpe_ratio(returns_series, window_days, annualization_factor_sqrt=252**0.5):
    """
    Calculates the annualized rolling Sharpe Ratio for a given period as specified by user.
    Assumes a risk-free rate of 0.
    """
    # Calculate rolling mean and standard deviation of daily returns
    rolling_mean_returns = returns_series.rolling(window=window_days).mean()
    rolling_std_returns = returns_series.rolling(window=window_days).std()

    # Avoid division by zero for cases where standard deviation is zero
    # Fill NaN from std=0 with 0 Sharpe, as there's no risk or return variability
    sharpe = (rolling_mean_returns / rolling_std_returns).fillna(0)
    return sharpe * annualization_factor_sqrt # Annualize the Sharpe Ratio

def calculate_macd(close_prices, fast_period=12, slow_period=26, signal_period=9):
    """
    Calculates MACD (Moving Average Convergence Divergence) indicators.
    """
    exp1 = close_prices.ewm(span=fast_period, adjust=False).mean()
    exp2 = close_prices.ewm(span=slow_period, adjust=False).mean()
    macd_line = exp1 - exp2
    signal_line = macd_line.ewm(span=signal_period, adjust=False).mean()
    macd_histogram = macd_line - signal_line
    return macd_line, signal_line, macd_histogram
```

# Filter Tickers to Focus Based on Setup Rules
Say for example, before the start of each trading day, out of the 500 tickers in the S&P 500 Index, we will like to take a closer look only at tickers that have their candle trading below the lower Bollinger Band. This snippet below identifies stocks that are currently trading below their lower Bollinger Band based on the most recent trading day.
It first extracts the latest date in the dataset, then filters the dataframe to include only rows from that date where the Candle_Below_Lower_BB flag is set to 1 (indicating the price has fallen below the lower band). 

```
# Find the latest date in the price_df
latest_date = price_df['Date'].max()

# Filter for the latest date and where Price_Below_Lower_BB is 1
companies_below_lower_bb = price_df[
    (price_df['Date'] == latest_date) &
    (price_df['Candle_Below_Lower_BB'] == 1)
]

# Get the list of unique ticker symbols
ticker_list = companies_below_lower_bb['Ticker_Symbol'].unique().tolist()

# Print the list of companies
if ticker_list:
    print(f"Companies trading below the Lower Bollinger Band as of {latest_date.strftime('%Y-%m-%d')}:")
    for ticker in ticker_list:
        print(ticker)
else:
    print(f"No companies found trading below the Lower Bollinger Band as of {latest_date.strftime('%Y-%m-%d')}.")

```


# Plot Candlestick Graphs for Tickers in Focus
This script uses the mplfinance package to generate technical analysis charts for selected stocks in a clean and systematic way in a large scale. In this example, we will like to take a look at the candlestick charts, Bollinger Bands and layer it with the MACD indicators.

The value of this approach is scalability and consistency: instead of manually charting each stock, this pipeline automatically generates standardized, information-rich visualizations across multiple tickers. This makes it easy to quickly identify trading signals across hundreds/thousands of tickers (e.g., price deviations, momentum shifts) and integrate chart generation into a larger data or LLM-driven workflow. 

![alt text](https://github.com/Amos-THX/Automating-Trading-Setup-Workflows/blob/main/NKE_candlestick.png?raw=true)
![alt text](https://github.com/Amos-THX/Automating-Trading-Setup-Workflows/blob/main/PODD_candlestick.png?raw=true)
