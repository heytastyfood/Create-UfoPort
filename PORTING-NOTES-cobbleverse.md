# Porting notes: Create-UfoPort on the Cobbleverse pack (205 mods, fabric 1.21.1)

## Source fix (committed here)
- `modules/porting_lib_ufo/.../entity/mixin/common/EntityMixin.java` —
  `port_lib$entityInit` (TAIL of `Entity.<init>`) virtual-called `getDimensions()`
  before subclass ctors ran; Cobblemon 1.7.3 `PokemonEntity.getDimensions`
  NPEs on `this.effects` there, so every saved Pokemon failed chunk NBT load and
  was DROPPED (23 lost in spawn chunks alone). Guarded with try/catch Throwable,
  falls back to vanilla dimensions. See commit 7eae3f86.

## Server-side companion fix (not in this repo, documented for reproduction)
- Supplementaries 3.6.7 ships Create-compat recipes gated only on
  `fabric:mod_loaded create`, referencing `create:cardboard` (a Create 6.x item;
  this port is the 0.5 line -> item absent). 3 parse ERRORs at boot
  (`supplementaries:present_2`, `trapped_present_2` recipes + advancements).
  Fixed on the server with world datapack
  `labworld/datapacks/ufoport-supplementaries-compat-fix/` overriding those 4
  files with a never-true `fabric:load_conditions`, so they are skipped cleanly.
  A robust in-jar fix is not possible: mod-datapack override ordering between
  mods is not deterministic, and registering a `create:cardboard` item would
  invent Create 6 content.

## Benign log noise (expected, safe)
- `Method overwrite conflict ... porting_lib_ufo_(transfer|model_loader)` vs
  sophisticatedcore's bundled porting_lib 3.1.0-beta.47 fragments: both lineages
  add identical-descriptor helper methods (`port_lib$getItemCache`, etc.);
  first writer wins, implementations are equivalent ports of the same code.
- `Shift.BY=2 ... LivingEntityMixin` maxShiftBy warning: cosmetic.
- night-config 3.6.3 JiJ vs pack's 3.8.0: loader picks 3.8.0, fine.
