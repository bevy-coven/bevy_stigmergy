# The field, and what it leaves to you

Moved from the README on 2026-10-31, unchanged, when the README was brought to the [org documentation guide](https://github.com/bevy-coven/.github/blob/main/DOCUMENTATION.md).

## The idea

Stigmergy is coordination *through the environment*. Agents write into a shared field and read it back, so a crowd converges on a trail, disperses from a threat, or recruits toward an alarm without anybody negotiating — the field is computed once and shared by all of them, and the group behaviour falls out.

```rust,ignore
use bevy_math::IVec2;
use bevy_stigmergy::{ChannelDef, StigGrid};

const SCENT: usize = 0;
const THREAT: usize = 1;

let mut field = StigGrid::<2>::new(width, height, floor_cells(), [
    ChannelDef { evaporate: 0.4, diffuse: 0.10, deposit_radius: 1.5 },  // SCENT
    ChannelDef { evaporate: 1.2, diffuse: 0.05, deposit_radius: 3.0 },  // THREAT
]);

field.deposit(SCENT, cell, 1.0);        // placement
field.evaporate_diffuse(dt);            // decay + spread, once a tick
let here = field.sample_cell(SCENT, c); // query
let uphill = field.gradient_cell(SCENT, c);
```

`gradient_cell` is the plain central difference `(v(x+1) − v(x−1), v(y+1) − v(y−1))` in open floor, and it reads a wall the way the tick does: a face nothing crosses. A neighbour that is rock or off the grid counts as the cell's own value, so a flat field has no slope beside a wall and an agent fleeing a threat is not pulled into it, and across a wall the component is the one-sided difference from the floor side. A cell that is not floor — rock, or off the grid however far — has no slope and reads `Vec2::ZERO`, as does an undefined channel.

## What it deliberately does not do

**It owns no resources, registers no systems, and has no schedule.** When deposits drain, when evaporation runs, and where both sit relative to the agents reading the field are gameplay decisions. A crate that made them would be unusable in any game that wanted different ones.

**It does not name your channels.** A channel table is game content. Channels are `usize` indices and `N` is yours to pick, so "what does channel 3 mean" stays a question about your game.

**It works in cell space, not world space.** Every entry point takes an `IVec2` cell. The crate does not decide how big a cell is or where the grid sits in the world.

`CellFrame` is an optional plain value type for the mapping nearly every caller writes anyway — an axis-aligned grid of uniform cells:

```rust,ignore
use bevy_math::Vec2;
use bevy_stigmergy::CellFrame;

// Cover a 120 x 80 metre field in 2-metre cells.
let frame = CellFrame::covering(Vec2::new(-60.0, -40.0), Vec2::new(60.0, 40.0), 2.0);

let mut field = StigGrid::<2>::new(frame.width, frame.height, frame.floor_cells(is_floor), [
    ChannelDef { evaporate: 0.4, diffuse: 0.10, deposit_radius: 1.5 },
    ChannelDef { evaporate: 1.2, diffuse: 0.05, deposit_radius: 3.0 },
]);

field.deposit(SCENT, frame.world_to_cell(agent), 1.0);
let uphill = field.gradient_cell(SCENT, frame.world_to_cell(agent));
```

`cell_centre`, `contains` and `cells()` (ascending row-major, the order `StigGrid::new` keeps its floor set in — though it accepts any order, and a repeated cell counts once) round it out. It is a convenience, not a requirement: a caller with a hex sheet, a rotated board or a camera-relative projection ignores it and keeps its own mapping, and nothing in the crate behaves differently either way.
