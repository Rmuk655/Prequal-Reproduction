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

## Project Structure

```text
Prequal-Reproduction/
├── README.md
├── .gitignore
├── CMakeLists.txt
│
├── backend/
│   ├── CMakeLists.txt
│   ├── include/
│   │   ├── state.h
│   │   └── workload.h
│   └── src/
│       ├── main.cpp
│       ├── state.cpp
│       ├── workload.cpp
│       └── handlers/
│           ├── work.cpp
│           ├── probe.cpp
│           └── health.cpp
│
├── load-balancer/
│   ├── CMakeLists.txt
│   ├── include/
│   │   ├── router.h
│   │   ├── candidate_selection.h
│   │   ├── server_state.h
│   │   └── probe_manager.h
│   └── src/
│       ├── main.cpp
│       ├── router.cpp
│       ├── candidate_selection.cpp
│       ├── server_state.cpp
│       ├── probe_manager.cpp
│       └── policies/
│           ├── random.cpp
│           ├── round_robin.cpp
│           └── prequal.cpp
│
├── workload-generator/
│   ├── CMakeLists.txt
│   ├── include/
│   │   └── generator.h
│   └── src/
│       ├── main.cpp
│       └── generator.cpp
│
├── antagonist/
│   ├── CMakeLists.txt
│   └── src/
│       ├── main.cpp
│       ├── cpu_load.cpp
│       └── workload_patterns.cpp
│
├── experiments/
│   ├── configs/
│   │   ├── baseline.yaml
│   │   ├── antagonist_load.yaml
│   │   └── heterogeneous_servers.yaml
│   └── run_experiment.sh
│
├── metrics/
│   ├── collector.cpp
│   └── analysis.py
│
├── docker/
│   ├── Dockerfile.backend
│   ├── Dockerfile.load-balancer
│   └── docker-compose.yml
│
├── docs/
│   ├── architecture.md
│   ├── prequal-notes.md
│   └── experiments.md
│
└── results/
    └── .gitkeep
```