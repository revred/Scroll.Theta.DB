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

- **days/**: Daily SQLite files organized as `YYYY/MM/DD/SYMBOL.YYYY-MM-DD.sqlite`
- **weeks/**: Weekly rollup metadata and summaries
- **schema/**: Database schema definitions and migrations  
- **probes/**: Quality validation reports and probe results
- **manifests/**: Daily validation manifests with checksums and GREEN/RED status

## Data Coverage

**Symbols**: SPX, SPY, XSP, GLD, USO, VIX
**Period**: 2018-2025 (7-year backfill)
**Resolution**: 1-minute bars with options greeks
**Quality**: Enterprise-grade validation with comprehensive probe suite

Each day's data includes a `manifest.json` with validation results, checksums, and quality status before being committed to this repository.

## Usage

Clone this repository to access validated market data:
```bash
git clone https://github.com/revred/Scroll.Theta.DB.git
```

Individual day files can be accessed at:
```
days/YYYY/MM/DD/SYMBOL.YYYY-MM-DD.sqlite
```

Each SQLite database contains:
- `underlying_minute`: Minute-level OHLCV data
- `option_minute`: Options greeks and quotes
- `chain_daily`: Daily options chain summary