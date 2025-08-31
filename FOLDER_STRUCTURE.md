# Stroll.Theta.DB Folder Structure Documentation

This document provides a comprehensive overview of the folder structure within the Stroll.Theta database repository.

## Repository Root Structure

```
C:/code/Stroll.Theta.DB/
├── .claude/                 # Claude Code configuration
├── .git/                    # Git version control metadata
├── days/                    # Primary data storage - daily SQLite files
├── logs/                    # Quality validation probe logs
├── manifests/               # Validation manifests and checksums
├── probes/                  # Probe configuration files
├── schema/                  # Database schema definitions
├── weeks/                   # Weekly summary metadata
├── README.md                # Main repository documentation
└── FOLDER_STRUCTURE.md      # This file
```

## Detailed Directory Breakdown

### /days/ - Primary Data Storage

The `days/` directory contains all SQLite database files organized in a hierarchical date-based structure:

```
days/
├── YYYY/                    # Year folders (2018-2025)
│   ├── MM/                  # Month folders (01-12)
│   │   ├── DD/              # Day folders (01-31)
│   │   │   ├── SYMBOL.YYYY-MM-DD.sqlite  # Daily database files
│   │   │   └── logs/        # Day-specific validation logs (deprecated)
│   │   └── [More days...]
│   └── [More months...]
└── [Symbol-specific folders - DEPRECATED]
    ├── spx/YYYY/            # Legacy SPX organization
    ├── vix/YYYY/            # Legacy VIX organization
    ├── spy/YYYY/            # Legacy SPY organization
    ├── gld/YYYY/            # Legacy GLD organization
    ├── uso/YYYY/            # Legacy USO organization
    └── xsp/YYYY/            # Legacy XSP organization
```

#### Current Data Structure (Post-Reorganization)
- **Path Format**: `days/YYYY/MM/DD/SYMBOL.YYYY-MM-DD.sqlite`
- **Example**: `days/2023/06/05/SPX.2023-06-05.sqlite`
- **Symbols**: SPX, VIX, GLD, SPY, USO, XSP
- **Date Range**: 2018-01-01 through 2025-08-30 (ongoing backfill)

#### Legacy Symbol-Based Structure (Being Phased Out)
- **Path Format**: `days/symbol/YYYY/SYMBOL.YYYY-MM-DD.sqlite`
- **Status**: Deprecated - Use date-based structure instead
- **Migration**: Files gradually moved to date-based organization

### /logs/ - Quality Validation Logs

Central logging directory for all data quality validation runs:

```
logs/
├── YYYY-MM-DD-HH-MM-SS/     # Timestamped probe run directories
│   ├── probe-summary.json   # Overall validation summary
│   ├── data-integrity.log   # Data integrity check results
│   ├── risk-analysis.log    # Risk analysis validation
│   ├── pattern-detection.log # Market pattern analysis
│   └── [Additional probe logs...]
└── [More timestamped runs...]
```

#### Log Structure Details
- **Naming**: Timestamp format `YYYY-MM-DD-HH-MM-SS`
- **Frequency**: Generated after each data collection/validation run
- **Retention**: All logs preserved for audit trail
- **Content**: Comprehensive validation results with health scores

### /manifests/ - Validation Manifests

Storage for validation manifests and data integrity checksums:

```
manifests/
├── YYYY-MM-DD/              # Daily manifest directories
│   ├── manifest.json       # Daily validation summary
│   ├── checksums.sha256     # File integrity checksums
│   └── status.json          # Overall day validation status
└── [More daily manifests...]
```

#### Manifest Contents
- **SHA256 Checksums**: For each SQLite file
- **Validation Status**: GREEN/RED quality flags
- **Probe Results**: Summary of all validation checks
- **Data Statistics**: File sizes, record counts, coverage metrics

### /probes/ - Quality Probe Configurations

Configuration files for the data validation probe system:

```
probes/
├── data-integrity.config    # Data integrity probe settings
├── risk-analysis.config     # Risk analysis probe configuration
├── pattern-detection.config # Pattern detection parameters
└── validation-rules.json    # Master validation rule definitions
```

#### Probe Types
- **Data Integrity**: Verifies database structure, constraints, and consistency
- **Risk Analysis**: Validates trading risk calculations and position data
- **Pattern Detection**: Identifies market patterns and anomalies
- **Custom Probes**: Extensible framework for additional validation

### /schema/ - Database Schema Definitions

Database schema definitions and migration scripts:

```
schema/
├── current/                 # Current schema version
│   ├── underlying_minute.sql # Underlying data table schema
│   ├── option_minute.sql    # Options data table schema
│   └── indexes.sql          # Index definitions
├── migrations/              # Schema migration scripts
│   ├── v1_to_v2.sql        # Version migration scripts
│   └── [More migrations...]
└── documentation/           # Schema documentation
    ├── field-definitions.md # Detailed field descriptions
    └── query-examples.sql   # Common query patterns
```

#### Schema Management
- **Versioning**: Tracked schema versions with migration paths
- **Consistency**: Enforced across all SQLite files
- **Documentation**: Comprehensive field definitions and examples
- **Evolution**: Controlled schema updates with backward compatibility

### /weeks/ - Weekly Summary Metadata

Weekly aggregation and summary data:

```
weeks/
├── YYYY-WW/                 # Year-week directories (ISO week numbering)
│   ├── summary.json        # Week trading summary
│   ├── statistics.json     # Aggregate statistics
│   └── quality-report.json # Weekly quality assessment
└── [More weekly summaries...]
```

#### Weekly Summaries Include
- **Trading Statistics**: Volume, volatility, range summaries
- **Data Quality Metrics**: Weekly validation success rates
- **Coverage Analysis**: Symbol and time coverage assessment
- **Performance Metrics**: Data collection and processing performance

## Data Organization Principles

### 1. Hierarchical Date Organization
- **Primary Structure**: Year → Month → Day → Files
- **Benefits**: Efficient navigation, logical grouping, scalable structure
- **Query Optimization**: Date-based queries leverage filesystem structure

### 2. Symbol Coexistence
- **Multi-Symbol Days**: All symbols for a trading day in same directory
- **Cross-Symbol Analysis**: Simplified multi-symbol research and validation
- **Atomic Daily Units**: Complete trading day data in single directory

### 3. Quality-First Architecture
- **Validation Logs**: Comprehensive logging alongside data
- **Integrity Checking**: SHA256 checksums and validation status
- **Audit Trail**: Complete history of all data operations and validations

### 4. Self-Contained Repository
- **Data + Logs**: Both data and validation logs in same repository
- **Complete Context**: All information needed for data analysis included
- **Version Control**: Full history of data collection and quality evolution

## File Naming Conventions

### Database Files
- **Format**: `SYMBOL.YYYY-MM-DD.sqlite`
- **Examples**: 
  - `SPX.2023-06-05.sqlite` (SPX index and options)
  - `VIX.2023-06-05.sqlite` (VIX index only)
  - `SPY.2023-06-05.sqlite` (SPY ETF and options)

### Log Directories
- **Format**: `YYYY-MM-DD-HH-MM-SS`
- **Example**: `2025-08-30-11-54-24`
- **Timezone**: Local system time (ET/EST)

### Manifest Files
- **Daily**: `YYYY-MM-DD/manifest.json`
- **Checksums**: `YYYY-MM-DD/checksums.sha256`
- **Status**: `YYYY-MM-DD/status.json`

## Access Patterns and Performance

### Optimized Access Patterns
1. **Single Day Analysis**: Direct path to `days/YYYY/MM/DD/`
2. **Date Range Queries**: Efficient traversal of date hierarchy
3. **Symbol-Specific Analysis**: Filter within date directories
4. **Quality Validation**: Cross-reference with logs and manifests

### Performance Considerations
- **SQLite Indexes**: Optimized for timestamp and symbol queries
- **File Size Management**: Daily partitioning prevents large file issues
- **Parallel Processing**: Date-based structure enables parallel analysis
- **Caching Strategy**: Filesystem caching benefits from logical organization

## Migration and Maintenance

### Ongoing Migrations
- **Symbol-Based → Date-Based**: Gradual migration from legacy structure
- **Log Consolidation**: Moving day-specific logs to central logs directory
- **Manifest Generation**: Retroactive manifest creation for existing data

### Maintenance Procedures
- **Integrity Checks**: Regular verification of file checksums and structure
- **Log Rotation**: Management of growing log directory
- **Schema Evolution**: Controlled updates with migration support
- **Quality Monitoring**: Continuous monitoring of data collection quality

## Future Structure Evolution

### Planned Enhancements
- **Yearly Archives**: Compression of older years for storage efficiency
- **Symbol Categories**: Enhanced organization by instrument type
- **Real-Time Integration**: Structure to support real-time data streams
- **Multi-Exchange Support**: Extension for multiple data sources

### Scalability Considerations
- **Storage Growth**: Current ~100GB projected for 7-year coverage
- **File Count Management**: Balance between granularity and file count
- **Query Performance**: Maintain efficient query patterns as data grows
- **Backup Strategy**: Structure optimized for incremental backup systems

---

**Last Updated**: August 30, 2025  
**Structure Version**: 2.0 (Date-based organization)  
**Migration Status**: In Progress (Symbol-based → Date-based)