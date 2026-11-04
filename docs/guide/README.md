# The bevy_stigmergy guide

| Page | What is in it |
|---|---|
| [The field, and what it leaves to you](model.md) | stigmergy, the channel table, the gradient at a wall, why the crate owns no schedule and names no channel, and `CellFrame` |
| [Ticking the field](ticking.md) | what `dt` means, conservation under diffusion, which thread does the work, `evaporate_diffuse_with` on your own pool, and bit-reproducibility |
| [Three gotchas](gotchas.md) | deposits mask their destination not their path, deposit order, and non-finite amounts |
| [The examples](examples.md) | `trail`, `rally` and `foraging`, and how to pick weights that are in scale |
| [References](references.md) | the papers and chapters the crate is built on |
