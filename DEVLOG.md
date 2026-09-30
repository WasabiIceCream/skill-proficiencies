# DEVLOG — Gameoverse's Skill Proficiencies fork

Fork of [balovich-matje/specialities](https://github.com/balovich-matje/specialities)
(MIT), kept here for a fix and a feature upstream hadn't picked up yet.

## 2026-09-29 — rebased onto upstream 1.6.4 (branch `gameoverse-1.6.4`)

Upstream 1.6.4 ships its own extended level cap (GitHub #6: `extendedLevels` + `extendedMaxLevel`, server-synced, up
to 999), so our `maxLevel` change is dropped; this server's config now sets `extendedLevels: true`,
`extendedMaxLevel: 999` (the old `maxLevel` key removed). Levels are stored as total XP either way, so nothing migrates.
Still carried: the XP-rounding fix (`SkillManager.addXp`: with `xpRateMultiplier: 0.25`, a 1 XP gain rounded to 0),
cherry-picked onto upstream's new cap logic. Built from the `26.1-fabric` node; deployed as
`specialities-1.6.4+26.1.2-gameoverse.jar`. The old `main` branch (1.6.3 + both fixes) is kept as history.

## 2026-09-20 — XP-rounding fix and configurable max level

Two asks, both raised with the upstream author first and not acted on there:

1. **XP-rate-too-low grants 0 XP.** `SkillManager.addXp()` rounded
   `amount * xpRateMultiplier` and silently dropped the whole gain whenever
   that rounded to 0 — confirmed live on this server's own config
   (`xpRateMultiplier: 0.25`, where a 1 XP gain rounds to 0). Fixed: only an
   exact `0.0` multiplier (the documented "disabled" value) skips XP now;
   any positive multiplier guarantees at least 1 XP.

2. **Configurable max skill level, requested at 999.** `Tuning.MAX_LEVEL`
   was a hardcoded `100`. Every level-scaled bonus formula in `Tuning.java`
   already divides by that constant rather than a literal `100`, so it was a
   clean conversion: turned it into `Tuning.maxLevel()`, backed by a new
   `maxLevel` field in `SpecialitiesConfig` (Cloth Config screen + JSON,
   clamped to `[1, 999]`). This server's config now sets it to `999`.

Both changes are the shared `src/` tree (Stonecutter-preprocessed across
every version/loader node), built and verified against the `26.1-fabric`
node specifically since that's this server's actual target (`BUILD
SUCCESSFUL`, `specialities-1.6.3+26.1.2.jar`). Boot-tested clean alongside
the re-enabled Chopt and Crackt fixes.

**Mechanism worth knowing, not currently a live concern**: every level-scaled
bonus (damage, mining speed, sweeping, milestones, luck, etc.) is computed as
a fraction of the current max level, so raising the cap stretches the same
curve out rather than adding a new tier past the old one. No players have
actually played on the server yet, so there's no existing progress this
affects right now — but if the cap is ever raised again after real players
have real levels, the same fraction-of-cap scaling would reduce everyone's
effective bonuses at their current level until they grind further. Worth
remembering before doing that later, not a concern for this change.

**If upstream ever ships either fix**, diff against the real release first
(`SkillManager.java` for the XP fix, `Tuning.java` + `SpecialitiesConfig.java`
+ the config-UI/lang files for the max-level feature) before switching back.
