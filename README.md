# sentinel-bench

Benchmarking framework for comparing Sentinel against Envoy, HAProxy, and Nginx.

## Quick Start

### Prerequisites

- [mise](https://mise.jdx.dev) - Task runner and tool manager
- [Docker](https://www.docker.com) - For containerized proxies

### Setup

```bash
# Trust and install dependencies
mise trust
mise run setup
```

This installs load generators (hey, k6, wrk, oha) and validates your environment.

### Run a Quick Benchmark

```bash
# Start a proxy
mise run up envoy

# Run quick 30-second benchmark
mise run quick

# Stop
mise run down
```

### Full Benchmark

```bash
# Full benchmark with multiple runs
mise run bench envoy passthrough

# With custom parameters
mise run bench envoy passthrough --duration 120 --rate 50000

# Compare all proxies
mise run bench all passthrough
```

## Tasks

Run `mise tasks` to see all available tasks:

| Task | Description |
|------|-------------|
| `mise run setup` | Install dependencies and validate environment |
| `mise run bench <proxy> <scenario>` | Run full benchmark |
| `mise run quick [url]` | Quick 30-second benchmark |
| `mise run up <proxy>` | Start proxy container |
| `mise run down` | Stop all containers |
| `mise run check-tools` | Show available load generators |
| `mise run results` | List benchmark results |
| `mise run clean` | Clean results directory |

## Repository Structure

```
sentinel-bench/
├── mise.toml           # Tool dependencies and short tasks
├── .mise/tasks/        # File-based tasks (bench, quick, setup)
├── docs/               # Methodology and design docs
│   └── CONCEPT.md      # Framework design document
├── configs/            # Equivalent proxy configurations
│   ├── sentinel/       # Sentinel KDL configs
│   ├── envoy/          # Envoy YAML configs
│   ├── haproxy/        # HAProxy configs
│   └── nginx/          # Nginx configs
├── scenarios/          # Test scenario definitions
├── docker/             # Docker Compose infrastructure
└── results/            # Benchmark results (gitignored)
```

## Scenarios

### Tier 1: Core Performance

| Scenario | Description |
|----------|-------------|
| `passthrough` | Simple L7 proxy, no processing |
| `routing` | 100+ routes, path-based matching |
| `loadbalancing` | Round-robin, least-conn, P2C |
| `tls` | HTTPS termination |
| `connection_scaling` | 1K, 10K, 50K concurrent connections |

### Tier 2: Sentinel-Specific

| Scenario | Description |
|----------|-------------|
| `agent_pipeline` | External agent processing overhead |
| `rate_limiting` | Token bucket accuracy under load |
| `circuit_breaker` | Trip/recovery behavior |

## Configuration

### Environment Variables

Set in `mise.toml` or override in your shell:

```bash
export BENCH_DURATION=60      # Test duration (seconds)
export BENCH_CONNECTIONS=100  # Concurrent connections
export BENCH_RATE=10000       # Target requests/second
export BENCH_WARMUP=10        # Warmup duration (seconds)
export BENCH_RUNS=3           # Number of runs
export BENCH_TARGET=http://localhost:8080
```

### Load Generator Priority

The framework auto-detects load generators in this order:

1. **oha** - Rust-based, HdrHistogram, recommended
2. **wrk2** - Corrects coordinated omission
3. **wrk** - Fast but no coordinated omission correction
4. **hey** - Simple HTTP load generator
5. **k6** - Scriptable, good for complex scenarios

Install the recommended generator:

```bash
mise run install-oha
```

## Configuration Equivalence

All proxies are configured with equivalent settings:

- Same worker thread count (auto-detect)
- Same connection limits
- Same keepalive settings
- **Access logging disabled** (critical for fair comparison)
- Same timeout values

See `docs/CONCEPT.md` for detailed equivalence matrix.

## Metrics Collected

- **Throughput**: requests/second, bytes/second
- **Latency**: p50, p75, p90, p95, p99, p99.9, max
- **Resources**: CPU%, memory MB, file descriptors
- **Errors**: connection errors, timeouts, HTTP errors

## Results

Results are saved to `results/<proxy>/<scenario>/<timestamp>/`:

```
results/
└── envoy/
    └── passthrough/
        └── 20241229_143022/
            ├── run_1.txt
            ├── run_2.txt
            ├── run_3.txt
            ├── params.json
            └── system_info.txt
```

## Methodology

See `docs/CONCEPT.md` for:

- Coordinated omission correction
- Warmup and cooldown procedures
- Statistical rigor (multiple runs, median reporting)
- Common pitfalls to avoid

## License

MIT
