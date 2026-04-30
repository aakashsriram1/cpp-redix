# Redix

Redis-compatible in-memory data store built from scratch in C++17.

Redix is an architectural study of the systems ideas behind Redis: a single-threaded non-blocking TCP server, a hand-written RESP protocol parser, custom cache-conscious data structures, TTL expiry, hashes, sorted sets, and performance measurement. The goal is not to copy every Redis feature. The goal is to implement the core mechanics that make an in-memory data store fast, predictable, and easy to reason about under load.

## Highlights

- C++17 systems programming project with Linux networking primitives
- Single-threaded, non-blocking TCP server built around `epoll`
- RESP parser with partial read handling and pipelined command support
- Compatible with real `redis-cli` connections
- Core Redis-style commands: `PING`, `SET`, `GET`, `DEL`, `EXISTS`, `EXPIRE`, `TTL`
- Custom Robin Hood hash table instead of `std::unordered_map`
- Incremental rehashing to avoid resize-driven latency spikes
- Ziplist-style compact encoding for small hashes
- Skip list implementation for sorted sets
- Benchmarking with `redis-benchmark`
- Profiling workflow using `perf`, flamegraphs, cache-miss analysis, and p50/p99 latency tracking

## Architecture

### Event Loop

Redix uses a single-threaded, non-blocking TCP server backed by Linux `epoll`. The server accepts client sockets, reads available data without blocking, writes buffered responses when sockets become writable, and executes commands on one thread to avoid lock contention in the core data path.

### RESP Parser

The protocol layer implements a stateful RESP parser by hand. It handles partial reads, multiple commands in the same buffer, and pipelined requests from Redis clients. This keeps parsing explicit and makes network behavior easy to inspect during profiling.

### Storage Engine

Keys are stored in custom cache-conscious structures designed for predictable access patterns. Expiration metadata is tracked alongside values so commands such as `EXPIRE` and `TTL` can be served without a separate external scheduler.

### Data Structures

- Robin Hood hash table for key lookup with tighter probe behavior than a naive hash map
- Incremental rehashing so large table growth does not create long pause times
- Ziplist-style encoding for small hashes to reduce allocation overhead and improve locality
- Skip lists for sorted sets, matching the core ordered-index idea used by Redis

## Supported Commands

| Category | Commands |
| --- | --- |
| Connection | `PING` |
| Strings | `SET`, `GET` |
| Keys | `DEL`, `EXISTS`, `EXPIRE`, `TTL` |
| Hashes | TODO: list supported hash commands |
| Sorted sets | TODO: list supported sorted set commands |

## Performance And Benchmarking

Redix is designed to be measured with Redis-compatible tooling such as `redis-benchmark`.

TODO: insert benchmark results after running on a fixed machine and build profile.

| Workload | Requests | Clients | p50 Latency | p99 Latency | Throughput |
| --- | ---: | ---: | ---: | ---: | ---: |
| `PING` | TODO | TODO | TODO | TODO | TODO |
| `SET` | TODO | TODO | TODO | TODO | TODO |
| `GET` | TODO | TODO | TODO | TODO | TODO |

## Profiling

The project includes a profiling-oriented workflow for understanding where time is spent and how memory layout affects latency.

TODO: insert flamegraph image and `perf stat` output.

| Metric | Result |
| --- | ---: |
| CPU cycles | TODO |
| Instructions per cycle | TODO |
| L1 data cache misses | TODO |
| Branch misses | TODO |
| p50 latency | TODO |
| p99 latency | TODO |

## Design Decisions

### Why `epoll`

`epoll` provides scalable readiness notifications for many sockets without one thread per client. It is the natural Linux primitive for studying high-throughput network servers.

### Why Single-Threaded Command Execution

Single-threaded execution keeps the command path deterministic and removes locks from the storage engine. This mirrors Redis's core design tradeoff: simplify concurrency so latency is easier to reason about.

### Why Robin Hood Hashing

Robin Hood hashing reduces probe length variance by moving entries with longer probe distances forward. That makes lookup behavior more predictable and provides a useful contrast with general-purpose hash maps.

### Why Incremental Rehashing

Growing a hash table all at once can create visible tail-latency spikes. Incremental rehashing spreads migration work across future operations so resizing does not dominate a single request.

### Why Ziplist-Style Encoding

Small hashes often have few fields. A compact contiguous representation reduces pointer chasing and allocation overhead, improving cache locality for small objects.

### Why Skip Lists For Sorted Sets

Skip lists provide ordered operations with simple implementation mechanics and expected logarithmic performance. They are a practical fit for sorted-set style range queries.

## Build And Run

```bash
cmake -S . -B build
cmake --build build
./build/redix --port 6379
```

If the project is built with a different command, update this section with the exact local build target.

## Example Usage

```bash
redis-cli -p 6379
127.0.0.1:6379> PING
PONG
127.0.0.1:6379> SET symbol AAPL
OK
127.0.0.1:6379> GET symbol
"AAPL"
127.0.0.1:6379> EXPIRE symbol 30
(integer) 1
127.0.0.1:6379> TTL symbol
(integer) 30
```

## Roadmap

- Fill out hash and sorted-set command coverage
- Add persistence experiments such as append-only logging
- Expand benchmark suite across mixed workloads
- Add CI for parser, storage, and command tests
- Publish flamegraphs and cache-miss analysis from repeatable benchmark runs
