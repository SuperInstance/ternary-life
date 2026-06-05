# ternary-life

**Conway's Game of Life with three states. The dead are dead, the living age, and something new emerges.**

The original Game of Life has two states: alive and dead. Simple rules, complex behavior. Now add a third state: *old*. Cells that have been alive for multiple generations. They're not dead, but they're not fresh either. They have history.

The rules shift: a cell is *born* if it has exactly three young neighbors (not old — old cells are crowded, not fertile). A young cell *survives* with 1-2 neighbors. An old cell dies from overcrowding (3+ neighbors) or loneliness (0 neighbors). Old cells can't birth new cells.

The result is a Game of Life with *inertia*. Young cells are volatile — they appear and disappear like standard Life. Old cells are sticky — they persist longer, creating stable structures that the young cells orbit around. The population naturally evolves toward a mix: young cells at the frontier, old cells in the core, empty cells at the boundary.

## What's Inside

- **`Grid`** — the ternary grid. Values: 0 (empty/dead), 1 (young/alive), -1 (old/aging)
- **`new(width, height)`** — create an empty grid
- **`set(x, y, value)`** / **`get(x, y)`** — read and write cells
- **`tick()`** — advance one generation. Apply the ternary Life rules
- **`census()`** — count cells in each state. `{young, old, dead}`
- **`oscillator_detected()`** — has the grid entered a repeating cycle?
- **`tick_idempotent_empty()`** — an empty grid stays empty (the conservation baseline)

## Quick Example

```rust
use ternary_life::*;

let mut grid = Grid::new(10, 10);
// Place a "glider" of young cells
grid.set(1, 0, 1);
grid.set(2, 1, 1);
grid.set(0, 2, 1);
grid.set(1, 2, 1);
grid.set(2, 2, 1);

// Run 20 generations
for _ in 0..20 {
    grid.tick();
}

let c = grid.census();
println!("Young: {}, Old: {}, Dead: {}", c.young, c.old, c.dead);
// The glider moves, some cells age to -1, the pattern drifts
```

## The Deeper Truth

**Three-state Life has a natural aging process.** Standard Conway Life is binary — cells are either alive or dead, with no memory of how long they've been alive. Ternary Life adds *time* as a state variable. The old state means "this cell has been alive for a while" — it carries history.

This creates a completely different dynamical landscape. In standard Life, all living cells are interchangeable. In ternary Life, the population has *structure*: a young frontier of volatile cells and an old core of stable cells. The frontier explores new territory. The core preserves what works. This is exactly how organizations, ecosystems, and civilizations work — young agents explore, old agents consolidate.

The oscillator detection is crucial: ternary Life has more oscillator types than binary Life because the old state creates more possible configurations. A structure that oscillates between {young, old, young, old...} is a new kind of oscillator that doesn't exist in the binary game.

**Use cases:**
- **Cellular automata research** — the simplest extension of Conway's Life
- **Generative art** — ternary grids produce richer visual patterns than binary
- **Education** — demonstrate emergence, aging, and population dynamics
- **Game design** — terrain generation with natural aging
- **Multi-agent modeling** — young vs. old agent dynamics

## See Also

- **ternary-fire** — fire spreading is Life with directional rules
- **ternary-minority** — the anti-Life: cells flip to the minority state
- **ternary-morph** — morphological analysis of Life patterns (erosion, dilation)
- **ternary-sandpile** — another cellular automaton, but with self-organized criticality
- **ternary-ising** — spin dynamics (the physics cousin of Life)
- **ternary-color** — visualize Life grids with ternary color palettes
- **ternary-percolation** — do living cells form spanning clusters?

## Install

```bash
cargo add ternary-life
```

## License

MIT
