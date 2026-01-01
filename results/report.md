# Benchmark Comparison Report

Generated: 2025-12-29T17:28:13+01:00

## Scenario: passthrough

| Proxy | Version | RPS | p50 (ms) | p99 (ms) | Timestamp |
|-------|---------|----:|--------:|---------:|-----------|
| **haproxy** | 2.9.15 | 14854 | 6.26 | 22.14 | 2025-12-29 |
| **nginx** | 1.25.5 | 14669 | 6.46 | 15.02 | 2025-12-29 |
| **envoy** | v1.31 | 12851 | 7.57 | 13.55 | 2025-12-29 |
| **sentinel** | sentinel 0.1.6 (release 25.12_16, commit dca40bd-dirty) | 5889 | 16.64 | 30.01 | 2025-12-29 |
| **sentinel** | sentinel 0.1.6 (release 25.12_16, commit dca40bd-dirty) | 5598 | 17.81 | 27.03 | 2025-12-29 |

## Summary

- **passthrough**: haproxy leads with 14854 RPS

---
*Note: Results may vary based on hardware, container runtime, and system load.*
