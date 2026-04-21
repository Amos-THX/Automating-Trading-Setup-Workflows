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


# Select Tickers to Focus Based on Setup Rules


# Plot Candlestick Graphs for Tickers in Focus
