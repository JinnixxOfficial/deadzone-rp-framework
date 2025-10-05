# deadzone-rp-framework

> **Automation-first RP core for wasteland servers: procedural encounters, scarcity economy, and faction reputation.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](#license) [![FiveM](https://img.shields.io/badge/FiveM-Framework-orange)](#requirements) [![Status](https://img.shields.io/badge/status-alpha-informational)](#roadmap)

> **TL;DR**: Deadzone is a modular FiveM RP framework designed for **post‑apocalyptic** servers. It favors **AI‑assisted systems** (encounters, economy, factions) and **clean extension points** so you can build unique survival stories without fighting the core.

---

## Table of Contents
- [Features](#features)
- [Why Deadzone?](#why-deadzone)
- [Architecture](#architecture)
- [Modules](#modules)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Extension Points](#extension-points)
- [Persistence](#persistence)
- [Performance](#performance)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [Code of Conduct](#code-of-conduct)
- [License](#license)
- [Credits](#credits)

---

## Features
- **AI‑driven encounters**: roaming patrols, salvage sites, ambushes, caravans, and world events.
- **Scarcity economy**: resource tiers, dynamic pricing, black‑market vendors, and supply routes.
- **Faction reputation**: territory control, diplomacy, favors, and unlockable services.
- **Automation‑first admin tools**: heatmaps, spawn rules, smart cleanup, and cron‑style schedulers.
- **Modular core**: plug in your own jobs, UIs, inventories, and progression systems.
- **Data‑provider abstraction**: swap between **JSON/Key‑Value** or **SQL** (oxmysql) without touching logic.
- **Event‑based API**: exports & events with a small, predictable surface (server & client).

> *Design target:* survival‑narrative RP with **minimal ceremony** and **no hard dependency** on a specific inventory or UI. Build the dream loop; Deadzone handles the world hum.

---

## Why Deadzone?
Other frameworks focus on city life or heavy economy grind. Deadzone is tuned for **ruined‑world storytelling**:
- **Emergent play over checklists** – procedural events nudge players into stories.
- **Meaningful scarcity** – the economy reacts to server‑wide conditions.
- **Reputation matters** – your choices shift who controls the map and what’s available.
- **Small core, big seams** – add modules without forking the framework.

---

## Architecture
```
resources/
  deadzone-rp-framework/
    core/
      server/        # event bus, scheduler, storage adapter, reputation, economy kernel
      client/        # client event bridge, blips/markers, lightweight UI hooks
      shared/        # typings, constants, enums, config schema
    modules/
      encounters/    # procedural spawners, AI behavior hooks
      economy/       # vendors, crafting, salvage tables, price model
      factions/      # rep curves, ranks, territory, diplomacy
      jobs/          # example jobs (courier, salvage run, patrol)
    config/
      config.lua     # knobs & toggles
      loot_tables.lua
      vendors.lua
    docs/
      guides/        # how-tos & integration notes
      api/           # events/exports reference
```
- **Core** exposes a minimal **event bus** and **exports**.
- **Modules** consume core events and register their own.
- **Storage adapter** (JSON or SQL) is injected via config.

---

## Modules
- **Encounters**: spawn rules (time, biome, heat), AI archetypes, loot rolls, cleanup.
- **Economy**: scarcity index, vendor price curves, crafting recipes, salvage routes.
- **Factions**: rep events, rank thresholds, territory claim/decay, diplomacy state.
- **Jobs (examples)**: courier, escort, salvage run, recon patrol (pure examples; replace with your own).
- **Admin/QoL**: heatmaps, force‑event, fast‑forward time, debug overlays.

---

## Requirements
- **FXServer** (latest recommended)
- **FiveM artifact**: recent
- **Recommended (optional)**: `ox_lib` for notifications/progress, `oxmysql` for SQL provider
- **No hard dependency** on qb‑core/esx; you can integrate if desired.

---

## Installation
1. **Clone / Download** this repository into your `resources/` folder:
   ```bash
   cd resources
   git clone https://github.com/<you>/deadzone-rp-framework.git
   ```
2. **Ensure** the resource in `server.cfg` (load early):
   ```cfg
   ensure deadzone-rp-framework
   ```
3. **Configure** the framework (see [Configuration](#configuration)).
4. **Choose a storage provider** (JSON or SQL) in `config/config.lua`.
5. **Start the server** and watch console for `Deadzone ✔ initialized`.

> If using **SQL**, install and configure **oxmysql** and add your connection string in `server.cfg`.

---

## Configuration
`config/config.lua` (excerpt; values shown are sensible defaults):
```lua
Config = {
  FrameworkName = "deadzone",

  -- Time & world pacing
  DayLengthMinutes = 180,          -- full day/night
  EncounterTickSeconds = 30,       -- core scheduler cadence

  -- Storage provider: "json" | "sql"
  Storage = {
    Provider = "json",
    JsonPath = "data/",          -- used when Provider = json
    Sql = {                       -- used when Provider = sql
      TablePrefix = "dz_",
    }
  },

  -- Economy
  Economy = {
    BasePrice = 1.0,
    ScarcityElasticity = 0.35,     -- 0..1; higher = prices react harder
    VendorRefreshMinutes = 30
  },

  -- Factions
  Factions = {
    DefaultRep = 0,
    GainPerJob = 10,
    LossOnBetrayal = 25,
    TerritoryDecayPerHour = 1,
  },

  -- Encounters
  Encounters = {
    MaxActive = 12,
    Heatmap = { radius = 350.0, cooldown = 900 },
    Biomes = { "urban", "industrial", "wasteland", "forest" }
  },

  -- Integrations
  Integrations = {
    UseOxLib = true,
    InventoryExport = "",       -- e.g., "ox_inventory" or your custom inventory resource
  }
}
```

Additional tables (loot/vendors/crafting) live in `config/loot_tables.lua` and `config/vendors.lua`.

---

## Usage
**Admin commands** (default; secure behind your admin system):
```
/dz.forceevent <encounterId>
/dz.fastforward <minutes>
/dz.heatmap
/dz.spawnvendor <type>
```

**Exports (server)**:
```lua
-- Grant faction reputation
exports['deadzone-rp-framework']:AddFactionRep(playerId, factionId, amount)

-- Start a curated encounter
exports['deadzone-rp-framework']:StartEncounter({
  type = 'ambush',
  biome = 'industrial',
  pos = vec3(123.4, -221.0, 30.1),
  difficulty = 2
})

-- Economy transaction helper
exports['deadzone-rp-framework']:BuyFromVendor(playerId, vendorId, item, amount)
```

**Events (server → client)**:
```lua
-- Notify client about an encounter marker
TriggerClientEvent('dz:client:encounter:marker', playerId, {
  pos = vec3(100.0,100.0,28.0), label = 'Salvage Site'
})
```

**Events (client → server)**:
```lua
-- Player finished salvage
TriggerServerEvent('dz:server:encounter:complete', encounterId, result)
```

> See `docs/api/` for the full list of exports + events (work in progress in alpha).

---

## Extension Points
- **Modules** register via a simple loader:
  ```lua
  -- modules/my_module/server/init.lua
  local MyModule = {}
  function MyModule:Init()
    -- subscribe to bus
    AddEventHandler('dz:bus:tick', function()
      -- do periodic work
    end)
  end
  return MyModule
  ```
- **Encounter providers** implement: `Spawn()`, `OnTick()`, `Cleanup()`.
- **Economy providers** can override price curves: `GetPrice(item)`.
- **Storage providers** implement: `get(key)`, `set(key, value)`, `incr(key, by)`.

Register your module in `fxmanifest.lua` or via the module loader list.

---

## Persistence
Pick one:
- **JSON/Key‑Value (default)** – simple files under `data/`. Good for small/medium servers or quick prototyping.
- **SQL (oxmysql)** – scalable storage, analytics friendly. Enable in config and create the tables with `dz_schema.sql` (provided in `/docs/db/`).

Swapping providers doesn’t change your game logic—only the adapter.

---

## Performance
- All heavy systems run on a **cooperative scheduler** (batched work per tick).
- Spawners use **density clamps & heatmaps** to avoid overpopulation.
- Server → client sync is **delta‑based** where possible.

> Target: maintain headroom on 128‑slot servers with sane encounter limits.

---

## Roadmap
- [ ] Alpha: core bus, JSON storage, baseline encounters/economy/factions
- [ ] Beta: SQL adapter, territory capture, caravan system, vendor UI hooks
- [ ] 1.0: docs site, module template generator, telemetry opt‑in

Have ideas? Open an issue with the **`proposal`** label.

---

## Contributing
PRs welcome! Please:
1. Open an issue to discuss big changes.
2. Follow the existing code style and add docs.
3. Test on a dedicated branch; link videos/logs where relevant.

`/docs/contributing.md` includes guidelines and a module template.

---

## Code of Conduct
This project follows a standard community **Code of Conduct**. By participating, you agree to uphold it. See `/docs/CODE_OF_CONDUCT.md`.

---

## License
**MIT** — see [`LICENSE`](LICENSE).

---

## Credits
Maintainer: **Jinnixx**.
Thanks to the FiveM community, open‑source maintainers, and everyone building wasteland servers that inspired this framework.

> *Topics suggestion:* `fivem` `lua` `rp` `post-apocalyptic` `ai` `automation` `framework` `gta5` `ox_lib` `oxmysql`

