# Bronze Tier Data Quality Workshop

## Overview

This workshop demonstrates **10 common mistakes** organizations make when uploading data to their bronze tier, and how to progressively fix them using Expanso pipelines.

## The Problem

Bronze tier data lakes often become data swamps because teams upload raw data without proper quality controls. This leads to:
- Storage bloat from duplicate records
- Compliance risks from exposed PII
- Query failures from schema inconsistencies
- Downstream pipeline crashes from malformed data

## Workshop Structure

### Module 0: The Problematic Pipeline
A high-volume data generator that exhibits all 10 anti-patterns.

### Progressive Fixes

| Module | Mistake | Solution | Processor |
|--------|---------|----------|-----------|
| 1 | No event IDs | Add UUID generation | `mapping` |
| 2 | Inconsistent timestamps | Normalize to ISO 8601 | `mapping` |
| 3 | Duplicate events | Cache-based deduplication | `dedupe` |
| 4 | Exposed PII | Hash/mask sensitive fields | `mapping` |
| 5 | No schema validation | Validate required fields | `mapping` + `switch` |
| 6 | Test data in production | Filter by environment | `mapping` + `deleted()` |
| 7 | Unbatched writes | Add count/time batching | `batching` |
| 8 | No rate limiting | Throttle throughput | `rate_limit` |
| 9 | No error handling | Try/catch with DLQ | `try` + `catch` |
| 10 | Single output failure | Multi-output fan-out | `broker` |

## Running the Workshop

### Prerequisites
- Expanso Edge installed (`curl -fsSL https://get.expanso.io/edge/install.sh | bash`)
- Or use Docker: `docker pull ghcr.io/expanso-io/expanso-edge:nightly`

### Quick Start

```bash
# Run the problematic pipeline (generates bad data fast!)
expanso-edge run -c pipelines/00-problematic-generator.yaml

# Run each fix progressively
expanso-edge run -c pipelines/01-add-event-ids.yaml
expanso-edge run -c pipelines/02-normalize-timestamps.yaml
# ... continue through all modules

# Run the final production-ready pipeline
expanso-edge run -c solutions/production-pipeline.yaml
```

## Key Concepts

### Bloblang Functions Used
- `uuid_v4()` - Generate unique identifiers
- `now()` - Current timestamp
- `fake("email")` - Synthetic data generation
- `random_int()` - Random number generation
- `deleted()` - Remove messages from pipeline

### Processor Patterns
- **Mapping**: Transform message structure
- **Switch**: Conditional routing
- **Dedupe**: Remove duplicates via cache
- **Branch**: Parallel processing with merge
- **Try/Catch**: Error handling
- **Broker**: Fan-out to multiple outputs

## The 10 Mistakes Explained

### 1. No Event IDs
**Problem**: Events arrive without unique identifiers, making deduplication and tracking impossible.
**Impact**: Can't correlate events, can't dedupe, can't audit.

### 2. Inconsistent Timestamps
**Problem**: Mixed formats (Unix, ISO, custom), null values, future dates.
**Impact**: Time-series queries fail, ordering is wrong, analytics are unreliable.

### 3. Duplicate Events
**Problem**: Retry logic, network issues, or upstream bugs cause duplicates.
**Impact**: Inflated metrics, storage bloat, incorrect aggregations.

### 4. Exposed PII
**Problem**: Email addresses, SSNs, names stored in plain text.
**Impact**: Compliance violations (GDPR, CCPA), security risks.

### 5. No Schema Validation
**Problem**: Missing required fields, wrong data types, unexpected nulls.
**Impact**: Downstream jobs crash, ETL failures, data quality issues.

### 6. Test Data in Production
**Problem**: Debug flags, test users, synthetic data mixed with real data.
**Impact**: Polluted analytics, false alerts, wasted storage.

### 7. Unbatched Writes
**Problem**: Writing records one at a time to storage.
**Impact**: High I/O costs, slow ingestion, connection exhaustion.

### 8. No Rate Limiting
**Problem**: Burst traffic overwhelms downstream systems.
**Impact**: Dropped data, cascading failures, increased costs.

### 9. No Error Handling
**Problem**: Bad records cause entire pipeline to crash.
**Impact**: Data loss, operational overhead, alert fatigue.

### 10. Single Output Point of Failure
**Problem**: One destination means one failure mode.
**Impact**: Data loss during outages, no backup, no redundancy.

## Files

```
workshops/bronze-tier-best-practices/
├── README.md                              # This file
├── pipelines/
│   ├── 00-problematic-generator.yaml      # The bad pipeline
│   ├── 01-add-event-ids.yaml              # Fix #1
│   ├── 02-normalize-timestamps.yaml       # Fix #2
│   ├── 03-deduplicate-events.yaml         # Fix #3
│   ├── 04-mask-pii.yaml                   # Fix #4
│   ├── 05-validate-schema.yaml            # Fix #5
│   ├── 06-filter-test-data.yaml           # Fix #6
│   ├── 07-add-batching.yaml               # Fix #7
│   ├── 08-add-rate-limiting.yaml          # Fix #8
│   ├── 09-add-error-handling.yaml         # Fix #9
│   └── 10-multi-output-fanout.yaml        # Fix #10
└── solutions/
    └── production-pipeline.yaml           # All fixes combined
```
