# Ticking the field: units, threads and determinism

Moved from the README on 2026-10-31, unchanged, when the README was brought to the [org documentation guide](https://github.com/bevy-coven/.github/blob/main/DOCUMENTATION.md).

## Units and time

The crate has no clock. `evaporate_diffuse(dt)` applies `ChannelDef::evaporate` **per unit of whatever `dt` you pass**: per second when you step in seconds, per game-day when you step in game-days. The same number means different things in each — `0.4` loses 40% of the field per second in the first and 40% per game-day in the second. `diffuse` is different: it is applied **per call**, not scaled by `dt`, so two calls at `dt/2` spread twice as much as one call at `dt`.

Diffusion moves value and never makes or loses any. Each call, every floor cell trades `diffuse / 4` of the difference with each floor neighbour — in open floor, a blend toward the neighbour average — and nothing across a wall, so with no deposits the floor total after `k` ticks is `(1 − evaporate·dt)^k` of the start, to float rounding, whether the scent sits in a hall or at the end of a dead-end corridor. Keep `diffuse` in `[0, 1]`; up to 1 no value goes negative.

## Which thread does the work

**Yours.** `evaporate_diffuse(dt)` runs inline on the thread that calls it and returns; the crate owns no thread pool and depends on none. A Bevy system that calls it shows up in a profile as compute on that worker, never as a wait on another pool's threads.

That is what `examples/bench.rs` checks in its `busy_pool` line: a 175×175 two-channel field ticked from a Bevy `App` system while every other `ComputeTaskPool` worker is busy. On a 16-core host at load average ≈65, 0.2.0 (which ran the stencil on rayon's pool) parked the system's thread on every tick, median 18.7–20.7 ms and max 97 ms; the inline tick parked on none of 6 000 ticks, median 0.25 ms, and p99 0.28–0.31 ms in five runs of six (the sixth, 1.9 ms, is the OS preempting a thread that never slept).

A large grid can spread the work over **your** pool instead. `evaporate_diffuse_with` hands you the pass as disjoint chunks and you run them wherever you like — here, Bevy's own `ComputeTaskPool`:

```rust,ignore
let pool = ComputeTaskPool::get();
field.evaporate_diffuse_with(dt, pool.thread_num(), |chunks| {
    pool.scope(|s| chunks.for_each(|chunk| s.spawn(async move { chunk.run() })));
});
```

`rayon::scope`, `std::thread::scope` or a plain loop work the same way. The result is bit-identical to `evaporate_diffuse` for any chunk count and any runner, and a chunk your runner drops instead of running is run inline afterwards, so a runner bug costs time, never a stale field. The spread costs a scope per diffusing channel, so it pays only on big grids: on the same host, quiet, one channel took 8 µs inline against 17 µs on the pool at 64×64, 229 µs against 88 µs at 256×256, and eight channels at 1024×1024 took 32 ms against 6 ms. While the pool is busy, the caller waits for whichever worker picked up a chunk; the inline call never waits for anyone.

## Determinism

Written to be bit-reproducible, because a simulation that hashes its state cannot afford otherwise: the neighbour sum keeps a fixed E/W/S/N order (float addition is non-associative); the diffusion is a pure stencil over disjoint output slots, so its result is identical however `evaporate_diffuse_with` splits it and on however many threads you run it; and skipping rock cells is exact rather than approximate. `channels()` and `cells()` expose the **full** grids so you can fold the rock-cells-stay-zero invariant into your own fingerprint.
