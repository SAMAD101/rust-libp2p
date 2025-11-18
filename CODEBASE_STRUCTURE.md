# rust-libp2p Codebase Structure

This document provides a comprehensive overview of the rust-libp2p codebase structure, its architecture, and development workflows.

## Table of Contents

1. [Overview](#overview)
2. [Repository Structure](#repository-structure)
3. [Core Components](#core-components)
4. [Architecture Patterns](#architecture-patterns)
5. [Development Workflow](#development-workflow)
6. [Testing Strategy](#testing-strategy)
7. [Contributing](#contributing)

---

## Overview

**rust-libp2p** is the Rust implementation of the [libp2p](https://libp2p.io) networking stack, a modular peer-to-peer networking framework. The codebase is organized as a Cargo workspace with multiple independent crates that work together to provide a complete P2P networking solution.

**Key Characteristics:**
- **Modular Design**: Each component (transport, protocol, muxer) is a separate crate
- **Feature-Based**: Components can be selectively included via Cargo features
- **Workspace-Based**: Uses Cargo workspace for managing multiple related packages
- **Hierarchical State Machines**: Core architecture pattern used throughout

---

## Repository Structure

The repository is organized into the following top-level directories:

```
rust-libp2p/
├── core/                          # libp2p-core - Core traits and types
├── swarm/                         # libp2p-swarm - Network behavior orchestration
├── protocols/                     # Application-layer protocols
├── transports/                    # Transport layer implementations
├── muxers/                        # Stream multiplexing protocols
├── misc/                          # Utility crates
├── identity/                      # libp2p-identity - Cryptographic identities
├── libp2p/                        # Main libp2p crate (aggregates all components)
├── examples/                      # Working examples and tutorials
├── docs/                          # Architecture and development documentation
├── scripts/                       # Build and release automation scripts
├── interop-tests/                 # Cross-implementation interoperability tests
├── hole-punching-tests/           # NAT traversal testing
├── wasm-tests/                    # WebAssembly-specific tests
└── .github/                       # CI/CD workflows and automation
```

### Directory Details

#### 1. **core/** - libp2p-core
The foundational crate that defines core abstractions:
- **`Transport` trait**: Abstraction for network transports (TCP, QUIC, WebSocket, etc.)
- **`StreamMuxer` trait**: Interface for multiplexing multiple streams over a single connection
- **Upgrade system**: Protocol negotiation using multistream-select
- **Core types**: `PeerId`, `Multiaddr`, connection handling

**Key Files:**
- `src/transport.rs` - Transport trait and combinators
- `src/muxing.rs` - Stream multiplexing interface
- `src/upgrade.rs` - Protocol upgrade mechanism
- `src/connection.rs` - Connection management

#### 2. **swarm/** - libp2p-swarm
Manages the peer-to-peer network behavior:
- **`Swarm`**: Central orchestrator that manages connections and behaviors
- **`NetworkBehaviour` trait**: Interface for implementing application protocols
- **`ConnectionHandler` trait**: Per-connection protocol handling
- **Event system**: Communication between behaviors and the swarm

**Key Files:**
- `src/lib.rs` - Swarm implementation
- `src/behaviour.rs` - NetworkBehaviour trait and helpers
- `src/handler.rs` - ConnectionHandler trait
- `src/connection.rs` - Connection pool management

#### 3. **protocols/** - Application Protocols
Independent protocol implementations built on libp2p-swarm:

| Protocol | Description |
|----------|-------------|
| **autonat** | NAT and firewall detection for network reachability |
| **dcutr** | Direct connection upgrade through relay (hole punching) |
| **floodsub** | Simple pub-sub protocol with message flooding |
| **gossipsub** | Efficient pub-sub with mesh networking and gossip |
| **identify** | Peer identification and capability discovery |
| **kad** | Kademlia DHT for distributed hash table and peer routing |
| **mdns** | Local network peer discovery using multicast DNS |
| **perf** | Performance testing and benchmarking protocol |
| **ping** | Liveness check and latency measurement |
| **relay** | Circuit relay for connection proxying |
| **rendezvous** | Rendezvous point for peer discovery |
| **request-response** | Generic request-response pattern framework |
| **stream** | Generic stream protocols |
| **upnp** | UPnP port mapping for NAT traversal |

#### 4. **transports/** - Transport Implementations
Network transport layer protocols:

| Transport | Description |
|-----------|-------------|
| **tcp** | TCP/IP transport (most common) |
| **quic** | QUIC transport with built-in encryption (UDP-based) |
| **websocket** | WebSocket transport (for browsers and HTTP-compatible networks) |
| **websocket-websys** | WebSocket for WebAssembly (browser) environments |
| **webrtc** | WebRTC transport for browser-to-server communication |
| **webrtc-websys** | WebRTC for WebAssembly environments |
| **webtransport-websys** | WebTransport for WebAssembly |
| **uds** | Unix Domain Sockets (local inter-process communication) |
| **dns** | DNS resolution for multiaddrs |
| **noise** | Noise protocol framework for encrypted connections |
| **tls** | TLS encryption for transports |
| **plaintext** | Unencrypted transport (testing/development only) |
| **pnet** | Private network support with pre-shared keys |

#### 5. **muxers/** - Stream Multiplexers
Protocols for multiplexing multiple streams over a single connection:

| Muxer | Description |
|-------|-------------|
| **yamux** | Yamux multiplexing protocol (recommended, efficient) |
| **mplex** | Mplex multiplexing protocol (legacy support) |
| **test-harness** | Testing utilities for muxer implementations |

#### 6. **misc/** - Utility Crates
Supporting libraries and tools:

| Crate | Purpose |
|-------|---------|
| **allow-block-list** | Connection allow/block list management |
| **connection-limits** | Connection count limits |
| **memory-connection-limits** | Memory-based connection limits |
| **metrics** | Prometheus metrics collection |
| **multistream-select** | Protocol negotiation implementation |
| **peer-store** | Peer information storage |
| **quick-protobuf-codec** | Async protobuf encoding/decoding |
| **quickcheck-ext** | QuickCheck property testing extensions |
| **rw-stream-sink** | Stream/Sink to AsyncRead/AsyncWrite adapter |
| **server** | rust-libp2p server binary |
| **webrtc-utils** | WebRTC utilities |
| **keygen** | Key generation tool |

#### 7. **libp2p/** - Main Aggregator Crate
The main `libp2p` crate that:
- Re-exports all protocol and transport implementations
- Provides feature flags for selective compilation
- Offers a unified API surface
- Documentation hub for the entire project

**Usage Pattern:**
```rust
// Add only what you need via features
[dependencies]
libp2p = { version = "0.56", features = ["tcp", "noise", "yamux", "gossipsub", "mdns"] }
```

#### 8. **examples/** - Working Examples
Practical examples demonstrating libp2p usage:

| Example | Demonstrates |
|---------|--------------|
| **ping** | Basic connectivity and the ping protocol |
| **chat** | Simple chat app with mDNS discovery and Gossipsub |
| **distributed-key-value-store** | DHT using Kademlia |
| **file-sharing** | File sharing with Kademlia and request-response |
| **ipfs-kad** | Connecting to the IPFS Kademlia network |
| **identify** | Peer identification protocol |
| **autonat** | NAT detection and autonat protocol |
| **dcutr** | Hole punching through relays |
| **relay-server** | Running a circuit relay server |
| **rendezvous** | Rendezvous protocol for discovery |
| **browser-webrtc** | WebRTC in browser environments |
| **metrics** | Prometheus metrics collection |
| **stream** | Generic stream protocols |
| **upnp** | UPnP port mapping |

#### 9. **docs/** - Documentation
Architecture and development documentation:
- `architecture.svg` - Visual architecture diagram
- `coding-guidelines.md` - Coding standards and patterns
- `maintainer-handbook.md` - Maintainer processes
- `release.md` - Release process documentation

---

## Core Components

### The Transport Stack

libp2p uses a layered transport architecture:

```
┌─────────────────────────────────────┐
│   Application Protocol              │  (Gossipsub, Kademlia, etc.)
├─────────────────────────────────────┤
│   NetworkBehaviour / Swarm          │  (Connection management, routing)
├─────────────────────────────────────┤
│   Stream Multiplexing               │  (Yamux, Mplex)
├─────────────────────────────────────┤
│   Security / Encryption             │  (Noise, TLS)
├─────────────────────────────────────┤
│   Base Transport                    │  (TCP, QUIC, WebSocket)
└─────────────────────────────────────┘
```

**Typical Transport Configuration:**
```rust
let transport = tcp::tokio::Transport::new(tcp::Config::default())
    .upgrade(upgrade::Version::V1)
    .authenticate(noise::Config::new(&keypair)?)
    .multiplex(yamux::Config::default())
    .boxed();
```

### The Swarm Architecture

The `Swarm` is the central component that:
1. Manages the connection pool
2. Drives the transport (accepting/dialing connections)
3. Polls all `NetworkBehaviour` implementations
4. Routes events between behaviors and connections

```rust
// Example Swarm setup
let swarm = SwarmBuilder::with_existing_identity(keypair)
    .with_tokio()
    .with_tcp(/* config */)
    .with_noise()
    .with_yamux()
    .with_behaviour(|key| MyBehaviour::new(key))
    .build();
```

### NetworkBehaviour Trait

The `NetworkBehaviour` trait is how you implement application-level protocols:
- Receives events from the swarm (new connections, addresses, etc.)
- Can instruct the swarm to dial peers, send events, etc.
- Multiple behaviors can be combined using the `#[derive(NetworkBehaviour)]` macro

```rust
#[derive(NetworkBehaviour)]
struct MyBehaviour {
    gossipsub: gossipsub::Behaviour,
    mdns: mdns::Behaviour,
    kad: kad::Behaviour,
}
```

---

## Architecture Patterns

### 1. Hierarchical State Machines

The entire codebase follows a hierarchical state machine pattern:
- Parents poll children in a loop
- Children return `Poll::Ready` when they make progress
- Children return `Poll::Pending` when they can't make progress (and register wakers)
- Events flow up (children to parents), commands flow down (parents to children)

**Why?** This pattern:
- Works naturally with Rust's `Future` model
- Makes control flow explicit and traceable
- Allows fine-grained control over execution order
- Prevents accidental concurrency bugs

### 2. Bounded Resources

**Everything is bounded** to prevent DoS attacks and memory leaks:
- Channels are bounded (never use unbounded channels)
- Local queues have maximum sizes
- Number of connections is limited
- Number of concurrent tasks is limited

### 3. Prioritization

When handling multiple work streams, the pattern is:
1. **First priority**: Return completed local work
2. **Second priority**: Finish pending local work
3. **Last priority**: Accept new work from remote

This ensures low latency and prevents remote peers from overwhelming local buffers.

### 4. Sequential by Default

- Keep things sequential unless proven to be a bottleneck
- Use `async/await` for sequential execution only
- Use manual `poll` implementations for concurrent operations
- Avoid spawning tasks unless necessary

### 5. Communication Patterns

- **Prefer channels over mutexes** ("share memory by communicating")
- Single ownership over data
- Event-driven communication between components

---

## Development Workflow

### Building the Project

```bash
# Build all workspace crates
cargo build --workspace

# Build with all features
cargo build --workspace --all-features

# Build specific crate
cargo build -p libp2p-swarm

# Build examples
cargo build --examples
```

### Running Tests

```bash
# Run all tests
cargo test --workspace

# Run tests for specific crate
cargo test -p libp2p-core

# Run tests with all features
cargo test --workspace --all-features

# Run a specific test
cargo test -p libp2p-swarm test_name
```

### Linting and Formatting

```bash
# Format code
cargo fmt --all

# Check formatting
cargo fmt --all -- --check

# Run clippy
cargo clippy --workspace --all-features

# Check for common issues
cargo clippy --workspace --all-features -- -D warnings
```

### Running Examples

```bash
# Navigate to example directory
cd examples/ping

# Run the example
cargo run

# Or run from workspace root
cargo run --example ping
```

### Documentation

```bash
# Build documentation
cargo doc --workspace --all-features --no-deps

# Build and open documentation
cargo doc --workspace --all-features --no-deps --open
```

---

## Testing Strategy

### Unit Tests
- Located alongside source code in `src/` directories
- Test individual components in isolation
- Use `#[cfg(test)]` modules

### Integration Tests
- Located in `tests/` directories within each crate
- Test interactions between components
- Use realistic scenarios

### Property-Based Testing
- Uses QuickCheck for property-based testing
- Located in `misc/quickcheck-ext/`
- Generates random test cases to find edge cases

### Interoperability Tests
- Located in `interop-tests/`
- Tests compatibility with other libp2p implementations (Go, JavaScript, etc.)
- Runs in CI against other implementations

### WebAssembly Tests
- Located in `wasm-tests/`
- Tests WASM-specific transports and behaviors
- Ensures browser compatibility

### Continuous Integration

CI workflows (in `.github/workflows/`):
- **ci.yml**: Main CI pipeline (build, test, lint)
- **interop-test.yml**: Cross-implementation testing
- **cargo-audit.yml**: Security vulnerability scanning
- **docs.yml**: Documentation building and hosting
- **docker-image.yml**: Docker image publishing
- **semantic-pull-request.yml**: PR title validation

---

## Contributing

### Workflow
1. **Squash-merge policy**: All PRs are squash-merged for clean history
2. **Conventional commits**: PR titles follow [conventional commit spec](https://www.conventionalcommits.org/)
3. **Changelog entries**: User-facing changes require changelog updates
4. **Automated merging**: Use "send-it" label when PR is ready

### Code Guidelines

1. **Use hierarchical state machines** - Follow the established pattern
2. **Bound everything** - No unbounded channels, queues, or task spawning
3. **Sequential by default** - Only make things concurrent when proven necessary
4. **Use iteration, not recursion** - Rust doesn't guarantee tail call optimization
5. **Correlate async responses** - Include request IDs in responses
6. **Prioritize local work** - Drain local queues before accepting remote work

See [`docs/coding-guidelines.md`](docs/coding-guidelines.md) for complete guidelines.

### Pull Request Process

1. Fork the repository
2. Create a feature branch
3. Make your changes following coding guidelines
4. Add tests for new functionality
5. Update changelog if user-facing changes
6. Ensure CI passes (treat it as self-service)
7. Request review
8. Apply "send-it" label when ready (no more commits after this!)

### Release Process

Releases are managed through `cargo-release`:
- Version bumping is coordinated across workspace
- Changelog headers are added automatically
- See `docs/release.md` for details

---

## Key Dependencies

### Internal Workspace Dependencies
- All crates reference each other via workspace dependencies
- `libp2p-identity` is patched at workspace level to ensure type compatibility
- Cyclic dev-dependencies use `path` references

### External Dependencies
- **multiaddr**: Multiaddress parsing and handling
- **multihash**: Multihash implementation
- **futures**: Async runtime abstractions
- **tokio**: Async runtime (optional, feature-gated)
- **quick-protobuf**: Protobuf encoding/decoding
- **prometheus-client**: Metrics collection

---

## Version Information

- **Rust Version**: 1.83.0 (minimum required)
- **Edition**: Rust 2021
- **Workspace Resolver**: Version 2
- **Current Version**: See individual crate `Cargo.toml` files

---

## Resources

- **Documentation**: https://docs.rs/libp2p
- **libp2p Specs**: https://github.com/libp2p/specs
- **Community Discussions**: https://github.com/libp2p/rust-libp2p/discussions
- **General libp2p Forum**: https://discuss.libp2p.io
- **Issue Tracker**: https://github.com/libp2p/rust-libp2p/issues

---

## Notable Users

rust-libp2p is used in production by major projects including:
- **Polkadot/Substrate** - Blockchain framework
- **Lighthouse** - Ethereum consensus client
- **IPFS implementations** - Distributed file systems
- **Filecoin** - Decentralized storage network
- Many others - see README.md for complete list

---

## License

MIT License - See [LICENSE](LICENSE) file for details

---

**Last Updated**: 2025-11-18  
**Maintained By**: libp2p maintainers  
**Primary Maintainer**: [@jxs](https://github.com/jxs)
