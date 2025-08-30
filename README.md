# Stroll.Theta Database Artifacts

This directory contains the daily-partitioned database files and validation metadata for the Stroll.Theta options backtesting system.

## Structure

- **days/**: Daily SQLite files organized as YYYY/MM/DD/SYMBOL.YYYY-MM-DD.sqlite
- **weeks/**: Weekly rollup metadata and summaries
- **schema/**: Database schema definitions and migrations
- **probes/**: Quality validation reports and probe results
- **manifests/**: Daily validation manifests with checksums and status

## Data Quality Standards

All data in this directory has passed comprehensive quality validation including:
- Join ratio ≥85% (underlying-options alignment)
- Zero-option days ≤2% miss rate
- Timestamp normalization (ET→UTC conversion)
- Sanity checks (IV bounds, bid/ask spreads, etc.)
- Integrity verification (no duplicate PKs, monotonic timestamps)

Each day's data includes a manifest.json with validation results, checksums, and GREEN/RED status.

