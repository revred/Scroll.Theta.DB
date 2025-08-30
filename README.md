# Stroll.Theta Database Repository

This repository contains validated, quality-assured financial market data for the Stroll.Theta options backtesting system.

## Data Quality Standards

All data in this repository has passed comprehensive validation including:
- **Join ratio** ≥85% (underlying-options alignment)
- **Zero-option days** ≤2% miss rate
- **Timestamp normalization** (ET→UTC conversion)
- **Sanity checks** (IV bounds, bid/ask spreads, etc.)
- **Integrity verification** (no duplicate PKs, monotonic timestamps)

## Repository Structure

- **days/**: Daily-partitioned data with comprehensive validation logs
  - `YYYY/MM/DD/SYMBOL.YYYY-MM-DD.sqlite` - Database files  
  - `YYYY/MM/DD/logs/TIMESTAMP/` - Quality probe results and validation reports
  - `YYYY/MM/DD/manifest.json` - Validation status, checksums, and metadata
- **weeks/**: Weekly rollup metadata and summaries  
- **schema/**: Database schema definitions and migrations

## Data Coverage

**Symbols**: SPX, SPY, XSP, GLD, USO, VIX
**Period**: 2018-2025 (7-year backfill)
**Resolution**: 1-minute bars with options greeks
**Quality**: Enterprise-grade validation with comprehensive probe suite

Each day's data includes a `manifest.json` with validation results, checksums, and quality status before being committed to this repository.

## Dataset Access Guide

### 1. Repository Access

**Clone the repository:**
```bash
git clone https://github.com/revred/Scroll.Theta.DB.git
cd Scroll.Theta.DB
```

**Check data availability:**
```bash
# List available years
ls days/

# List available months for a specific year  
ls days/2023/

# List specific days
ls days/2023/06/

# Check data for a specific date
ls days/2023/06/05/
```

### 2. Database File Structure

**File naming convention:**
```
days/YYYY/MM/DD/SYMBOL.YYYY-MM-DD.sqlite
```

**Example paths:**
```
days/2023/06/05/SPX.2023-06-05.sqlite    # S&P 500 Index options
days/2023/06/05/GLD.2023-06-05.sqlite    # SPDR Gold Trust ETF
days/2023/06/05/manifest.json            # Validation metadata
days/2023/06/05/logs/                     # Quality validation reports
```

### 3. Database Schema

**Each SQLite database contains:**

- **`underlying_minute`**: Minute-level OHLCV data
  ```sql
  SELECT ts, open, high, low, close, volume 
  FROM underlying_minute 
  WHERE symbol = 'SPX' 
  LIMIT 10;
  ```

- **`option_minute`**: Options greeks, quotes, and analytics  
  ```sql
  SELECT ts, strike, expiry, right, bid, ask, delta, gamma, iv
  FROM option_minute 
  WHERE u_symbol = 'SPX' AND expiry > date('now')
  LIMIT 10;
  ```

- **`chain_daily`**: Daily options chain summaries
  ```sql
  SELECT expiry, total_contracts, avg_iv, max_oi
  FROM chain_daily 
  WHERE symbol = 'SPX'
  ORDER BY expiry;
  ```

### 4. Quality Validation Logs

**Log structure:**
```
days/YYYY/MM/DD/logs/TIMESTAMP/
├── inventory.csv              # File counts and coverage
├── join_ratio.tsv            # Underlying-options alignment
├── minute_density.tsv        # Time series completeness  
├── zero_option_days.tsv      # Days with missing options data
├── sanity_violations.tsv     # Data quality violations
├── integrity_violations.txt  # Database integrity status
└── pattern_analysis.txt      # ML-based anomaly detection
```

### 5. Manifest Files

**Each day includes a `manifest.json`:**
```json
{
  "date": "2023-06-05",
  "symbols": ["SPX", "GLD"],
  "status": "GREEN",
  "qualityChecks": {
    "joinRatio": "PASSED",
    "zeroOptionDays": "PASSED", 
    "timestampNormalization": "PASSED",
    "sanityChecks": "PASSED",
    "integrityChecks": "PASSED"
  },
  "files": [
    {
      "name": "SPX.2023-06-05.sqlite",
      "size": 53334016,
      "sha256": "459c8d3f71c21acd2cff63f9981618c1085880d02d52690681d48ad9e5961c60"
    }
  ]
}
```

### 6. Data Quality Standards

**GREEN Status Requirements:**
- Join ratio ≥85% (underlying-options timestamp alignment)
- Zero-option days ≤2% of trading days  
- No critical sanity violations (IV sentinels, bid>ask, etc.)
- Complete timestamp normalization (ET→UTC)
- Integrity checks passed (no duplicate keys, valid ranges)

**Accessing Quality Metrics:**
```bash
# Check overall status
cat days/2023/06/05/manifest.json | jq '.status'

# Review quality violations  
cat days/2023/06/05/logs/*/sanity_violations.txt

# Analyze join ratio performance
cat days/2023/06/05/logs/*/join_ratio.txt
```

### 7. Command-Line Data Access

**SQLite command-line examples:**
```bash
# Connect to a database
sqlite3 days/2023/06/05/SPX.2023-06-05.sqlite

# Query recent options data
sqlite3 days/2023/06/05/SPX.2023-06-05.sqlite \
  "SELECT COUNT(*) FROM option_minute WHERE right='C'"

# Export to CSV
sqlite3 -header -csv days/2023/06/05/SPX.2023-06-05.sqlite \
  "SELECT * FROM underlying_minute LIMIT 100" > spx_sample.csv
```

### 8. Python Integration

**Basic data access:**
```python
import sqlite3
import pandas as pd

# Connect to database
conn = sqlite3.connect('days/2023/06/05/SPX.2023-06-05.sqlite')

# Load underlying data
underlying = pd.read_sql(
    "SELECT * FROM underlying_minute WHERE symbol='SPX'", 
    conn, 
    parse_dates=['ts']
)

# Load options data  
options = pd.read_sql(
    """SELECT ts, strike, expiry, right, bid, ask, delta, iv 
       FROM option_minute 
       WHERE u_symbol='SPX' AND expiry='2023-06-16'""", 
    conn,
    parse_dates=['ts', 'expiry']
)

conn.close()
```

### 9. Data Integrity Verification

**Verify data integrity:**
```bash
# Check file checksums
cd days/2023/06/05/
sha256sum *.sqlite

# Compare with manifest
cat manifest.json | jq -r '.files[].sha256'

# Validate database structure
sqlite3 SPX.2023-06-05.sqlite ".schema"
```

### 10. Troubleshooting

**Common issues and solutions:**

- **Missing data**: Check `manifest.json` status and probe logs for gaps
- **Quality issues**: Review `sanity_violations.tsv` and `join_ratio.txt`  
- **Timestamp problems**: Verify `span_validation.txt` for range issues
- **Schema questions**: Reference main project documentation at [Stroll.Theta](https://github.com/user/Stroll.Theta)