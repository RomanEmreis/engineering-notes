# std Channels vs Tokio Channels in Rust

## Context

Rust offers multiple ways to communicate between concurrent tasks:

- **`std::sync::mpsc`** (and third-party variants like `crossbeam-channel`) for OS-thread-based concurrency.
- **`tokio::sync` channels** for async tasks scheduled by Tokio.

They look similar at the API level - send values, receive values, but they live in very different concurrency models. Mixing them casually can lead to performance problems, deadlocks, or accidental blocking inside an async runtime.

This note explains the differences, trade-offs, and how to choose.

## Mental Model

### std channels (thread world)
`std::sync::mpsc` is designed for **blocking** communication between **OS threads**.

- `recv()` blocks the current thread.
- `send()` may block in bounded variants (std is unbounded, so mostly “doesn’t block”, but can still be expensive under contention).
- Works with `std::thread`, CPU-bound workers, and synchronous code.

### tokio channels (async world)
Tokio channels are designed for **non-blocking** coordination between **async tasks**.

- `recv().await` yields the task to the runtime instead of blocking a thread.
- Backpressure is explicit via bounded channel capacity.
- Designed to work with cancellation, task scheduling, and async shutdown patterns.

## The Big Rule

**Never block a Tokio worker thread.**

If you call blocking `std` primitives (including `std::sync::mpsc::Receiver::recv`) directly inside an async task, you risk:

- starving the runtime (other tasks stop progressing),
- latency spikes,
- deadlocks in worst cases.

If you must use blocking calls inside async code, move them into:

- `tokio::task::spawn_blocking(...)` (for CPU/blocking IO), or
- a dedicated thread.

## Channel Types Overview

### `std::sync::mpsc`
- **Multi-producer, single-consumer**.
- **Unbounded** channel in stdlib.
- Receiver is not cloneable.
- Blocking receive (`recv`, `recv_timeout`).
- Simple and stable, but limited and not async-friendly.

### `tokio::sync::mpsc`
- **Multi-producer, single-consumer**.
- Bounded channel with capacity.
- `Sender` is cloneable.
- `Receiver::recv().await` is async and cancellation-aware.

### `tokio::sync::broadcast`
- **Multi-producer, multi-consumer** (each receiver gets every message).
- Bounded ring-buffer semantics.
- Receivers can lag and get `Lagged` errors (important to handle explicitly).
- Great for events, notifications, “pub-sub”.

### `tokio::sync::watch`
- “Last value wins”.
- Receivers always see the latest value, not every event.
- Great for config/state propagation and shutdown flags.

### `tokio::sync::oneshot`
- Single send, single receive.
- Great for request/response, completion signals.

## Backpressure and Memory Behavior

### std mpsc (unbounded)
Because it’s unbounded, the risk is:

- producers can outpace consumers,
- memory usage grows without a natural limit,
- latency grows because the queue grows.

This is often fine for small workloads, but it becomes dangerous for bursty or untrusted input.

### Tokio mpsc (bounded)
Tokio mpsc is typically bounded and will apply backpressure:

- When the buffer is full, `send().await` waits until there is capacity.
- This helps keep memory stable and forces the system to self-regulate.

If you need “fire and forget”, Tokio also provides `try_send()` patterns, but you must decide what to do on overflow.

## Blocking vs Async Waiting: Why It Matters

In a Tokio runtime, the runtime uses a limited number of worker threads to drive many tasks.

- Blocking in a task blocks the whole worker thread.
- Async waiting yields execution to other tasks.

So:

- **std channels are correct** when your concurrency is “thread-per-worker”.
- **tokio channels are correct** when your concurrency is “many async tasks on few threads”.

## When std Channels Are Still a Great Choice

Use std channels (or `crossbeam-channel`) if:

- You are building a **synchronous** library.
- You are using **std threads** for CPU-bound parallelism.
- You want “classic pipeline” style concurrency without async.
- You are integrating with a non-async ecosystem.

In high-performance synchronous Rust, `crossbeam-channel` is often preferred over `std::sync::mpsc` due to better performance and features (bounded + select).

## When Tokio Channels Are the Right Tool

Use Tokio channels when:

- Communication happens between Tokio tasks.
- You need `.await`-based receiving.
- You want bounded queues + backpressure.
- You care about async cancellation, graceful shutdown.
- You want structured patterns like request/response, broadcast, watch.

If your code is already async, Tokio channels keep you in the same model.

## Bridging Between std and Tokio Worlds

Sometimes you must integrate sync and async components.

### Pattern A: Dedicated thread → Tokio channel
- Sync component runs in a dedicated thread.
- It sends messages into `tokio::mpsc::Sender`.

This is common for:
- reading from blocking sources,
- stdin/stdout bridges,
- legacy libraries.

### Pattern B: Tokio task → std channel (rare)
Usually avoid. If you must do it, don’t call blocking `recv()` in async context.
Instead:
- use `spawn_blocking` or a thread to receive, then forward into Tokio.

### Pattern C: Use `tokio::sync::mpsc` everywhere + `spawn_blocking` at edges
This is often the cleanest approach in async applications:
- async core uses Tokio channels,
- blocking edges are isolated.

## Shutdown Semantics

### std channels
Shutdown is “implicit”:
- if all senders drop, receiver eventually gets `Err(Disconnected)`.

### Tokio channels
Similar principle, but more structured:
- `recv().await` returns `None` when all senders are dropped.
- `broadcast` has explicit lag handling.
- `watch` can signal change and also detect sender drop.

**Engineering tip:** Use explicit shutdown signals (`watch` or `broadcast`) rather than relying solely on “drop closes channel” when you need predictable shutdown ordering.

## Performance Notes (Practical)

- Tokio channels integrate with the runtime scheduler and are tuned for async workloads.
- std channels are simpler but can become bottlenecks under contention.
- For high-throughput sync pipelines, consider `crossbeam-channel`.
- For high-throughput async pipelines, keep channels bounded and prefer batching when possible.

Avoid designing systems where:
- a single receiver becomes a “hot spot” bottleneck,
- messages are extremely small and extremely frequent (consider batching or different primitives).

## Quick Decision Table

| Need | Best Tool |
|------|-----------|
| Pure sync + threads | `std::sync::mpsc` or `crossbeam-channel` |
| Async tasks + `.await` receive | `tokio::sync::mpsc` |
| 1-to-many event stream | `tokio::sync::broadcast` |
| Share latest state/config | `tokio::sync::watch` |
| Single response signal | `tokio::sync::oneshot` |
| Bounded queue + backpressure | `tokio::sync::mpsc` (bounded) |

## Final Takeaway

The difference isn’t “which channel is better” — it’s which concurrency model you are in:

- **Threads** → blocking primitives (std/crossbeam)
- **Async runtime** → async primitives (Tokio)

The fastest way to introduce subtle production issues in async Rust is to accidentally use blocking channels in the hot path. Keep blocking at the edges, and use Tokio-native channels for the async core.
