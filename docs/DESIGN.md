# Piloting Lattice Design

Piloting Lattice is a data-only Caves of Qud mod for Caves of Qud Modding Jam 5: Burgeoning.

## Jam timing note

The jam encourages physically creating mod data files during the submission window: **2026-07-01 12:00 UTC to 2026-07-08 12:00 UTC**.

This repository was created after the jam window opened. Its initial commit ports clean concept/design documentation only; the actual mod data files are created in later jam-window commits.

## Core concept

Piloting Lattice adds **passive chimeric vine rovers** to Burgeoning's plant summon tables.

The play loop is:

1. Use **Burgeoning** to grow a patch of plants.
2. Sometimes, a passive, mobile, low-ego vine chimera grows among them.
3. Use existing **Domination** on the vine rover.
4. Pilot the physically useful plant body while the player's original body lies dormant.

This is intentionally a data-only systems interaction: Burgeoning grows the body; Domination supplies the pilot.

## Design rules

- Data-only: no C#, Harmony, DLLs, or scripting.
- CC0/public domain for jam-pack eligibility.
- No fake mechanics as flavor:
  - rovers need the real `Chimera` morphotype mutation,
  - rovers need custom chimeric vine anatomies,
  - rovers need actual natural weapons and combat skills,
  - rovers need to be valid Domination targets by inheriting from `SapientMutatedVine` or otherwise lacking plant `MentalShield`.
- Rovers are good bodies and bad minds:
  - Ego 1,
  - very low Willpower / MA,
  - very low Intelligence,
  - strong physical bodies, senses, natural weapons, and body-appropriate combat skills.
- Undominated rovers should be inert chassis, not free summons:
  - they do not follow the player,
  - they do not pursue or bodyguard against enemies,
  - adjacent self-defense is acceptable.

## Implementation blueprint

Expected files once implementation begins:

- `manifest.json` — mod metadata.
- `ObjectBlueprints/Creatures.xml` — rover creatures, passive AI controls, natural weapons, mutations, and skills.
- `Bodies.xml` — custom vine chimera anatomies with increasing tendrils/limbs.
- `PopulationTables.xml` — merges rover entries into `PlantSummoning1` through `PlantSummoning9`.
- Optional `Textures/...` — custom sprites if time allows; base-game plant/natural-weapon tiles are sufficient for a first build.

## Passive AI blueprint

Rovers should inherit from `SapientMutatedVine`, which removes plant `MentalShield` and makes Domination possible. The inherited sapient-vine mental stats must be overridden using `sValue` to avoid base stat ranges leaking through.

Recommended shared base rover settings:

```xml
<part Name="Brain" Hostile="false" Wanders="false" Mobile="true" Calm="true" Staying="true" IgnoreCombat="true" HostileWalkRadius="0" MinKillRadius="0" MaxKillRadius="0" Factions="Vines-100" />
<part Name="AISuppressIndependentBehavior" />
```

Intent:

- `Mobile="true"`: dominated rovers can move.
- `AISuppressIndependentBehavior`: suppress autonomous bodyguard behavior.
- `Staying="true"`: prevent rovers from following the player around.
- `IgnoreCombat="true"`, `HostileWalkRadius="0"`, `MinKillRadius="0"`, `MaxKillRadius="0"`: prevent undominated rovers from moving to engage hostiles.

Prototype observations from private pre-jam testing:

- Domination worked on rovers.
- Rovers were fun and useful to control directly.
- `AISuppressIndependentBehavior` stopped autonomous bodyguard attacking.
- `Staying="true"` stopped following the player.
- `IgnoreCombat="true"` plus zero kill/walk radii stopped active enemy pursuit.
- Rovers may still attack adjacent enemies, which felt acceptable and natural.
- Emergent play pattern: grow a garden, dominate a chassis, trust the surrounding plant patch to guard the original body, and lure enemies back toward the garden when useful.

## Rover line blueprint

| Blueprint | Tier role | Key traits |
|---|---|---|
| `PilotingLattice_TwitchingRootling` | low-tier scout/body | Level 1, fast/frail, 2 probing tendrils |
| `PilotingLattice_VineHusk` | early combat chassis | Level 3, forked body, tendrils + thorns, starts multiweapon |
| `PilotingLattice_BraidedVineRover` | mid-tier multi-limb body | Level 6, stronger body, thorns + root clubs, Telepathy |
| `PilotingLattice_KnottedVineMarionette` | high-tier combat chassis | Level 9, stronger cudgel/short-blade skill package, Telepathy |
| `PilotingLattice_CrownedTrellisHusk` | rare top-tier chassis | Level 13, strongest physical body, best skill package, Telepathy |

All rovers should have:

- `Chimera`
- `DarkVision`
- `PhotosyntheticSkin`
- very low mental stats
- passive AI controls
- `ExcludeFromDynamicEncounters`
- `Cudgel`, `ShortBlades`, and `Tactics_Run`

Higher tiers can add Multiweapon, Cudgel, Short Blade, Endurance, Telepathy, and Charge-related skills.

## Population table blueprint

Merge rovers into all actual Burgeoning plant summoning tables, `PlantSummoning1` through `PlantSummoning9`.

Target approximate per-plant-table-draw chance based on the prototype weights:

| Table | Direct chance | Approx. including recursive neighboring tables |
|---|---:|---:|
| `PlantSummoning1` | 2.88% | 3.73% |
| `PlantSummoning2` | 6.12% | 6.74% |
| `PlantSummoning3` | 5.23% | 6.13% |
| `PlantSummoning4` | 6.82% | 7.57% |
| `PlantSummoning5` | 4.84% | 5.87% |
| `PlantSummoning6` | 7.89% | 8.86% |
| `PlantSummoning7` | 8.33% | 9.61% |
| `PlantSummoning8` | 10.29% | 11.52% |
| `PlantSummoning9` | 7.78% | 10.02% |

Because Burgeoning creates multiple plants per cast, per-cast chance of seeing at least one rover is substantially higher than per-table-draw chance.

## Validation checklist for the in-window implementation

- Data files are created after 2026-07-01 12:00 UTC.
- No `.cs` files are present.
- Mod loads without build-log errors.
- Rovers can be wished into existence.
- Rovers can appear from Burgeoning table sampling.
- Rovers are valid Domination targets.
- Rovers do not follow the player before domination.
- Rovers do not move to attack on behalf of the player before domination.
- Adjacent self-defense remains possible and is acceptable.
- Dominated rovers can move, see in darkness, and use their natural weapons.
- License remains CC0.
