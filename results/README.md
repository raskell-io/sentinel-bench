# Benchmark Results

This directory contains benchmark results from `mise run bench`.

## Directory Structure

Each benchmark run creates a timestamped directory:

```
results/
├── 20241229_143022_envoy_passthrough/
│   ├── metadata.json     # Machine-readable metadata (for analytics)
│   ├── summary.md        # Human-readable summary
│   └── runs/
│       ├── run_1.txt     # Run 1 output (human-readable)
│       ├── run_1.json    # Run 1 data (machine-readable)
│       ├── run_2.txt
│       ├── run_2.json
│       └── ...
├── 20241229_144500_haproxy_passthrough/
│   └── ...
└── ...
```

## Metadata Schema (v1.0.0)

Each `metadata.json` contains:

```json
{
  "schema_version": "1.0.0",
  "id": "unique-uuid",
  "timestamp": {
    "start": "ISO-8601 timestamp",
    "end": "ISO-8601 timestamp",
    "timezone": "UTC"
  },
  "benchmark": {
    "proxy": "envoy|haproxy|nginx|sentinel",
    "scenario": "passthrough|routing|tls|...",
    "name": "optional custom name",
    "config_file": "configs/envoy/passthrough.yaml"
  },
  "proxy_info": {
    "name": "envoy",
    "version": "1.31.0",
    "container": "bench-envoy"
  },
  "environment": {
    "hostname": "machine-name",
    "machine_model": "MacBookPro18,3",
    "os": {
      "name": "macOS",
      "version": "14.0",
      "kernel": "23.0.0"
    },
    "cpu": {
      "model": "Apple M1 Pro",
      "cores": 10,
      "architecture": "arm64"
    },
    "memory_gb": 32,
    "container_runtime": "Docker version 24.0.0"
  },
  "test_parameters": {
    "duration_seconds": 60,
    "warmup_seconds": 10,
    "connections": 100,
    "threads": 4,
    "target_rate": 10000,
    "runs": 3,
    "target_url": "http://localhost:8080"
  },
  "load_generator": {
    "tool": "oha",
    "version": "1.4.0"
  },
  "runs": [
    {"run": 1, "output": "runs/run_1.txt", "data": "runs/run_1.json"},
    {"run": 2, "output": "runs/run_2.txt", "data": "runs/run_2.json"},
    {"run": 3, "output": "runs/run_3.txt", "data": "runs/run_3.json"}
  ]
}
```

## Analytics

To aggregate results across all runs:

```bash
# List all benchmarks
find results -name "metadata.json" | xargs jq -s '.'

# Get all Envoy results
find results -name "metadata.json" -exec jq 'select(.benchmark.proxy == "envoy")' {} \;

# Compare proxies for a scenario
find results -name "metadata.json" -exec jq -r '[.benchmark.proxy, .proxy_info.version] | @tsv' {} \;
```

## GitHub Pages

Results are tracked in git for display on GitHub Pages. Each benchmark run can be browsed:

- `summary.md` - Rendered as a readable report
- `metadata.json` - Consumed by visualization tools
- `runs/*.json` - Detailed data for charts and graphs
