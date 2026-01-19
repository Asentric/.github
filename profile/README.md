## Asentric

**A Go SDK for building real-time blockchain security monitoring with custom detection rules.**

[![Go Version](https://img.shields.io/badge/Go-%3E%3D%201.22-blue.svg)](https://golang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**🎥 [Watch Demo Video](https://youtu.be/Oz2dfZPI_8A)**

---

## Overview

Asentric SDK is a framework for real-time smart contract security monitoring that enables developers to:

- Define monitoring targets via YAML configuration
- Write custom detection logic in Go
- Run self-hosted watchers
- Receive alerts via webhook or console in real-time

Asentric is **not** a SaaS platform or YAML-based rule engine. It is an SDK with a runtime pattern similar to [Ponder.sh](https://ponder.sh), designed for developers who want full control over their security monitoring infrastructure.

**Built for the Mantle Network ecosystem, portable to any EVM chain.**

## Problem Statement

Most blockchain exploits are detectable minutes or hours before actual losses occur. Abnormal token transfers, suspicious privilege changes, and unusual contract interactions often precede attacks. However, existing tools either require complex infrastructure setup or lack the flexibility to detect protocol-specific risks.

**Our Solution:**
Asentric provides a developer-friendly SDK with:

- **Custom Detection Rules** - Write security logic in Go, not YAML
- **Real-time Monitoring** - WebSocket-based event streaming from any EVM chain
- **Zero Infrastructure** - Works out-of-the-box with no Redis, no databases (for basic usage)
- **Webhook Alerts** - Integrate with any backend or notification system
- **Mantle Native** - Pre-configured for Mantle Sepolia, ready for mainnet

## Key Features

| Feature | Description |
|---------|-------------|
| **Pure Detection Rules** | Write detection logic as pure functions with no side effects |
| **Real-time Monitoring** | WebSocket-based event streaming from any EVM chain |
| **Deterministic Execution** | Same input always produces same output, enabling replay and testing |
| **Zero Dependencies** | No external infrastructure required for basic usage |
| **Flexible Alerting** | Console output for development, webhook for production |
| **EVM Compatible** | Works with any EVM-compatible blockchain |

## Quick Start

### Installation

```bash
go install github.com/asentric/asentric/cmd/asentric@latest
```

### Create a New Project

```bash
asentric init my-watcher
cd my-watcher
go mod tidy
```

### Run

```bash
go run cmd/watcher/main.go
```

---

## Architecture

Asentric SDK follows a clean, deterministic architecture:

```
[Blockchain]
     │
     │ WebSocket (eth_subscribe)
     ▼
┌─────────────┐
│ EventSource │  ← Subscribe to logs/events
└─────────────┘
     │
     │ Event
     ▼
┌─────────────┐     ┌─────────────────┐
│ Dispatcher  │ ──► │ ContextBuilder  │
└─────────────┘     └─────────────────┘
     │                      │
     │                      │ Context
     │                      ▼
     │              ┌─────────────┐
     └────────────► │   Engine    │
                    └─────────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
      [Rule 1]        [Rule 2]        [Rule N]
           │               │               │
           └───────────────┼───────────────┘
                           │
                           │ Alerts
                           ▼
                    ┌─────────────┐
                    │  AlertSink  │  ← Send to webhook/console
                    └─────────────┘
```

## Core Design Principles

### SDK-First Approach

Asentric is designed as a **framework**, not a service:

- **YAML for configuration** - Configuration is declarative, not logic
- **Rules are code** - Detection logic written in Go, not config
- **Deterministic engine** - Same input always produces same output
- **Runtime handles I/O** - Side effects managed by runtime, not rules
- **Zero dependencies** - No external infrastructure required for basic usage

### Pure Detection Rules

Rules are pure functions:
- ✅ No side effects
- ✅ No I/O operations
- ✅ Deterministic
- ✅ Easy to test

### Clear Separation

| Component | Responsibility |
|-----------|----------------|
| **Engine** | Rule evaluation (pure, deterministic) |
| **Runtime** | Event ingestion, alert delivery (I/O) |
| **Rules** | Detection logic (pure functions) |
| **Context** | Immutable data snapshot |

---

## Core Components

### Engine

The rule execution engine that evaluates all registered rules against blockchain events.

- **Stateless** - No per-event state storage
- **Deterministic** - Same input → Same output
- **Pure** - No I/O operations

### Rules

Custom detection logic implemented as Go structs.

**Example Rule:**
```go
type LargeTransferRule struct {
    Threshold *big.Int
}

func (r *LargeTransferRule) Evaluate(ctx Context) (*Alert, error) {
    for _, log := range ctx.Logs() {
        if log.Event.Name == "Transfer" {
            value := utils.GetFieldBigInt(log.Event.Fields, "value")
            if value != nil && value.Cmp(r.Threshold) > 0 {
                return asentric.NewAlert(...), nil
            }
        }
    }
    return nil, nil
}
```

### Runtime

Orchestrates event ingestion, rule evaluation, and alert delivery.

- **EventSource** - WebSocket subscription to blockchain
- **Dispatcher** - Bridges events to engine
- **AlertSink** - Delivers alerts (console/webhook)

### Context

Immutable snapshot of transaction data, block info, and event logs.

- Read-only access to transaction data
- Decoded event logs
- Block metadata
- ABI registry for decoding

---

## Project Structure

### SDK Repository (asentric-sdk)

```
asentric-sdk/
├── pkg/asentric/          # Public SDK API
│   ├── engine.go          # Rule execution engine
│   ├── rule.go            # Rule interface
│   ├── context.go          # Execution context
│   ├── alert.go            # Alert model
│   └── runtime.go          # Runtime orchestrator
├── cmd/asentric/          # CLI tools
│   └── init               # Project scaffolding
├── internal/              # Private implementation
│   ├── source/            # EventSource implementations
│   ├── sink/              # AlertSink implementations
│   └── dispatcher/        # Event dispatcher
├── examples/              # Usage examples
└── docs/                  # Documentation
```

### User Project (Generated by `asentric init`)

```
my-watcher/
├── config/
│   ├── asentric.yaml      # Runtime configuration
│   └── registry.yaml      # Target contracts
├── rules/
│   └── example_rule.go    # Detection rules
├── abi/                   # Contract ABI files
├── cmd/
│   └── watcher/
│       └── main.go        # Entry point
└── go.mod
```

## Technology Stack

- **Language:** Go 1.22+
- **Blockchain:** Mantle Network (EVM-compatible, portable to any EVM chain)
- **Event Source:** WebSocket RPC (eth_subscribe)
- **Config:** YAML-based configuration
- **Alerting:** Console (dev) / Webhook (production)

---

## Current Status

**Hackathon MVP - Delivered Components**

✅ **Core SDK** (`pkg/asentric`)
- Pure, deterministic rule execution engine
- Context-based event processing
- Severity-based alert generation
- Extensible rule interface

✅ **CLI Tool** (`cmd/asentric`)
- Project scaffolding with `asentric init`
- Mantle Sepolia pre-configured
- Ready-to-run project templates

✅ **Runtime System**
- WebSocket event source for real-time monitoring
- Console and Webhook alert sinks
- Builder pattern for easy configuration

**Explicitly out of scope for MVP:**

- Production-ready UI dashboard
- Historical backfills and indexing jobs
- Multi-chain per project (1 chain per project)
- Plugin / extension marketplaces
- Long-term stable public APIs

---

## Philosophy

Asentric prioritizes **developer experience** and **architectural clarity** over feature completeness.

**Design Goals:**

- ✅ Understandable in hours, not weeks
- ✅ Easy to fork and adapt
- ✅ Clean starting point for production systems
- ✅ Simple > Complex
- ✅ Code > Config (rules are Go, not YAML)

---

## Documentation

- 📖 [Full Documentation](https://asentric-docs.vercel.app/) 

## Use Cases

### Protocol Teams
Deploy private instances with custom rules tailored to your contracts. Early warning system for incident detection.

### Security Researchers
Extend the heuristics engine, propose new detection rules, and build protocol-specific monitoring agents.

### Developers
Build custom security monitoring systems with full control over detection logic and infrastructure.

---

## Contributing

Contributions are welcome! Areas where contributions are particularly valuable:

- New detection rules
- Protocol registry entries
- Documentation improvements
- Bug fixes and testing
- Performance optimizations

Please refer to the main repository for contributing guidelines.

---

## License

MIT License. See the `LICENSE` file in the repository for details.

---

## Links

- 🌐 [Documentation](https://asentric.io)
- 💻 [GitHub Repository](https://github.com/asentric/asentric)
- 🐛 [Issue Tracker](https://github.com/asentric/asentric/issues)
- 🎥 [Demo Video](https://youtu.be/Oz2dfZPI_8A)

---

**Built for the Mantle Network ecosystem by security researchers and protocol developers.**


