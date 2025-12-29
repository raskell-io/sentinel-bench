# Sentinel Benchmarking Framework

## Overview

This framework provides a fair, reproducible methodology for benchmarking Sentinel against other popular reverse proxies: Envoy, HAProxy, and Nginx.

## Design Principles

1. **Fair Comparison** - Identical workloads, equivalent configurations, same hardware
2. **Coordinated Omission Correction** - Use tools like wrk2/Nighthawk that account for this
3. **Measure Below the Knee** - Test at realistic load levels, not just maximum throughput
4. **Reproducibility** - Infrastructure-as-code, containerized, automated
5. **Local-First** - Run benchmarks on local hardware before scaling to cloud

## Repository Structure

```
sentinel-bench/
├── docs/                   # Documentation and methodology
├── configs/                # Equivalent proxy configurations
│   ├── sentinel/           # Sentinel KDL configs
│   ├── envoy/              # Envoy YAML configs
│   ├── haproxy/            # HAProxy configs
│   └── nginx/              # Nginx configs
├── scenarios/              # Test scenario definitions
├── runners/                # Orchestration scripts
├── analysis/               # Result processing & visualization
├── results/                # Raw results & reports
├── docker/                 # Docker Compose infrastructure
└── scripts/                # Utility scripts
```

## Load Generation Tools

| Tool | Use Case | Why |
|------|----------|-----|
| **wrk2** | Primary throughput/latency | Corrects coordinated omission, well-understood |
| **Nighthawk** | Advanced HTTP/2/3 testing | Envoy's own tool, HdrHistogram, open-loop mode |
| **k6** | Scripted scenarios | Complex user journeys, JavaScript extensibility |
| **oha** | Quick validation | Rust-based, fast iteration during development |

### Tool Selection Rationale

**wrk2** is preferred over the original wrk because it:
- Maintains constant throughput regardless of server response time
- Corrects for "coordinated omission" - a measurement error where slow responses cause the load generator to inadvertently reduce request rate
- Produces accurate latency percentiles under load

**Nighthawk** (from the Envoy project) offers:
- Native HTTP/2 and HTTP/3 support
- Open-loop mode for stress testing
- HdrHistogram for accurate percentile calculations
- gRPC-based distributed load generation

## Test Scenarios

### Tier 1: Core Performance (Required for Claims)

These scenarios establish baseline performance characteristics that any proxy should excel at.

| Scenario | Description | Metrics |
|----------|-------------|---------|
| **Passthrough** | Simple L7 proxy, no processing | RPS, p50/p95/p99 latency |
| **Routing** | 100+ routes, path-based matching | Route match overhead |
| **Load Balancing** | Round-robin, P2C, least-conn | Distribution fairness, RPS |
| **TLS Termination** | HTTPS frontend, HTTP backend | Handshake cost, session reuse |
| **Connection Scaling** | 1K, 10K, 100K concurrent | Memory, CPU, latency degradation |

### Tier 2: Sentinel-Specific Features

These scenarios highlight Sentinel's unique capabilities.

| Scenario | Description | Metrics |
|----------|-------------|---------|
| **Agent Pipeline** | Echo agent on request path | Agent overhead per call |
| **Rate Limiting** | Token bucket under load | Accuracy, false positives |
| **Circuit Breaker** | Backend failures, recovery | Trip latency, recovery time |
| **Body Inspection** | Request/response body processing | Throughput with body sizes |

### Tier 3: Soak & Stress

Long-running tests to identify memory leaks and stability issues.

| Scenario | Description | Duration |
|----------|-------------|----------|
| **Soak Test** | Steady load, memory leak detection | 24-72 hours |
| **Spike Test** | 10x load bursts | 1 hour with bursts |
| **Chaos** | Backend failures, network partitions | 4 hours |

## Metrics

### Throughput Metrics

```yaml
throughput:
  - requests_per_second        # Successful requests per second
  - bytes_per_second           # Total throughput in bytes
  - successful_requests_pct    # Success rate percentage
```

### Latency Metrics

```yaml
latency:
  - p50                        # Median latency
  - p75                        # 75th percentile
  - p90                        # 90th percentile
  - p95                        # 95th percentile
  - p99                        # 99th percentile
  - p99.9                      # 99.9th percentile (tail latency)
  - p99.99                     # 99.99th percentile
  - max                        # Maximum observed latency
  - time_to_first_byte         # TTFB
  - connection_time            # TCP connection establishment
  - tls_handshake_time         # TLS negotiation time
```

### Resource Usage Metrics

```yaml
resource_usage:
  - cpu_percent_user           # User-space CPU usage
  - cpu_percent_system         # Kernel CPU usage
  - memory_rss_mb              # Resident set size
  - memory_peak_mb             # Peak memory usage
  - file_descriptors           # Open file descriptor count
  - connections_active         # Active connection count
```

### Error Metrics

```yaml
errors:
  - connection_errors          # Failed connections
  - timeout_errors             # Request timeouts
  - http_errors_by_status      # Breakdown by HTTP status code
```

## Configuration Equivalence

For fair comparison, all proxies must be configured equivalently:

| Feature | Sentinel | Envoy | HAProxy | Nginx |
|---------|----------|-------|---------|-------|
| Workers | `workers = N` | `--concurrency N` | `nbproc N` | `worker_processes N` |
| Max Connections | `max_connections` | `overload_manager` | `maxconn` | `worker_connections` |
| Connection Pooling | `upstream.pool` | `cluster.connection_pool` | `pool` | `keepalive` |
| Health Checks | `health_check` | `health_checks` | `option httpchk` | `health_check` |
| Access Logging | **Disabled** | **Disabled** | **Disabled** | **Disabled** |
| Circuit Breakers | **Disabled** | **Disabled** | N/A | N/A |

**Critical**: All proxies must have access logging **disabled** during benchmarks, as logging introduces significant I/O overhead.

## Local Infrastructure

### Docker Compose Architecture

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Client     │────▶│    Proxy     │────▶│   Backend    │
│   Container  │     │   Container  │     │   Container  │
│   (wrk2/oha) │     │   (SUT)      │     │   (echo)     │
└──────────────┘     └──────────────┘     └──────────────┘
                           │
                    ┌──────▼──────┐
                    │  Metrics    │
                    │  Collector  │
                    │ (Prometheus)│
                    └─────────────┘
```

### Container Resource Allocation

For reproducible results, containers should have:
- CPU limits matching the test configuration
- Memory limits to detect OOM conditions
- Network isolation (dedicated Docker network)

### Local Machine Preparation

For accurate local benchmarks:

1. **Close unnecessary applications** - Browsers, IDEs, etc. consume resources
2. **Disable CPU throttling** - If possible, set CPU governor to "performance"
3. **Use wired network** - Avoid WiFi variability for network tests
4. **Multiple runs** - Run each scenario 3-5 times, report median

## Avoiding Common Pitfalls

| Pitfall | Solution |
|---------|----------|
| Coordinated omission | Use wrk2 or Nighthawk (corrects by default) |
| Measuring at max load | Test at 50%, 70%, 90% of saturation |
| Single connection tests | Minimum 100 connections per worker thread |
| Ignoring warmup | 30s warmup before measurement |
| Different TLS settings | Standardize cipher suites, session reuse |
| Backend bottleneck | Backend must be 10x faster than proxy |
| Container resource contention | Dedicated CPU/memory limits |
| Network namespace overhead | Use host network for load generator |

## Methodology

### Standard Test Procedure

1. **Warmup Phase** (30 seconds)
   - Establish connections
   - Prime caches and JIT (for JVM-based proxies)
   - Not included in measurements

2. **Measurement Phase** (60-120 seconds)
   - Constant request rate (for latency tests)
   - Maximum throughput (for RPS tests)
   - Collect all metrics

3. **Cooldown Phase** (10 seconds)
   - Graceful connection close
   - Final metric collection

### Load Levels

Test at multiple load levels to characterize performance:

- **Light**: 10% of estimated capacity
- **Medium**: 50% of estimated capacity
- **Heavy**: 80% of estimated capacity
- **Stress**: 100%+ of estimated capacity

### Statistical Rigor

- Run each scenario **minimum 5 times**
- Report **median** values (not mean)
- Include **standard deviation** for variability
- Discard outliers using IQR method

## Future: Cloud Infrastructure

When scaling to cloud benchmarks, use:

### Recommended Setup (AWS)

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   Client     │────▶│    Proxy     │────▶│   Backend    │
│  (c6i.4xl)   │     │  (c6i.2xl)   │     │  (c6i.2xl)   │
│  Nighthawk   │     │  SUT         │     │  echo-server │
└──────────────┘     └──────────────┘     └──────────────┘
```

### Isolation Requirements

- Dedicated hosts (no noisy neighbors)
- CPU frequency scaling disabled
- C-states disabled in BIOS
- IRQ affinity configured
- Network: 10Gbps+ between nodes

## Implementation Phases

### Phase 1: Foundation (Current)
- Repository structure
- Docker Compose for local testing
- Equivalent proxy configurations
- Basic wrk2/oha runner with JSON output

### Phase 2: Scenario Library
- Implement all Tier 1 scenarios
- Automated configuration generation
- Result aggregation and comparison scripts

### Phase 3: CI Integration
- Performance regression detection
- PR gates with performance budgets
- Historical tracking dashboard

### Phase 4: Public Benchmarks
- Reproducible benchmark suite
- Published methodology
- Regular updates (quarterly)

## References

- [Envoy Benchmarking Best Practices](https://www.envoyproxy.io/docs/envoy/latest/faq/performance/how_to_benchmark_envoy)
- [Nighthawk Load Generator](https://github.com/envoyproxy/nighthawk)
- [wrk2 - Constant Throughput HTTP Benchmarking](https://github.com/giltene/wrk2)
- [Coordinated Omission (Gil Tene)](https://www.youtube.com/watch?v=lJ8ydIuPFeU)
- [How NOT to Measure Latency](https://www.youtube.com/watch?v=lJ8ydIuPFeU)
- [NickMRamirez/Proxy-Benchmarks](https://github.com/NickMRamirez/Proxy-Benchmarks)
