# Asymptotic Optimality vs Real Performance

## Context

Sliding-window algorithm maintaining k smallest elements under a distance constraint.

Baseline implementations were already asymptotically optimal: O(n log k)

However, runtime performance varied dramatically depending on data representation and deletion strategy.

## Observation

The main bottlenecks were not algorithmic complexity but:

- data layout
- memory access patterns
- indirection level
- container design

## Design Changes

Transition from generic abstractions to specialised representations:

- value-based structures → index-based state
- hash maps → compact contiguous arrays
- generic containers → specialised heaps
- struct pairs → packed 64-bit keys
- lazy deletion via counters → index flags

## Results

Same algorithmic complexity.

Different performance characteristics.

C# implementation:
- ~550 ms
- ~75 MB RSS (actual data much smaller, but CLR overhead is real)

Initial Rust implementation:
- ~110 ms
- ~6.8 MB RSS

Optimised Rust implementation:
- ~60 ms
- ~4.7 MB RSS

## Key Insight

Asymptotic optimality does not imply real-world efficiency.

Memory layout and indirection frequently dominate performance.


## Implications

Relevant beyond algorithmic problems:

- low-latency services
- streaming pipelines
- infrastructure-level code
- performance-sensitive middleware
