# CLAUDE.md

## Project

A standalone **Battle Brothers** mod that ports **World of Warcraft classes** into the game, each with a handful of basic abilities. Standalone means it does not depend on the Legends mod. It may depend on Modern Hooks and MSU.

Battle Brothers has no classes, so a WoW class is built from:
- **Background**: the "class". Sets starting stats, talents, gear and the perk tree it can use.
- **Active skills**: the abilities (Charge, Fireball, Backstab...). One `.nut` script each.
- **Perks**: the talent tree. Usually grants a skill or modifies one.

Fan project, non-commercial. Do not ship Blizzard art, audio or text. Reuse vanilla Battle Brothers icons and sounds. Ability names can be homages; keep descriptions our own.

## Hard constraints (read first)

- **You cannot run the game.** Never claim a mod "works". Write the code, then ask the user to test and paste back the game log or error text. Treat their log as the source of truth.
- **Never commit decompiled vanilla scripts** or anything extracted from `data_001`. They are copyrighted. Commit only our own files. Keep the decompile in a gitignored folder (`vanilla/`) for reference.
- Squirrel (the game's language) fails at runtime, not compile time. A typo only shows up in the log when that code path runs. Be careful and re-read what you write.
- Do not guess engine APIs. **Grep the decompiled vanilla scripts for a working example first**, then copy its pattern. If you are unsure, say so and ask.

## Tooling

- **Adam Milazzo's mod kit**: decrypts and decompiles scripts, and packs sprite sheets (`bbrusher`). Docs: http://www.adammil.net/blog/v133_Battle_Brothers_mod_kit.html
- **bbkit** (https://github.com/Enduriel/bbkit): sets the working directory, decompiles, and builds and installs the mod zip.
- **Modern Hooks** and **MSU** (Nexus Mods, Battle Brothers). MSU docs: https://github.com/MSUTeam/MSU/wiki
- Windows-focused tooling. Work happens on the user's machine; give commands for Windows (cmd/PowerShell) unless told otherwise.
- Community help: the BB Modding Discord.

Decompile steps (user runs these):
1. Extract `data_001` from the game's `data` folder with 7-Zip into `vanilla/data_001/`.
2. Run the kit's `massdecompile.bat` on `vanilla/data_001/scripts`.
3. Read and grep the resulting `.nut` files.

## Repo layout

```
/
├── CLAUDE.md
├── README.md
├── .gitignore            # must ignore vanilla/, *.zip builds, kit binaries
├── mod_wow_classes/      # the mod source, mirrors the game's data folder
│   ├── scripts/
│   │   ├── !mods_preload/        # mod registration + hooks entrypoint
│   │   ├── skills/actives/       # ability scripts
│   │   ├── skills/perks/         # perk scripts
│   │   └── skills/backgrounds/   # class backgrounds
│   ├── gfx/                      # only if we add sprites
│   └── preload/                  # only if we add sprite sheets
├── tools/                # build/install scripts (our own)
└── docs/                 # design notes, class specs, test log
```

The zip is dropped **unextracted** into `Battle Brothers\data`. Name it with a version, e.g. `mod_wow_classes_0.1.0.zip`. Folder structure inside the zip must mirror the game's `data` folder exactly.

## Squirrel conventions

Verify each against a vanilla example before relying on it.

- Squirrel 2.x. Tabs for indentation. Classes use `this.inherit("scripts/skills/skill", {...})`.
- Typical active skill: file in `scripts/skills/actives/`, `this.skill.create()` then set `this.m.ID`, `Name`, `Description`, `Icon`, `IconDisabled`, `Overlay`, `SoundOnUse`, `Type`, `Order`, `IsActive`, `IsTargetingActor`, `IsAttack`, `ActionPointCost`, `FatigueCost`, `MinRange`, `MaxRange`. Override `getTooltip()`, `isUsable()`, `onVerifyTarget()`, `onUse()`.
- Prefer **hooking** vanilla classes (Modern Hooks / MSU) over overwriting vanilla files, so we stay compatible with other mods. Never copy a whole vanilla file just to change one function.
- Use our own prefix for every ID and file: `wow_` (for example `actives.wow_charge`, `perk.wow_improved_charge`). Never collide with vanilla IDs.
- Register the mod and its version through the MSU/Modern Hooks API in `!mods_preload`.
- Log with `this.logInfo(...)` while debugging; remove noisy logs before release.

## Design rules for classes

- **No mana.** Battle Brothers uses Action Points (AP) and Fatigue. Map resource costs to fatigue and AP. Document the mapping per class.
- Keep power in line with vanilla. The game is about attrition and permadeath; a class should be interesting, not dominant. Compare every number against a similar vanilla skill.
- Prefer reusing vanilla effects and skills (rend, stun, daze, rally, etc.) before inventing new status effects.
- Pets, summons, shapeshifting and DoT spell systems are **hard**. Defer them to late stages.
- One spec doc per class in `docs/classes/<class>.md`: fantasy, background stats, ability list with AP/fatigue/range/effect, perk tree, and which vanilla skill each ability is based on.

## Class order (easiest first)

1. **Warrior**: Charge-style gap closer, Rend, Shield Slam, Battle Shout (rally-like). Vertical slice, build this first.
2. **Rogue**: Backstab, Sinister Strike, Evasion, Kidney Shot.
3. **Hunter**: Aimed Shot, Multi-Shot, Concussive Shot. Defer pets.
4. **Paladin**: Judgement, Holy Light (heal), Blessing/aura, Hammer of Justice.
5. **Shaman**: Lightning Bolt, Healing Wave, Frost Shock, totems (hard).
6. **Mage / Warlock / Priest / Druid**: custom spells, DoTs, summons, forms. Last.

## Staged plan

- **Stage 0**: repo, `.gitignore`, build/install script, mod registration that loads cleanly with an empty "hello" log line. User confirms it appears in the game log.
- **Stage 1**: Warrior background plus one skill (Charge). User tests.
- **Stage 2**: finish Warrior skills and a small perk tree. Add a balance pass.
- **Stage 3+**: next classes, one at a time. Release on GitHub after each class.

Do one stage at a time. Do not start the next until the user reports the current one works.

## Workflow for each task

1. Read the relevant vanilla example (grep `vanilla/`). Name the file you copied the pattern from.
2. Write the smallest change that does the thing.
3. Update the class spec in `docs/classes/`.
4. Tell the user exactly what to test in-game (for example "start a new campaign, hire a Warrior background brother, use Charge on an enemy") and what log lines to paste back.
5. Commit with a clear message. Small commits, one idea each.

## Testing log

Keep `docs/test-log.md`. After each user test, record the date, the version, what was tried, the result and any error text pasted from the game log. Use it to avoid repeating fixes.

## Releases

- SemVer in the mod registration and in the zip name.
- Use GitHub Releases to attach the zip.
- README must list requirements (Modern Hooks, MSU), install steps (drop zip unextracted into `data`), and known issues.
