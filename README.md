# Stroll.Theta Database Repository

This repository contains the complete historical financial market database for the Stroll.Theta options backtesting system. All data is systematically collected from ThetaData API with comprehensive quality validation.

## Repository Overview

**Repository**: https://github.com/revred/Stroll.Theta.DB  
**Data Coverage**: January 2018 - August 2025 (systematic backfill in progress)  
**Symbols**: SPX, VIX, GLD, SPY, USO, XSP (indices and options)  
**Update Frequency**: Monthly backfill with daily commits  

## Directory Structure

```
C:/code/Stroll.Theta.DB/
├── days/                    # Daily-partitioned SQLite databases
│   ├── 2023/06/05/         # YYYY/MM/DD structure
│   │   ├── SPX.2023-06-05.sqlite  # SPX index & options data
│   │   ├── VIX.2023-06-05.sqlite  # VIX index data
│   │   ├── GLD.2023-06-05.sqlite  # GLD ETF & options data
│   │   ├── SPY.2023-06-05.sqlite  # SPY ETF & options data
│   │   ├── USO.2023-06-05.sqlite  # USO ETF & options data
│   │   └── XSP.2023-06-05.sqlite  # XSP options data
│   └── [More years/months/days...]
├── logs/                    # Quality validation probe logs
│   ├── 2025-08-30-11-54-24/
│   │   ├── probe-summary.json
│   │   ├── data-integrity.log
│   │   ├── risk-analysis.log
│   │   └── pattern-detection.log
│   └── [More timestamped runs...]
├── manifests/              # Daily validation manifests (SHA256, status)
├── probes/                 # Quality validation configurations
├── schema/                 # Database schema definitions
└── weeks/                  # Weekly summary metadata
```

## Data Quality Standards

Every database file has passed comprehensive quality validation:

### Core Validation Criteria
- ✅ **Data Integrity**: No duplicate primary keys, monotonic timestamps
- ✅ **Completeness**: Full trading day coverage (9:30 AM - 4:00 PM ET)  
- ✅ **Accuracy**: Bid/ask spread validation, IV bounds checking
- ✅ **Consistency**: Underlying-options alignment verification
- ✅ **Timeliness**: Real-time ET→UTC timestamp normalization

### Quality Metrics
- **Join Ratio**: ≥85% underlying-options alignment
- **Zero-Option Days**: ≤2% miss rate for options data
- **Health Score**: Tracked for each validation run
- **Status**: GREEN/RED validation status per day

## Database Schema

Each daily SQLite file contains two primary tables:

### underlying_minute
Minute-level OHLCV data for indices and ETFs:
```sql
CREATE TABLE underlying_minute (
    id INTEGER PRIMARY KEY,
    symbol TEXT NOT NULL,
    ts BIGINT NOT NULL,      -- Unix timestamp (milliseconds UTC)
    open REAL,
    high REAL, 
    low REAL,
    close REAL,
    volume INTEGER
);
```

### option_minute  
Minute-level options data with Greeks:
```sql
CREATE TABLE option_minute (
    id INTEGER PRIMARY KEY,
    u_symbol TEXT NOT NULL,   -- Underlying symbol
    symbol TEXT NOT NULL,     -- Options symbol
    expiry TEXT NOT NULL,
    strike REAL NOT NULL,
    right TEXT NOT NULL,      -- 'C' or 'P'
    ts BIGINT NOT NULL,       -- Unix timestamp (milliseconds UTC)
    bid REAL,
    ask REAL,
    delta REAL,
    gamma REAL,
    theta REAL,
    vega REAL,
    iv REAL                   -- Implied volatility
);
```

## Data Access Examples

### Python/SQLite Access
```python
import sqlite3
from datetime import datetime

# Connect to a daily database
conn = sqlite3.connect('days/2023/06/05/SPX.2023-06-05.sqlite')

# Get SPX minute bars for the day
df = pd.read_sql_query("""
    SELECT ts, open, high, low, close, volume 
    FROM underlying_minute 
    WHERE symbol = 'SPX'
    ORDER BY ts
""", conn)

# Convert timestamps to readable format
df['datetime'] = pd.to_datetime(df['ts'], unit='ms', utc=True)
```

### Direct SQL Query
```bash
# Query from command line
sqlite3 days/2023/06/05/SPX.2023-06-05.sqlite \
  "SELECT COUNT(*) as minute_bars FROM underlying_minute WHERE symbol='SPX';"
```

### Options Chain Analysis
```sql
-- Get end-of-day options chain
SELECT expiry, strike, right, bid, ask, delta, iv
FROM option_minute 
WHERE u_symbol = 'SPX' 
  AND ts = (SELECT MAX(ts) FROM option_minute WHERE u_symbol = 'SPX')
ORDER BY expiry, strike, right;
```

## Systematic Data Collection

The repository is populated through a systematic monthly backfill process:

### Collection Status
- ✅ **August 2025**: Complete (indices only - future month)
- ✅ **July 2025**: Complete (indices only - future month)  
- ✅ **June 2025**: Complete (indices only - future month)
- ✅ **March 2022**: Complete (all symbols)
- ✅ **Golden Week 2023**: Complete (June 5-9, 2023 - all symbols)
- 🔄 **In Progress**: Systematic backfill Aug 2025 → Jan 2018

### Data Sources
- **Primary**: ThetaData API (subscription-based)
- **License**: INDEX.PRO + OPTION.STANDARD
- **Coverage**: Full minute-level historical data
- **Update**: Monthly batch processing with quality validation

## Quality Validation Process

Each data collection run includes:

1. **Data Acquisition**: ThetaData API fetch with retry logic
2. **Quality Probes**: Comprehensive validation suite
   - Data integrity checks
   - Pattern detection analysis  
   - Risk analysis validation
   - Completeness verification
3. **Logging**: Timestamped logs with health scores
4. **Git Commit**: Automatic commit to repository with status

### Probe Logs
Access detailed validation results in `logs/` directory:
- **probe-summary.json**: Overall health score and status
- **data-integrity.log**: Detailed validation results
- **risk-analysis.log**: Trading risk assessment
- **pattern-detection.log**: Market pattern analysis

## File Sizes and Storage

**Current Storage**: 73 SQLite files  
**Estimated Final Size**: ~100GB+ with full 7-year coverage  

### Typical Daily File Sizes
- **SPX**: ~5.3MB/day (index + options)
- **VIX**: ~5.0MB/day (index only)
- **SPY**: ~8.2MB/day (ETF + options)
- **GLD**: ~4.1MB/day (ETF + options) 
- **USO**: ~3.8MB/day (ETF + options)
- **XSP**: ~6.7MB/day (options only)

## Troubleshooting

### Common Issues
```bash
# Check recent data collection
ls -la days/*/\**/\**/\*.sqlite | tail -10

# Verify probe logs
ls -la logs/ | head -5

# Check database integrity
sqlite3 days/2023/06/05/SPX.2023-06-05.sqlite "PRAGMA integrity_check;"

# Query validation status
find logs/ -name "probe-summary.json" | xargs grep -l "GREEN"
```

### Data Validation
```sql
-- Verify data completeness for a trading day
SELECT 
    COUNT(*) as total_minutes,
    MIN(datetime(ts/1000, 'unixepoch')) as first_bar,
    MAX(datetime(ts/1000, 'unixepoch')) as last_bar
FROM underlying_minute 
WHERE symbol = 'SPX';
```

## Repository Maintenance

This repository is automatically maintained through:
- **Monthly Data Collection**: Systematic backfill process
- **Quality Validation**: Every data commit includes probe validation
- **Git Integration**: Automatic commits with descriptive messages
- **Health Monitoring**: Continuous quality score tracking

## Repository Split and Organization

The Stroll.Theta project is organized across two specialized repositories:

### 🗂️ [Stroll.Theta.DB](https://github.com/revred/Stroll.Theta.DB) (This Repository)
**Purpose**: Database files, logs, and data validation
- **Contents**: SQLite database files, quality probe logs, CI workflows
- **Directory**: `C:\code\Stroll.Theta.DB\`
- **Focus**: Data storage, validation results, automated data integrity testing

### ⚙️ [Stroll.Theta](https://github.com/revred/Stroll.Theta)  
**Purpose**: Implementation code and data generation tools
- **Contents**: C# projects, data collection orchestration, ThetaData integration
- **Directory**: `C:\code\Stroll.Theta\`
- **Focus**: Data generation, RefineDataset implementation, collection infrastructure

### Integration
- **Data Flow**: `Stroll.Theta` → generates data → commits to `Stroll.Theta.DB`
- **CI/CD**: GitHub Actions in `Stroll.Theta.DB` validate data integrity
- **Testing**: RefineDataset CLI tools in `Stroll.Theta` ensure quality gates

## Additional Documentation

- **[FOLDER_STRUCTURE.md](FOLDER_STRUCTURE.md)**: Detailed directory organization and file naming conventions
- **Implementation Code**: [github.com/revred/Stroll.Theta](https://github.com/revred/Stroll.Theta)
- **Database Repository**: [github.com/revred/Stroll.Theta.DB](https://github.com/revred/Stroll.Theta.DB)

**Last Updated**: August 31, 2025  
**Collection Status**: Active (Aug 2025 → Jan 2018 backfill)  
**Data Quality**: 100% validation pass rate

