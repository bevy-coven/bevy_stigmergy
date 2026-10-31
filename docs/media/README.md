# Media: every image, with the command that made it

| File | What it shows | Command | Captured | Shows |
|---|---|---|---|---|
| `docs/foraging.gif` | the README's hero: 90 foragers find the gap in the wall; 8 s, 80 frames | not recorded when it was made. `examples/foraging.rs` now has a capture knob: `STIG_FRAMES=480 STIG_SHOT=/tmp/forage/f STIG_EVERY=6 cargo run --release --example foraging`, then the org guide's `ffmpeg` palette command over `/tmp/forage/f-*.png` | 2026-08-09 (`ff85cae`) | 0.1.0 |
| `docs/media/foraging.png` | the README's still: the hero's last frame, the trail the colony settled on | `ffmpeg -i docs/foraging.gif -vf "select=eq(n\,79)" -frames:v 1 docs/media/foraging.png` | 2026-10-31 | 0.1.0 (from the GIF) |

`docs/foraging.gif` stays at that path: the READMEs of published releases name it, and a published README never changes.
