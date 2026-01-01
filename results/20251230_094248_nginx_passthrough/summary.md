# Benchmark Results

## Overview

| Property | Value |
|----------|-------|
| **Proxy** | nginx |
| **Version** | 1.25.5 |
| **Scenario** | passthrough |
| **Date** | 2025-12-30T09:42:48+01:00 |
| **Machine** | Mac16,1 |

## Environment

| Property | Value |
|----------|-------|
| **OS** | macOS 15.6 |
| **Kernel** | 24.6.0 |
| **CPU** | Apple M4 |
| **Cores** | 10 |
| **Memory** | 16 GB |
| **Architecture** | arm64 |

## Test Parameters

| Parameter | Value |
|-----------|-------|
| **Duration** | 60s |
| **Warmup** | 10s |
| **Connections** | 100 |
| **Target Rate** | 10000 RPS |
| **Runs** | 3 |
| **Load Generator** | oha 1.12.1 |

## Results

See individual run files in `runs/` directory:

- [Run 1](runs/run_1.txt)
- [Run 2](runs/run_2.txt)
- [Run 3](runs/run_3.txt)

## Raw Data

- [metadata.json](metadata.json) - Machine-readable metadata
- [runs/run_1.json](runs/run_1.json) - Run 1 data
- [runs/run_2.json](runs/run_2.json) - Run 2 data
- [runs/run_3.json](runs/run_3.json) - Run 3 data
