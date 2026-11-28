# AE2 Catalyst Patterns

A small Applied Energistics 2 addon for Minecraft 1.20.1 (Forge) that lets
you mark ingredients of a processing pattern as **catalysts**: things a
recipe needs at the machine but does not consume, like molds, casts,
presses and similar tools in any mod.

## The problem

AE2's crafting planner assumes every pattern ingredient is consumed once
per operation. Put a non-consumed item into a pattern and a 100-operation
job suddenly demands 100 of them, and the planner may even try to
autocraft more of them recursively. Leave it out of the pattern and the
planner does not know it is needed at all.

## How it works

1. Open AE2's pattern encoding terminal in processing mode and set up the
   pattern as usual.
2. **Alt+click** an input slot to mark that ingredient as a catalyst.
   Marked slots get an orange frame, and the slot tooltip shows the state.
3. Encode. If at least one ingredient is marked, the result is a
   **Catalyst Pattern**; with no marks it stays a plain processing
   pattern.
4. Put the pattern into a pattern provider as usual.

You can also convert an existing processing pattern: put it alone in a
crafting grid to get a catalyst pattern, then hold it and right-click to
edit the marks afterwards. Loading a catalyst pattern into the encoding
terminal restores its marks onto the slots.

Planning: a catalyst counts **at most once per pattern**, no matter how
many operations the job runs, because it reports itself as its own
"remaining item" and AE2's planner reuses remaining items across runs
(the same mechanism that makes buckets work in crafting patterns). A
catalyst is **never required and never crafted**: if one is in stock,
the plan allocates it; if none is in stock, the plan simply goes ahead
without it, on the assumption that one is already resident at the
machine.

Execution: if the planner allocated a catalyst, it is taken from ME
storage and pushed to the machine with the first operation of the job.
The CPU never waits for it to come back. Once no running job needs the
catalyst anymore and the machine no longer holds any of the pattern's
other inputs, the catalyst is taken back out of the machine and
returned to ME storage, ready for the next job. If none was allocated,
nothing is pushed for the catalyst and the resident one at the machine
does the work.

## Notes and limits (v0.1.0)

- A catalyst that only exists inside its machine is fully supported:
  jobs plan and run without one in storage. The flip side is that a job
  also starts when the catalyst is nowhere at all — the other inputs
  are pushed and the machine simply cannot start until a catalyst is
  put in.
- Automatic returns work with both block pattern providers and cable
  part providers.
- Deliveries are remembered in memory only. If the game stops while a
  catalyst is out, it stays at the machine (take it out by hand once).
- The catalyst is taken back through the machine's item inventory, the
  same way it was pushed in. Some machines refuse automation extraction
  from the slot that holds the catalyst — AE2's own inscriber does
  this for its presses unless it is set to separated sides. For those,
  the catalyst is taken from the machine's internal inventory instead,
  the same inventory its GUI shows.
- Mekanism machines are supported the same way (optional — Mekanism
  does not need to be installed). Their side configuration decides
  what automation may extract, so if the capability refuses, the
  catalyst is taken from the machine's internal slot list instead and
  no side setting is needed. This covers machines such as Astral
  Mekanism & Energistics' Mekanical Inscriber.
- Use one pattern provider per catalyst pattern. If the same catalyst
  pattern is served by several providers (parallel machines), only the
  first machine to receive a push gets the catalyst from storage.
- Two different patterns that share the same catalyst in a single job
  also share the one allocated copy. Run them as separate jobs, or stock
  the second machine by hand.
- Do not use a provider's blocking mode with catalyst patterns: the
  resident catalyst makes the machine look permanently busy to that
  check.
- The catalyst pattern currently looks like a normal processing pattern.

## Building

Forge MDK 1.20.1-47.4.0, Java 17. AE2 is pulled from CurseMaven
(file 8991271). Run `gradlew build`, or use the GitHub Actions workflow.
