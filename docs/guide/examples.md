# The examples

Moved from the README on 2026-10-31, unchanged, when the README was brought to the [org documentation guide](https://github.com/bevy-coven/.github/blob/main/DOCUMENTATION.md).

```sh
cargo run -p bevy_stigmergy --example trail     # terminal: a scent trail spreads, fades, respects walls
cargo run -p bevy_stigmergy --example rally     # terminal: the vectorial pheromone tracking a target
cargo run -p bevy_stigmergy --example foraging  # the gif above. Needs a GPU.
```

`trail` and `rally` render the field as ASCII — no window, no GPU. `trail` lays a channel along a walked path across a map with a wall and a single gap, so you can watch diffusion go through the gap and not through the wall. `rally` has three fixed scouts marking a target that keeps moving, then stops marking so you can see the recruitment expire itself.

`foraging` is three channels at once — a HOME beacon, a FOOD beacon, and a TRAIL that only laden foragers lay — and it is the example to read if you are tuning this into a game, because the single hardest thing here is picking weights that are in scale with each other. Measured in that example, a beacon gradient averages about `0.09` per cell. The first wander term tried was `2.2`, which is *larger than the homing signal it competes with*, so laden foragers wandered the whole map laying trail everywhere and the "path" was an undifferentiated wash. Steering weights have to be set against `weight * |gradient|`, not by taste — and the way to find out is to print the gradient magnitudes rather than squint at the picture.
