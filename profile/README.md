## Asentric

**Mantle Security Realtime Alerting**

A lightweight, logs-based smart contract security monitoring system for Mantle Network, built for speed, clarity, and hackathon iteration.

---

## Overview

Asentric is an open-source, logs-based on-chain security monitoring system for the Mantle Network.

It ingests Ethereum event logs directly from Mantle RPC, normalizes them into structured security events, evaluates them using a rule engine, and publishes actionable alerts in near real-time.

The project serves both as a public security monitoring service and as a developer SDK, enabling protocol teams and independent developers to extend the rule engine, add custom protocol mappings, and deploy private monitoring instances.

**🎥 [Watch Demo Video](https://youtu.be/Oz2dfZPI_8A)**

## Problem Statement

Developers and users on Mantle Network currently lack real-time visibility into high-risk on-chain events such as:

- Unauthorized role or permission changes
- Large withdrawals from protocol vaults
- Sensitive function calls
- Smart contract upgrades
- Suspicious interactions from new or unknown wallets

While general monitoring solutions (Forta, Tenderly, block explorers) exist, they are not tailored for the Mantle ecosystem. Asentric addresses this gap by providing Mantle-first risk detection that focuses exclusively on smart contract interactions, reducing noise while delivering faster and more contextually relevant alerts.

## Architecture

Asentric follows a modular, event-driven architecture:

```text
Watcher (Go, Infra-only)
    ├─ Subscribes to logs via SubscribeFilterLogs (WebSocket)
    ├─ Normalizes raw logs into LogEvent
    └─ Optionally publishes LogEvent to Redis Streams (stream:raw)

Processor (Go, Domain)
    ├─ Consumes LogEvent from Redis
    ├─ Resolves protocol metadata (registry)
    ├─ Applies rule heuristics
    ├─ Emits security alerts
    └─ Publishes to alert stream (stream:alerts)

Dispatcher (Go, Output)
    ├─ Consumes security alerts
    ├─ Formats messages
    └─ Sends to X / Webhook / other channels (mocked for MVP)
```

**Message Transport:** Redis Streams  
Used for ordered processing, replayability, and simple horizontal scaling via consumer groups.

---

## Core Design Principles (Hackathon SDK)

This project intentionally does **not** provide:

- Plugin ecosystems or extension marketplaces
- Dynamic loading of untrusted third-party code
- Long-term stable public APIs
- A complex configuration or deployment story

Instead, it focuses on:

- A small, readable Go codebase
- Clear boundaries between infra and domain logic
- A reference implementation that is easy to fork and modify

### Clear Separation of Concerns

| Layer      | Responsibility              |
|-----------|-----------------------------|
| Watcher   | Blockchain IO (logs only)   |
| Processor | Security reasoning / rules  |
| Dispatcher| Output & delivery           |
| Registry  | Metadata, not logic         |

No rule logic, protocol heuristics, or risk scoring live inside infrastructure code.

### Reusable Watcher

The watcher SDK:

- Emits generic `LogEvent` domain events
- Has zero knowledge of:
  - Protocols
  - Rules
  - Severity or risk models

This enables:

- Multiple processors reading from the same raw stream
- Parallel experimentation on rules without touching ingestion
- Reuse of the same watcher SDK across different apps or deployments

### Explicit SDK Scope

The Asentric SDK is meant to:

- Be understandable end-to-end during a hackathon
- Be forked and adapted, rather than imported as a black box
- Serve as a reference architecture for logs-based security monitoring

---

## Core Components

### Watcher Service (Logs-Based)

The watcher is a pure infrastructure service built around a small SDK in `pkg/watcher`.

**Key pieces:**

- `EventSource` interface for emitting `LogEvent`
- `LogsSource` implementation using `SubscribeFilterLogs`
- `StreamEngine` optional wrapper to publish `LogEvent` into Redis Streams

**Responsibilities:**

- Connect to Mantle RPC over WebSocket
- Subscribe to logs using address/topic filters
- Normalize `types.Log` into `domain.LogEvent`
- Optionally publish `LogEvent` into `stream:raw` via Redis Streams

**Does not:**

- Decode business meaning of events
- Apply rule logic or make risk decisions
- Know anything about specific protocols

### Processor Service (Rule Engine)

The processor consumes normalized events (`LogEvent`) and applies security heuristics.

**Responsibilities:**

- Convert Redis stream messages back into `LogEvent`
- Resolve protocol metadata via the registry
- Apply rule logic implemented as Go interfaces
- Emit security alerts when rules are triggered

Rules operate purely on:

- `domain.LogEvent` (normalized on-chain event)
- Optional `registry.ContractMeta` (context)

### Dispatcher Service

The dispatcher consumes alerts and forwards them to external channels.

**Responsibilities:**

- Consume alerts from `stream:alerts`
- Format messages for human consumption
- Send to X / Webhook / logging targets (mocked for MVP)

The dispatcher is intentionally simple and focused on output mechanics.

### Protocol Registry

A YAML-based metadata registry used only for enrichment.

**Stores:**

- Contract address → protocol name / tags
- Known event signatures
- Risk-related metadata
- Human-readable descriptions

The registry is not an execution engine; logic remains in the processor and rule code.

---

## Rule Engine (MVP Scope)

Rules are implemented as Go structs that satisfy a small interface (see `pkg/rules`).

**MVP rules focus on:**

- Large withdrawals or high-value transfers
- Unknown contract interactions
- Role / ownership changes (e.g., `grantRole`, `transferOwnership`)
- Contract upgrades (proxy events)
- Sensitive event detection based on known selectors

Rules consume only:

- `domain.LogEvent` (normalized event)
- Optional `registry.ContractMeta` (context)

---

## Technology Stack

- **Language:** Go 1.22+
- **Blockchain:** Mantle Network (OP Stack)
- **Ethereum Client:** go-ethereum (via custom `pkg/client`)
- **Transport:** Redis Streams
- **Config:** YAML + environment variables (Viper)
- **Deployment:** Docker / Docker Compose (MVP)
- **Docs:** Separate `internal-docs` repository for internal architecture and operations

---

## Project Structure (Backend Repository)

Approximate layout of the backend repository:

```text
asentric/
├── apps/
│   ├── watcher/        # Watcher service (logs-based ingestion)
│   ├── processor/      # Rule engine & alert generation
│   └── dispatcher/     # Alert delivery
├── pkg/
│   ├── watcher/        # LogsSource, StreamEngine (watcher SDK)
│   ├── domain/         # LogEvent and converters
│   ├── rules/          # Rule interface and rule engine
│   ├── registry/       # Protocol registry abstractions
│   ├── alerts/         # Alert structures
│   ├── mq/             # Redis Streams wrapper
│   ├── client/         # Ethereum RPC client (SubscribeFilterLogs, etc.)
│   └── config/         # Service configuration loaders
├── configs/            # YAML configs for watcher / processor / dispatcher
├── deployments/        # Docker / deployment manifests
├── docs/               # Component-level docs (e.g. watcher implementation)
└── Makefile            # Build and dev tooling
```

A separate `internal-docs/` repository contains:

- High-level architecture and decision log
- SDK and rule-engine specifications
- Operations timeline and development guide
- Security notes and internal processes

---

## Current Status

**Hackathon MVP**

**Included:**

- Logs-based watcher SDK (`pkg/watcher`)
- `LogEvent` domain model (`pkg/domain`)
- Rule engine operating on log events (`pkg/rules`)
- Redis Streams pipeline (raw events and alerts)
- Basic alert dispatcher (X / Webhook mocked)
- YAML-based protocol registry

**Explicitly out of scope for MVP:**

- Production-ready UI dashboard
- Historical backfills and indexing jobs
- Multi-chain support
- Plugin / extension APIs
- Strong stability guarantees for public APIs

---

## Philosophy

This project prioritizes correctness, debuggability, and architectural clarity over feature completeness.

Asentric is meant to be:

- Understandable in hours, not weeks
- Easy to fork and adapt to new protocols or chains
- A clean starting point for more advanced production systems

---

## License

Asentric is released under the MIT License.  
See the `LICENSE` file in the backend repository for details.

---

## Contributing

Contributions are welcome.

- For backend changes, see the `asentric` repository and open a Pull Request with a clear description and semantic commit messages.
- For documentation and architecture updates, use the `internal-docs` repository and keep decision logs and diagrams in sync with code changes.

Please discuss major design changes in an issue or RFC-style document before opening a large PR.

---

## Contact

- Issues and feature requests: GitHub Issues in the corresponding repository
- For protocol integrations or deeper collaboration: reach out via the contact information listed in the organization profile or open an issue labeled `integration`.


