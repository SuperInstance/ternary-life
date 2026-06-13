# ternary-life

Ternary Game of Life — a three-state extension of Conway's cellular automaton where cells have **lifecycle stages**: **0 = dead, 1 = young, −1 = old**. Young and old cells follow different survival rules, creating richer dynamics than the classical binary Game of Life.

## Why It Matters

Conway's Game of Life (1970) uses binary cells: alive or dead. The ternary extension adds an **age dimension** within living cells, creating emergent behaviors impossible in the binary version:

- **Multi-generation stability** — old cells have different survival thresholds, enabling structures that persist across age transitions
- **Richer oscillator space** — ternary rules permit oscillator periods inaccessible to binary Life
- **Natural aging model** — populations exhibit lifecycle dynamics (birth → youth → age → death)
- **Compact encoding** — each cell is a single signed byte, directly compatible with ternary ecosystem arithmetic

The ternary rule set creates a **phase transition** in the complexity of emergent patterns. Classical Life has one survival rule for all living cells; ternary Life has two (young vs. old), opening a vastly larger rule space.

## How It Works

### Update Rules

The ternary Life transition function:

| Current state | Neighbor count (alive) | Next state | Explanation |
|---------------|----------------------|------------|-------------|
| Dead (0) | Exactly 3 | Young (1) | Birth |
| Dead (0) | ≠ 3 | Dead (0) | No birth |
| Young (1) | 2 or 3 | Old (−1) | Survives, ages |
| Young (1) | <2 or >3 | Dead (0) | Dies (loneliness/overcrowding) |
| Old (−1) | 1 or 2 | Old (−1) | Survives (resilient) |
| Old (−1) | <1 or >2 | Dead (0) | Dies |

Key differences from classical Life:
- **Old cells are more resilient to loneliness** — they survive with just 1 neighbor (vs. 2 needed for young cells). This models how established structures resist dissolution.
- **Old cells are less tolerant of overcrowding** — they die with 3+ neighbors (vs. 3 being fine for young cells). This models how aging populations can't handle density pressure.
- **Birth always produces young cells** — there's no direct birth-to-old transition.

### Neighbor Counting

An "alive" neighbor is any cell with value 1 (young) or −1 (old):

```
N(x, y) = |{(x', y') ∈ Moore(x, y) : cell(x', y') ∈ {1, −1}}|
```

Moore neighborhood = 8 surrounding cells (3×3 grid minus center). Boundary cells have fewer neighbors (non-wrapping).

### Stability and Oscillation

A grid is **stable** if one tick produces the identical configuration:

```
stable ⟺ grid == tick(grid)
```

An **oscillator** with period p satisfies:

```
grid == tick^p(grid)  ∧  grid ≠ tick^k(grid)  for k < p
```

The crate's `find_oscillators` function searches for periods 1 through 20.

### Complexity

| Operation | Time | Space |
|-----------|------|-------|
| `new(w, h)` | O(w·h) | O(w·h) |
| `tick()` | O(w·h) | O(w·h) |
| `census()` | O(w·h) | O(1) |
| `is_stable()` | O(w·h) | O(w·h) |
| `find_oscillators()` | O(p · w · h) | O(w · h) |
| `count_alive_neighbors(x, y)` | O(1) | O(1) |

Where w × h = grid dimensions, p = max period checked.

### Census

The census function returns population counts:

```
(young_count, old_count, dead_count)
```

The age distribution (young vs. old ratio) is a useful diagnostic: a healthy oscillating system cycles through different ratios, while a static system has a fixed ratio.

## Quick Start

```rust
use ternary_life::LifeGrid;

let mut grid = LifeGrid::new(20, 20);

// Seed with a glider pattern (using young cells)
grid.set(1, 2, 1);
grid.set(2, 3, 1);
grid.set(3, 1, 1);
grid.set(3, 2, 1);
grid.set(3, 3, 1);

// Evolve 50 generations
for generation in 0..50 {
    grid.tick();
    let (young, old, dead) = grid.census();
    println!("Gen {}: young={}, old={}, dead={}", generation, young, old, dead);
}

// Check if the system has stabilized
if grid.is_stable() {
    println!("System has reached stability");
}

// Search for oscillating behavior
let periods = grid.find_oscillators();
if !periods.is_empty() {
    println!("Oscillator detected with period {}", periods[0]);
}
```

## API

### `LifeGrid`

| Method | Description |
|--------|-------------|
| `new(width, height)` | Create all-dead grid |
| `set(x, y, val)` | Set cell value (0, 1, or −1) |
| `tick()` | Advance one generation |
| `census() -> (usize, usize, usize)` | Count (young, old, dead) |
| `is_stable() -> bool` | Check if grid is fixed point |
| `find_oscillators() -> Vec<usize>` | Detect oscillation periods ≤ 20 |

## Architecture Notes

This crate implements **η (eta) layer** simulation in the γ + η = C framework:

- **η (eta)**: The cellular automaton engine — state transitions, neighbor counting, census. This crate provides η-layer ternary Life dynamics.
- **γ (gamma)**: External coordination — multi-grid synchronization, parallel evaluation of large grids, checkpoint/restore. Provided by ecosystem crates.
- **C**: The complete ternary CA system. The cell states {−1, 0, +1} map directly to ternary weights, spins, and marks used throughout the ecosystem, making Life patterns analyzable with the same algebraic tools.

## References

- **Conway's Game of Life**: Gardner, M., "Mathematical Games: The Fantastic Combinations of John Conway's New Solitaire Game 'Life'," Scientific American, 223(4), 120-123, 1970.
- **Three-State Cellular Automata**: Wolfram, S., "Universality and Complexity in Cellular Automata," Physica D, 10(1-2), 1-35, 1984.
- **Aging in CAs**: Ermentrout, G.B. & Edelstein-Keshet, L., "Cellular Automata Approaches to Biological Modeling," Journal of Theoretical Biology, 160(1), 97-133, 1993.
- **Moore Neighborhood**: Moore, E.F., "Machine Models of Self-Reproduction," Proceedings of Symposia in Applied Mathematics, 14, 17-33, 1962.
- **Oscillators in Life**: Flammenkamp, A., "Spaceships and Oscillators in Life-like Cellular Automata," 2003. Database of known CA patterns.

## License

MIT
