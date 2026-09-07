# Prequal-Reproduction

## High-Level Architecture
                         ┌─────────────────────┐
                         │ Experiment Controller│
                         │ / Configuration      │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │  Workload Generator │
                         │                     │
                         │ Request arrival rate│
                         │ Request parameters  │
                         └──────────┬──────────┘
                                    │
                                    ▼
                   CLIENT-SIDE LOAD BALANCER
        ┌─────────────────────────────────────────────────┐
        │                                                 │
        │  ┌───────────────────────────────────────────┐  │
        │  │         Candidate Selection               │  │
        │  │         Power of d Choices                │  │
        │  └───────────────────┬───────────────────────┘  │
        │                      ▼                          │
        │  ┌───────────────────────────────────────────┐  │
        │  │          PReQuaL Policy                   │  │
        │  │                                           │  │
        │  │ • RIF information                         │  │
        │  │ • Latency estimates                       │  │
        │  │ • Server ranking                          │  │
        │  └───────────────┬───────────────────────────┘  │
        │                  │                              │
        │       ┌──────────┴──────────┐                   │
        │       ▼                     ▼                   │
        │ ┌─────────────┐      ┌──────────────┐           │
        │ │ Server State│      │ Probe Manager│           │
        │ │ / Estimates │      │              │           │
        │ └─────────────┘      │ Async probes │           │
        │                      │ Reuse        │           │
        │                      └──────┬───────┘           │
        └─────────────────────────────┼───────────────────┘
                                      │
                          Requests + Probes
                                      │
              ┌───────────────────────┼───────────────────────┐
              ▼                       ▼                       ▼
       ┌──────────────┐        ┌──────────────┐       ┌──────────────┐
       │  Backend S1  │        │  Backend S2  │       │  Backend S3  │
       │              │        │              │       │              │
       │ • /work      │        │ • /work      │       │ • /work      │
       │ • RIF        │        │ • RIF        │       │ • RIF        │
       │ • latency    │        │ • latency    │       │ • latency    │
       │ • /probe     │        │ • /probe     │       │ • /probe     │
       └──────────────┘        └──────────────┘       └──────────────┘
              ▲                       ▲                       ▲
              │                       │                       │
       ┌──────┴──────┐         ┌──────┴──────┐        ┌──────┴──────┐
       │ Antagonist  │         │ Antagonist  │        │ Antagonist  │
       │ Load        │         │ Load        │        │ Load        │
       └─────────────┘         └─────────────┘        └─────────────┘


                     ┌──────────────────────┐
                     │ Metrics Collection   │
                     │                      │
                     │ • Mean latency       │
                     │ • p50/p95/p99        │
                     │ • Throughput         │
                     │ • Probe overhead     │
                     │ • RIF distribution   │
                     └──────────────────────┘

## Directory Structure

Prequal-Reproduction/
│
├── README.md
├── .gitignore
│
├── docs/
│   ├── architecture.md
│   ├── prequal-notes.md
│   └── experiments.md
│
├── backend/
│   ├── Cargo.toml
│   └── src/
│       ├── main.rs
│       ├── state.rs
│       ├── handlers/
│       │   ├── work.rs
│       │   ├── probe.rs
│       │   └── health.rs
│       └── workload.rs
│
├── load-balancer/
│   ├── Cargo.toml
│   └── src/
│       ├── main.rs
│       │
│       ├── policies/
│       │   ├── mod.rs
│       │   ├── random.rs
│       │   ├── round_robin.rs
│       │   └── prequal.rs
│       │
│       ├── candidate_selection.rs
│       ├── server_state.rs
│       ├── probe_manager.rs
│       └── router.rs
│
├── workload-generator/
│   ├── Cargo.toml
│   └── src/
│       ├── main.rs
│       └── generator.rs
│
├── antagonist/
│   ├── cpu_load.rs
│   └── workload_patterns.rs
│
├── experiments/
│   ├── configs/
│   │   ├── baseline.yaml
│   │   ├── antagonist_load.yaml
│   │   └── heterogeneous_servers.yaml
│   │
│   └── run_experiment.py
│
├── metrics/
│   ├── collector.py
│   └── analysis.py
│
├── docker/
│   ├── Dockerfile.backend
│   ├── Dockerfile.load-balancer
│   └── docker-compose.yml
│
└── results/
    └── .gitkeep