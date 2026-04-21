# Pulling Data From Yahoo Finance
This script retrieves historical market data for a list of tickers using yfinance, pulling up to 24 months of daily price data in a single, multi-threaded request. The data is structured with a MultiIndex format—organizing each ticker alongside its corresponding metrics (e.g., Open, High, Low, Close, Volume)—which enables efficient large-scale analysis across multiple tickers in a unified dataset.

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

# Select Tickers to Focus Based on Setup Rules


# Plot Candlestick Graphs for Tickers in Focus
