# Research Notes and Links

This file captures the useful context gathered before implementation.

## Jam rules and goals

Jam page: <https://itch.io/jam/caves-of-qud-modding-jam-5>

Key points:

- Jam: **Caves of Qud Modding Jam: Burgeoning**.
- Theme: **Burgeoning** — life flourishing, regrowth, biospheres, growth from ruins/rot/death.
- Submission window: **2026-07-01 12:00 UTC to 2026-07-08 12:00 UTC**.
- The jam is unranked.
- For consideration in the compilation modpack:
  - release under **CC0/public domain**,
  - use **data files only**,
  - do **not** submit scripting mods.
- Jam text says data files include XML, JSON, RPM maps, images, and music.
- Target latest stable Caves of Qud build listed by the jam: **1.0.5 / build 211.X**.

## Core wiki links

Modding overview:

- <https://wiki.cavesofqud.com/wiki/Modding:Overview>
- Main reminders:
  - top-level `manifest.json`, optional `workshop.json`,
  - XML files can be nested anywhere under the mod folder,
  - textures live under `Textures/`, but XML texture references omit the `Textures/` prefix,
  - prefix internal names to avoid conflicts,
  - specify only changed fields and use `Load="Merge"` where appropriate.

Useful modding pages:

- Objects: <https://wiki.cavesofqud.com/wiki/Modding:Objects>
- Bodies/anatomies: <https://wiki.cavesofqud.com/wiki/Modding:Bodies>
- Creature AI: <https://wiki.cavesofqud.com/wiki/Modding:Creature_AI>
- Giving creatures inventory: <https://wiki.cavesofqud.com/wiki/Modding:Giving_Creatures_Inventory_Items>
- Genotypes/subtypes: <https://wiki.cavesofqud.com/wiki/Modding:Genotypes_and_Subtypes>
- Mutations: <https://wiki.cavesofqud.com/wiki/Modding:Mutations>
- Activated abilities: <https://wiki.cavesofqud.com/wiki/Modding:Activated_Abilities>
- Snapjaw Mages tutorial: <https://wiki.cavesofqud.com/wiki/Modding:Tutorial_-_Snapjaw_Mages>

Relevant game-mechanics pages:

- Burgeoning: <https://wiki.cavesofqud.com/wiki/Burgeoning>
- Domination: <https://wiki.cavesofqud.com/wiki/Domination>
- Chimera: <https://wiki.cavesofqud.com/wiki/Chimera>
- Night Vision: <https://wiki.cavesofqud.com/wiki/Night_Vision>
- Mental shield: <https://wiki.cavesofqud.com/wiki/Mental_shield>
- Plant summoning tables: <https://wiki.cavesofqud.com/wiki/Population:PlantSummoning>
- Legendary creatures: <https://wiki.cavesofqud.com/wiki/Legendary_creature>

## Data-only feasibility conclusions

A new all-in-one mutation that summons a custom vine creature and immediately auto-dominates it is **not data-only feasible**. The mutation modding docs show that `Mutations.xml` registers a mutation, but new behavior requires a C# class derived from `BaseMutation`.

Data-only feasible version:

- Add new creatures to existing Burgeoning plant summoning population tables.
- Make those creatures valid Domination targets.
- Let the player use existing Burgeoning and existing Domination as a two-step system interaction.

This preserves jam-pack eligibility and makes the interaction feel like an intentional systems loophole.

## Mechanics constraints that matter

### Burgeoning

- Mental mutation.
- Summons allied plants into an area.
- Plant selection comes from `PlantSummoning` tables.
- Effective mental mutation level benefits from Ego, so high Ego indirectly improves access to higher plant-summoning tiers.

### Domination

- Mental mutation.
- Controls an adjacent creature while the user's original body lies dormant.
- Defender side uses mental defenses and level.
- Mental shield blocks Domination.
- Original body vulnerability is a core balance point and should remain.

### Mental shield

- Plants normally have mental shields.
- Mental shield blocks Domination.
- Sapient plants/fungi have mental shield removed in base-game patterns.
- Therefore, vine rovers must inherit from a sapient plant pattern or explicitly remove `MentalShield`.

### Chimera

- Chimera is a real morphotype mutation, not just a descriptive tag.
- Use actual `<mutation Name="Chimera" />` on rover creatures.
- If a genotype tag is used, it should not be a substitute for the actual mutation.
- Base-game NPCs show the pattern of using actual `Chimera` plus other body/mutation content.

### Night vision / vibration sense

- Use actual `DarkVision` or comparable existing mutation rather than pure description.
- Flavor can describe vibration-sense, but the gameplay affordance should be real vision-in-dark behavior.

### Passive AI

Creature AI can be influenced through the `Brain` part.

Useful attributes/parts for this project:

- `Hostile="false"`
- `Wanders="false"`
- `Calm="true"`
- `Factions="Vines-100"`
- `Staying="true"` — observed to stop Burgeoning-spawned rovers from following the player.
- `IgnoreCombat="true"` — suppresses active combat pursuit in testing.
- `HostileWalkRadius="0"`, `MinKillRadius="0"`, `MaxKillRadius="0"` — suppress movement to engage hostiles in testing.
- `<part Name="AISuppressIndependentBehavior" />` — observed to stop autonomous attacking/bodyguard behavior in one test pass.

Avoid:

- `Mobile="false"` on rovers, because a dominated rover must be able to move.

In-game testing notes so far:

- Mod loads without build-log errors.
- Domination works on spawned rovers.
- Plain `Hostile="false"`, `Wanders="false"`, and `Calm="true"` were not enough; Burgeoning-created allies followed the player and defended them.
- `AISuppressIndependentBehavior` stopped autonomous attacking.
- `Staying="true"` stopped following.
- `IgnoreCombat="true"` and zero kill/walk radii stopped active pursuit/defense behavior; rovers still attack adjacent enemies, which feels acceptable and natural.
- The private pre-jam prototype felt good in play: rovers were fun to control directly, and the inert-chassis behavior supported the intended fiction.
- Emergent play pattern: use Burgeoning to grow a garden, dominate a rover, leave the original body guarded by the plant patch, and lure enemies back toward the garden.

## Base-game data reference points

On a typical Linux Steam install, base data is under:

`$HOME/.local/share/Steam/steamapps/common/Caves of Qud/CoQ_Data/StreamingAssets/Base/`

Useful files and things to inspect:

- `Mutations.xml`
  - `Burgeoning` mutation entry.
  - `Domination` mutation entry.
  - `Chimera` morphotype entry.
  - `DarkVision` / Night Vision class references.
- `ActivatedAbilities.xml`
  - `CommandBurgeoning`.
  - `CommandDominateCreature`.
  - `CommandEndDomination`.
- `ObjectBlueprints/Creatures.xml`
  - `MutatedPlant` base object.
  - `MutatedVine` base object.
  - `SapientMutatedVine`, which removes `MentalShield`.
  - `NaturalWeapon`, `NaturalFist`, `SoftManipulator`, `Bite`, etc.
  - `Jilted Lover` and `Lovers_Thorns` as a plant natural-weapon example.
  - Irudad / other chimeric NPCs as actual `Chimera` examples.
- `Bodies.xml`
  - `Vine` anatomy.
  - Plant body-part variants like bines/tendrils/roots.
  - Chimera-relevant body part weights and body part definitions.
- `PopulationTables.xml`
  - `PlantSummoning1` through `PlantSummoning9`.
  - `PlantSummoning_Generics*`.
  - Examples of `Load="Merge"` style table changes from tutorials.
- `Genotypes.xml`
  - Only use if defining a real player genotype; not needed for rover creatures unless design expands.

## Implementation patterns

### Object blueprints

Use `ObjectBlueprints/Creatures.xml` for:

- vine rover creature blueprints,
- rover natural weapons,
- possibly rare elite variants.

Prefer inheritance from base plant/vine types, then override only what differs.

### Bodies

Use `Bodies.xml` for custom anatomies:

- start from the base `Vine` idea,
- increase tendril/hand-equivalent count over tiers,
- ensure body slots line up with natural weapons.

### Population tables

Use `PopulationTables.xml` to merge rovers into Burgeoning tables:

- `PlantSummoning1` through `PlantSummoning9`,
- lower-tier weak rovers in low tables,
- stronger chimeric bodies in higher tables,
- rare elite/legendary-like rovers can be explicit rare variants rather than relying on procedural legendary generation.

### Natural weapons

Natural weapons should be real equipment objects inheriting from `NaturalWeapon`, then assigned by `inventoryobject`.

Use existing examples before inventing fields:

- `Lovers_Thorns` for plant thorns,
- `SoftManipulator` / `HardManipulator` for hand-equivalent manipulators,
- other `NaturalWeapon` derivatives for damage/skill conventions.

## Validation checklist

- No `.cs` files.
- Mod manager loads the mod without build-log errors. Confirmed in early testing.
- Each rover blueprint can be wished for.
- Population table sampling can produce rovers via Burgeoning.
- Rovers are valid Domination targets. Confirmed in early testing.
- Undominated rovers remain passive: they should not follow or move to defend before domination. Adjacent self-defense is currently acceptable.
- Dominated rovers can move.
- Dominated rovers can see in the dark.
- Natural weapons appear and work.
- Higher Burgeoning effective levels access better rover bodies.
- License remains CC0.
