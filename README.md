# ternary-life

**Game of Life with a lifecycle: young, old, dead. Aging as a rule, not a bug.**

Conway's Game of Life has two states: alive and dead. Birth requires exactly 3 neighbors. Survival requires 2 or 3. That's it — and from those rules, all complexity emerges.

But what if being alive isn't a single state? What if cells *age*? This crate implements a ternary Game of Life where cells progress through a lifecycle: **0 = dead → 1 = young → -1 = old → 0 = dead**. Young cells survive with 2-3 neighbors (standard). Old cells survive with only 1-2 neighbors (harder to kill — experience matters). But old cells can't give birth — only fresh young cells emerge from empty space.

This single change — adding aging — creates fundamentally different dynamics. The standard Game of Life has still lifes and oscillators. The ternary lifecycle has *generational turnover*: populations boom with young cells, mature into old cells, then die back, creating cyclic population waves with period ~15.

## What's Inside

- **`LifeGrid`** — the grid: `cells: Vec<i8>`, `width: usize`. Dead simple storage
- **`tick()`** — one generation. Rules:
  - **Dead (0) + 3 neighbors → Young (1)**: birth as before
  - **Young (1) + 2-3 neighbors → Old (-1)**: survive but age
  - **Old (-1) + 1-2 neighbors → Old (-1)**: survive (experienced cells are resilient)
  - **Old (-1) + anything else → Dead (0)**: death from overcrowding or loneliness
  - **Young (1) + <2 or >3 neighbors → Dead (0)**: standard death
- **`census()`** — count young, old, dead cells
- **`is_stable()`** — has the grid stopped changing?
- **`find_oscillators()`** — detect periodic patterns (period ≤ 20)

## Quick Example

```rust
use ternary_life::*;

let mut grid = LifeGrid::new(20, 20);

// Place a "seed" — a cluster of young cells
grid.set(10, 10, 1); // young
grid.set(11, 10, 1);
grid.set(12, 10, 1);

// Run the lifecycle
for _ in 0..100 {
    grid.tick();
    let (young, old, dead) = grid.census();
    // Watch: young boom → old accumulate → die-off → young boom again
}

// The lifecycle IS a built-in clock
// Period ≈ 15 ticks from boom to bust to boom
```

## The Deeper Truth

**Aging creates a built-in clock.** In standard Life, patterns can be immortal (gliders, still lifes). In ternary Life, everything ages. Young becomes old, old dies. This means:

1. **No immortal static patterns** — even the most stable structure eventually ages out
2. **Cyclic dynamics** — the boom/bust lifecycle creates periodic population waves
3. **Different survival rules for different ages** — old cells are *more* resilient (1-2 neighbors) but young cells are the *only* ones that emerge from birth
4. **Generational turnover** — the population is always renewing, never stagnating

This mirrors biological reality more faithfully than standard Life. In real ecosystems, aging isn't a failure — it's the mechanism that prevents stagnation and enables renewal.

**Use cases:**
- **Artificial life** — study how aging affects ecosystem dynamics
- **Generative art** — lifecycle patterns create distinctive visual textures
- **Education** — compare binary Life vs. ternary Life to see how small rule changes create large effects
- **Population dynamics** — boom/bust cycles with generational turnover
- **Cellular automata research** — ternary Life as a bridge between binary CA and continuous models

## See Also

- **ternary-fire** — forest fire model (another lifecycle: tree → burning → empty → tree)
- **ternary-ising** — ternary lattice dynamics (no lifecycle, different physics)
- **ternary-sandpile** — self-organized criticality (avalanche dynamics)
- **ternary-drift** — population-level dynamics without spatial structure

## Install

```bash
cargo add ternary-life
```

## License

MIT
