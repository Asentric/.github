# Asentric

**Mantle Security Realtime Alerting** - A lightweight, real-time smart contract security monitoring system built specifically for Mantle Network.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Go Version](https://img.shields.io/badge/Go-1.22+-00ADD8.svg)](https://golang.org/)

## Overview

Asentric is an open-source security monitoring system designed to provide real-time detection of suspicious on-chain activities on Mantle Network. The system monitors smart contract interactions, classifies known versus unknown protocol interactions, executes rule-based risk heuristics, and publishes actionable alerts through multiple channels.

The project serves both as a public security monitoring service and as a developer SDK, enabling protocol teams and independent developers to extend the rule engine, add custom protocol mappings, and deploy private monitoring instances.

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

```
Watcher (Go)
    ├─ Ingests blocks from Mantle RPC
    ├─ Filters smart contract interactions only
    ├─ Decodes calldata and extracts function selectors
    └─ Publishes raw events to Redis Streams

Processor (Go)
    ├─ Consumes events from Redis Streams
    ├─ Identifies protocols from YAML registry
    ├─ Applies configurable security rules
    ├─ Generates alerts when rules are triggered
    └─ Publishes alerts to alert stream

Dispatcher (Go)
    ├─ Consumes alerts from Redis Streams
    ├─ Formats alerts for different channels
    └─ Publishes to X/Twitter, Webhooks, etc.

Registry & SDK
    ├─ YAML-based protocol registry
    ├─ Auto-discovery for unknown protocols
    └─ Extensible rule engine interface
```

**Message Transport**: Redis Streams for lightweight, ordered event processing with support for consumer groups and horizontal scaling.

## Core Components

### Watcher Service

The ingestion layer that connects to Mantle RPC endpoints and monitors blockchain activity. It filters transactions to include only smart contract interactions, decodes calldata to identify function selectors, and publishes structured events to the message queue.

**Key Features**:
- Real-time block polling from Mantle L2
- Transaction filtering (contract interactions only)
- Calldata decoding and function selector extraction
- Configurable batch processing and polling intervals

### Processor Service

The rule engine that processes events from the watcher, identifies protocols through the registry, and evaluates security rules. When a rule is triggered, it generates an alert with appropriate severity levels.

**Key Features**:
- Protocol identification from YAML registry
- Configurable rule engine with priority-based evaluation
- Auto-discovery mechanism for unknown protocols
- Hot-reloadable configuration (registry and rules)

### Dispatcher Service

The alert publisher that consumes alerts from the processor and distributes them to configured channels. Supports multiple output formats and channels including social media and webhooks.

**Key Features**:
- Multi-channel alert distribution
- Configurable alert formatting
- Rate limiting and retry mechanisms
- Mock mode for development and testing

### Protocol Registry

A YAML-based registry that maps contract addresses to protocol information, including function selectors and their associated risk levels. Supports manual curation and auto-discovery of new protocols.

**Key Features**:
- Address-to-protocol mapping
- Function selector risk classification
- Metadata storage (website, description, etc.)
- Auto-discovery with manual approval workflow

### Rule Engine

A modular, extensible rule system that evaluates transaction contexts against configurable security heuristics. Rules can be enabled/disabled, prioritized, and extended through the SDK.

**Core Rules (MVP)**:
1. Large Withdrawal Detection - Monitors withdrawals above configurable thresholds
2. Unknown Protocol Detection - Flags interactions with unregistered contracts
3. Role Change Detection - Detects grantRole, revokeRole, and ownership transfers
4. Contract Upgrade Detection - Monitors upgradeTo and upgradeToAndCall calls
5. Sensitive Function Detection - Alerts on high-risk function calls based on registry

## Technology Stack

- **Language**: Go 1.22+
- **Blockchain**: Mantle Network (Ethereum L2)
- **Message Queue**: Redis Streams
- **Configuration**: YAML with Viper (environment variable support)
- **Ethereum Client**: go-ethereum
- **Deployment**: Docker, Docker Compose

## Project Structure

```
asentric/
├── apps/              # Service applications
│   ├── watcher/      # Blockchain ingestion service
│   ├── processor/    # Rule evaluation service
│   └── dispatcher/   # Alert publishing service
├── pkg/              # Public SDK packages
│   ├── config/       # Configuration management
│   ├── registry/     # Protocol registry
│   ├── rules/        # Rule engine
│   ├── alerts/       # Alert types
│   ├── mq/           # Message queue abstraction
│   ├── decoder/      # Calldata decoder
│   └── mantlenode/   # Mantle RPC client
├── internal/         # Private packages
├── sdk/             # Developer SDK examples
├── configs/         # YAML configuration files
├── scripts/         # Utility scripts
└── deployments/     # Docker and deployment configs
```

## Configuration

Asentric uses a hierarchical configuration system:

1. **Environment Variables** (highest priority) - Override all other sources
2. **YAML Configuration Files** - Service-specific settings in `configs/`
3. **Default Values** - Hardcoded fallbacks

Configuration files:
- `configs/watcher.yaml` - Watcher service configuration
- `configs/processor.yaml` - Processor service configuration
- `configs/dispatcher.yaml` - Dispatcher service configuration
- `configs/registry.yaml` - Protocol registry
- `configs/rules.yaml` - Rule configurations

## Developer SDK

Asentric provides a Go-based SDK that enables developers to:

- Implement custom detection rules
- Extend the protocol registry
- Deploy private monitoring instances
- Embed the rule engine into existing infrastructure

The SDK exposes clean interfaces for rule implementation, protocol registration, and event processing. See the main repository for examples and documentation.

## Use Cases

### Public Security Monitoring

Real-time alerts for high-risk on-chain activities, providing transparency for all Mantle Network users. Alerts are published to public channels (X/Twitter) and webhook endpoints.

### Protocol Team Monitoring

Protocol teams can deploy private instances with custom rules tailored to their specific contracts. Early warning system for incident detection and watchdog functionality during mainnet deployments.

### Security Research

Independent researchers can extend the heuristics engine, propose new detection rules, and build protocol-specific monitoring agents.

## Current Status

**MVP Phase** - The project is currently in MVP development with the following scope:

- Real-time watcher for Mantle testnet
- YAML-based protocol registry with auto-discovery
- Configurable rule engine with 5 core rules
- Alert dispatcher with mock implementations for Twitter and Webhooks
- Developer SDK with basic examples
- Docker-based deployment

**Roadmap**: See the main repository for planned features including protocol-specific rule packs, enhanced dashboard, risk scoring system, and multi-chain expansion.

## Contributing

Contributions are welcome. Areas where contributions are particularly valuable:

- New detection rules
- Protocol registry entries
- Performance optimizations
- Documentation improvements
- Bug fixes and testing

Please refer to the main repository for contributing guidelines and code of conduct.

## License

This project is licensed under the MIT License. See the LICENSE file in the main repository for details.

## Contact

- **Issues**: [GitHub Issues](https://github.com/asentric/asentric/issues)
- **Discussions**: [GitHub Discussions](https://github.com/asentric/asentric/discussions)

---

Built for the Mantle Network ecosystem by security researchers and protocol developers.
