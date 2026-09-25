# Setting Up Real Market Data

This guide walks you through replacing simulated data with real historical market data from Yahoo Finance and FRED (Federal Reserve Economic Data).

## Prerequisites

1. Python 3.8+
2. Virtual environment (recommended)
3. pip

## Step 1: Install Dependencies

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

## Step 2: Get a FRED API Key (Optional, for unemployment data)

1. Go to [FRED API](https://fred.stlouisfed.org/docs/api/fred/)
2. Sign up for a free API key
3. Save it in a `.env` file:
   ```
   FRED_API_KEY=your_api_key_here
   ```

## Step 3: Create Data Fetching Script

Create `data/fetch_real_data.py`:

```python
import yfinance as yf
import pandas as pd
import json
from datetime import datetime
import os
from dotenv import load_dotenv

load_dotenv()

# Configuration
SECTORS = {
    'XLV': 'Health Care',
    'XLE': 'Energy',
    'XLK': 'Information Technology',
    'XLF': 'Financials',
    'XLY': 'Consumer Discretionary',
    'XLRE': 'Real Estate',
    'XLP': 'Consumer Staples',
}
START_DATE = '2020-01-01'
END_DATE = '2024-12-31'

def fetch_sector_data():
    """Fetch historical sector ETF data from Yahoo Finance"""
    print("Fetching sector data...")
    
    all_data = {}
    for ticker, name in SECTORS.items():
        print(f"  Downloading {ticker}...")
        data = yf.download(ticker, start=START_DATE, end=END_DATE, progress=False)
        all_data[ticker] = data['Close'].to_dict()
    
    return all_data

def fetch_vix_data():
    """Fetch VIX (market volatility) data"""
    print("Fetching VIX data...")
    vix = yf.download('^VIX', start=START_DATE, end=END_DATE, progress=False)
    return vix['Close'].to_dict()

def fetch_sp500_data():
    """Fetch S&P 500 (SPY) data"""
    print("Fetching S&P 500 data...")
    spy = yf.download('SPY', start=START_DATE, end=END_DATE, progress=False)
    return spy['Close'].to_dict()

def fetch_unemployment_data():
    """Fetch unemployment rate from FRED"""
    try:
        from fredapi import Fred
        api_key = os.getenv('FRED_API_KEY')
        fred = Fred(api_key=api_key)
        
        print("Fetching unemployment data...")
        unrate = fred.get_series('UNRATE', start_date=START_DATE, end_date=END_DATE)
        
        # Convert to daily interpolation (FRED data is monthly)
        unrate_daily = unrate.resample('D').ffill()
        return unrate_daily.to_dict()
    except Exception as e:
        print(f"Warning: Could not fetch unemployment data: {e}")
        return {}

def format_for_backtester(sectors, vix, sp500, unemployment):
    """Convert data to backtester format"""
    print("Formatting data for backtester...")
    
    data = []
    dates = pd.date_range(start=START_DATE, end=END_DATE, freq='B')  # Business days only
    
    for date in dates:
        date_str = date.strftime('%Y-%m-%d')
        
        # Skip if data not available for this date
        if date_str not in sp500:
            continue
        
        record = {
            'date': date_str,
            'spy': float(sp500[date_str]),
            'vix': float(vix.get(date_str, 15.0)),  # Default to 15 if missing
            'unemployment': float(unemployment.get(date_str, 4.0)),  # Default to 4.0 if missing
            'sectors': {}
        }
        
        for ticker in SECTORS:
            if date_str in sectors[ticker]:
                record['sectors'][ticker] = float(sectors[ticker][date_str])
        
        data.append(record)
    
    return data

def save_data(data, filename='data/real_market_data.json'):
    """Save data to JSON file"""
    print(f"Saving data to {filename}...")
    os.makedirs(os.path.dirname(filename), exist_ok=True)
    
    with open(filename, 'w') as f:
        json.dump(data, f, indent=2)
    
    print(f"✓ Successfully saved {len(data)} trading days to {filename}")

def main():
    print("=" * 60)
    print("Real Market Data Fetcher")
    print("=" * 60)
    
    sectors = fetch_sector_data()
    vix = fetch_vix_data()
    sp500 = fetch_sp500_data()
    unemployment = fetch_unemployment_data()
    
    data = format_for_backtester(sectors, vix, sp500, unemployment)
    save_data(data)
    
    print("\n✓ Done! You can now use this data in the backtester.")
    print(f"  Data covers {len(data)} trading days from {START_DATE} to {END_DATE}")

if __name__ == '__main__':
    main()
```

## Step 4: Run the Fetcher

```bash
python data/fetch_real_data.py
```

This creates `data/real_market_data.json` with all your historical market data.

## Step 5: Update Backtester to Use Real Data

In `index.html`, replace the `generateHistoricalData()` function call with:

```javascript
// Load real data
fetch('data/real_market_data.json')
  .then(res => res.json())
  .then(data => {
    const historicalData = data.map(d => ({
      date: new Date(d.date),
      spy: d.spy,
      vix: d.vix,
      unemployment: d.unemployment,
      sectors: d.sectors
    }));
    
    const backtest = calculateReturns(historicalData, rotationRule);
    const calcs = calculateMetrics(backtest);
    setResults(backtest);
    setMetrics(calcs);
    renderCharts(backtest);
  });
```

## Step 6: Cache Data (Optional)

To avoid re-fetching every time, add this to your data fetching script:

```python
import time

CACHE_FILE = 'data/real_market_data.json'
CACHE_EXPIRY = 86400  # 24 hours in seconds

def load_or_fetch():
    if os.path.exists(CACHE_FILE):
        mtime = os.path.getmtime(CACHE_FILE)
        if time.time() - mtime < CACHE_EXPIRY:
            print("Loading from cache...")
            with open(CACHE_FILE, 'r') as f:
                return json.load(f)
    
    # Fetch new data if cache doesn't exist or expired
    sectors = fetch_sector_data()
    # ... rest of fetching logic
```

## Troubleshooting

### Issue: "No module named 'yfinance'"
**Solution**: Make sure you've activated your virtual environment and installed dependencies:
```bash
source venv/bin/activate
pip install -r requirements.txt
```

### Issue: FRED API key not working
**Solution**: 
1. Check that your `.env` file is in the root directory
2. Verify the API key is correct
3. If issues persist, the unemployment data will default to 4.0

### Issue: Data gaps or missing values
**Solution**: The script uses forward-fill (`ffill()`) to interpolate monthly unemployment data to daily. For sector data, any missing trading days are skipped (markets don't trade on weekends/holidays).

## Data Structure Reference

The JSON file produced has this structure:

```json
[
  {
    "date": "2020-01-02",
    "spy": 321.85,
    "vix": 12.45,
    "unemployment": 3.5,
    "sectors": {
      "XLV": 119.45,
      "XLE": 68.23,
      "XLK": 124.50,
      "XLF": 85.23,
      "XLY": 126.40,
      "XLRE": 82.10,
      "XLP": 61.20
    }
  },
  ...
]
```

## Next Steps

1. Run the fetcher once a month to keep data current
2. Add walk-forward validation using data windows
3. Test your rotation strategy against 2025+ data (forward performance)
4. Compare with the simulated data version to ensure compatibility

## Resources

- [yfinance Documentation](https://github.com/ranaroussi/yfinance)
- [FRED API Guide](https://fred.stlouisfed.org/docs/api/)
- [Yahoo Finance Data Quality](https://finance.yahoo.com/)
