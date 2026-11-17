# Lucis

Lucis is a lightweight server-side light engine. It replaces expensive parts of the vanilla Minecraft lighting pipeline with a faster region-based system designed for high update throughput, faster chunk lighting and better scaling on multi-core CPUs allowing for faster worldgen and spawning massive structures without drastic TPS drops.

Originally started as a research project to learn how Minecraft's light engine works, ended rewriting lighting because the original one was too slow for me and decided to have some fun attempting to do something with it!

## What Lucis does? Why it exists?

**Why?** Mostly for fun and researching. I wanted to spawn huge (like really huge!) structures into the world, but got stuck that vanilla light engine just... you know haven't expected THAT much amount of blocks to calculate light and just hangs itself, so I've tried to solve that.

Yes, Starlight and ScalableLux exists, you can use them too. However, I wanted to research and look inside myself instead of using existing solutions. So, here is **Lucis**, that speeds up lighting up to ~8-12x than vanilla and ~1.5x faster than ScalableLux/Starlight!

Lucis optimizes chunk lighting during world generation on a chunk preparation stage and runtime block light and skylight updates after block changes.

Instead of relying only on vanilla per-update lighting behavior, Lucis groups light changes by region, processes them asynchronously where possible, then publishes the resulting light data back to Minecraft's light engine in batches.

The goal is very simple. Reduce lighting cost during heavy block updates, chunk loading and skylight changes.

## How it even works? Is this another fork of Starlight?

No, this is an another approach to optimize light calculations where possible. Let me attempt to describe that.

Minecraft stores light per section and updates it whenever blocks are placed, removed, generated or changed. While that will work well for general gameplay, but it can become slow when the game has to process many light-affecting blocks in a short time or generating the world.

Lucis changes this by collecting lighting work into regions. A region is a small owned area of chunks that can be processed as one unit. When a block update happens, Lucis records the change, groups it with nearby changes, and sends that work to its runtime lighting system. This avoids treating every small update as a fully separate lighting operation.

For chunk lighting, Lucis can compute light during world generation using its own relighting path. It reads the chunk and nearby light data, calculates skylight and block light, then publishes the finished light sections back into Minecraft light thread.

As for runtime block changes, Lucis uses it's own light queue. If many blocks change during the same tick or close together, the queue lets Lucis process them as grouped region jobs. This is especially useful for dense block updates, terrain edits, generated structures, and skylight changes caused by roofs or large openings.

Lucis also separates calculation from publishing. The heavy work can be done outside the main light update path, so the final section data can be published back in batches. This reduces the number of small light tasks that have to go through the light thread one by one. The result is lower overhead when many light changes happen together.

The main technical quirk is that we still need to respect Minecraft's chunk state and light engine rules to not break many other mods that somehow get light data or implements it's own system. Lucis computes new light data, tracks changed regions, and then hands the result back to the engine.

## Compatibility

While Lucis attempts to maintain compatability with other mods where possible and it should not break anything, something may break eventually. Proceed with a caution when it comes to a large modpacks.

Known notes:
- **ScalableLux** (or any Starlight fork) is incompatible by definition. You should choose only Lucis or only ScalableLux, not both. Attempting to put both will lead to many crashes. Do not try to throw both into your modpack, thanks. It won't work that way.
- **Sable** is compatible with Lucis. Currently Lucis does not inject its light for Sable plots, so they're currently use vanilla lighting for their projections. May be changed later once a better solution is found.
- Should work mostly fine with most of worldgen optimization mods such as **Generator Accelerator**, **C2ME**, **Fast Noise** and **ModernFix** (some patches still count I guess). Lucis will speed up the world generation up to +15-25% in terms of speed in different scenarios. Neat, right?

If a mod also replaces or heavily modifies Minecraft lighting internals then it is very likely to conflict with Lucis.

## Modpacks

You can use it in personal and public modpacks, including distributed packs, as long as the LGPLv3 license terms are respected.
