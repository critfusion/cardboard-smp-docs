# Cardboard SMP — Moderator Guide

Welcome. This is everything a moderator needs to know about the plugins running on Cardboard SMP, what commands you have access to, and how to handle common situations.

If you're new, **read the [Common Scenarios](#common-scenarios) section first** — it covers 90% of what you'll actually do day-to-day.

---

## Server basics

- **Address:** `mc.mrcriter.com` (Java port 25565 · Bedrock port 19132)
- **Version:** Paper 1.21.11 with ViaVersion stack (clients 1.7+ all supported)
- **Crossplay:** Bedrock players appear with a `.` prefix on their Gamertag (Floodgate convention)
- **Map viewer:** http://mc.mrcriter.com:8100 (live 3D web map)

---

## Plugin catalog

Everything installed on the server and what it does. If you hit a command you don't know, search this table for the prefix.

| Plugin | Version | Purpose | Common mod commands |
|---|---|---|---|
| **LuckPerms** | 5.5.42 | Permission groups + rank management | `/lp user <name> info`, `/lp user <name> parent set <group>` |
| **Vault** | 1.7.3 | Economy API bridge (needed by Essentials/others) | *(no direct commands)* |
| **EssentialsX** | 2.21.2 | Core mod commands: teleport, mute, kick, balance, etc. | `/tp`, `/kick`, `/mute`, `/tempmute`, `/ban`, `/tempban`, `/warp`, `/fly`, `/heal`, `/feed`, `/invsee` |
| **EssentialsX Chat** | 2.21.2 | Chat formatting + channel prefixes | *(config-driven)* |
| **Geyser-Spigot** | 2.9.5 | Bedrock ⇄ Java protocol bridge | `/geyser list`, `/geyser reload` |
| **Floodgate** | 2.2.5 | Bedrock player auth (no Java account linking needed) | `/floodgate linkaccount` |
| **GriefPrevention** | 16.18.7 | Player land claims | `/abandonclaim`, `/trust`, `/adminclaims`, `/basicclaims`, `/unclaim`, `/ignoreclaim` |
| **WorldEdit** | 7.4.2 | Block editing (for builds) | `//wand`, `//set`, `//copy`, `//paste`, `//undo` |
| **WorldGuard** | 7.0.16 | Region protection (spawn safe zone etc.) | `/rg info`, `/rg list`, `/rg flag <region> <flag> <value>` |
| **CoreProtect** | 23.1 | Block action logging + rollback | `/co i` (inspect), `/co lookup`, `/co rollback`, `/co restore` |
| **DiscordSRV** | 1.30.4 | Discord ⇄ chat bridge | *(pending bot token)* |
| **ViaVersion** | 5.8.1 | Client protocol translation (1.13+ clients) | `/viaversion list`, `/viaversion player <name>` |
| **ViaBackwards** | 5.8.1 | Client protocol translation (1.13–newest) | *(transparent)* |
| **ViaRewind** | 4.0.15 | Legacy client support (1.7–1.12) | *(transparent)* |
| **BlueMap** | 5.16 | Web map at :8100 | `/bluemap reload`, `/bluemap status` |
| **Chunky** | 1.4.40 | World pre-generation for perf | `/chunky start`, `/chunky pause`, `/chunky status` |
| **CardboardStarterKit** | 0.1.0 | First-join starter kit (auto-expires) | *(automatic; no commands yet)* |

---

## Your commands by category

### Teleports & movement (EssentialsX)

| Command | What it does |
|---|---|
| `/tp <player>` | Teleport yourself to a player |
| `/tphere <player>` | Teleport a player to you |
| `/tp <player1> <player2>` | Teleport player1 to player2 |
| `/fly [player]` | Toggle fly mode |
| `/gamemode <creative\|survival\|spectator> [player]` | Change game mode |
| `/speed <1–10> [player]` | Set walk/fly speed multiplier |
| `/back` | Return to your last location before a tp/death |
| `/vanish` | Go invisible (shows for ops only) |
| `/spawn` | Go to world spawn |

### Moderation actions

| Command | What it does |
|---|---|
| `/kick <player> [reason]` | Kick a player (they can rejoin) |
| `/ban <player> [reason]` | Permanent ban |
| `/tempban <player> <duration> [reason]` | Time-limited ban. Duration like `1d`, `6h`, `30m` |
| `/mute <player>` | Mute (can't chat) |
| `/tempmute <player> <duration>` | Time-limited mute |
| `/unmute <player>` | Undo mute |
| `/pardon <player>` | Undo ban |
| `/warn <player> <message>` | Warn (logged) |
| `/jail <player> <jailname> [duration]` | Put in a jail region *(requires a jail set with `/setjail`)* |

**Rule of thumb:** warn → tempmute (1h) → tempban (1d) → perma. Always leave a reason; it shows in logs for the next mod.

### Inventory / item help

| Command | What it does |
|---|---|
| `/give <player> <item> [amount]` | Give an item (EssentialsX wrapper) |
| `/minecraft:give <player> <item>[components] [amount]` | **Bypass Essentials** — required for 1.21 component syntax like enchanted items. Example: `/minecraft:give Arya9295 netherite_sword[enchantments={"minecraft:sharpness":5}] 1` |
| `/invsee <player>` | Open another player's inventory (read-only by default) |
| `/clear <player>` | Empty someone's inventory |
| `/heal [player]` | Full heal |
| `/feed [player]` | Refill hunger |

### Rollback (CoreProtect)

If a player griefs or you need to undo destruction:

1. **Inspect a block:** `/co i` → toggle inspector mode, then left/right-click blocks to see their history.
2. **Rollback:** `/co rollback u:<player> t:<time> r:<radius>`  
   - Example: undo last 6 hours of griefing by `BadPlayer` within 50 blocks of where you're standing: `/co rollback u:BadPlayer t:6h r:50`
3. **Restore** (undo your rollback): same syntax with `/co restore`.

Rollbacks are reversible, but **always announce in mod chat** before running large ones. They pause the server briefly.

### Land claims (GriefPrevention)

Regular players self-claim with a golden shovel. As a mod you can:

| Command | What it does |
|---|---|
| `/adminclaims` | Switch to admin-claim mode (claims won't use your claim-blocks budget) |
| `/abandonclaim` | Abandon the claim you're standing in (dangerous — confirm first) |
| `/deleteclaim` | Admin-delete the claim you're standing in |
| `/ignoreclaims` | Bypass GP restrictions (use sparingly — you can destroy any claim while on) |
| `/trust <player>` | Give a player full build rights in your current claim |
| `/untrust <player>` | Remove trust |
| `/claimslist <player>` | See all claims a player owns |

**If a player reports their claim was griefed:** stand in the claim, `/co lookup t:<time> r:10` to find who did it, then rollback.

### Worldedit (builds)

For spawn builds, fixing griefing, or quick terraforming:

1. `//wand` — gives you the selection wand (wooden axe)
2. Left-click corner 1, right-click corner 2
3. `//set <block>` fills; `//copy` / `//paste` moves; `//undo` reverts

**Please don't WE in areas where players have builds** unless you've checked `/co lookup` first.

### Cardboard SMP specifics

| Feature | Mod-relevant |
|---|---|
| Day cycle | Locked (`gamerule advance_time false`). Don't change; it's intentional while the map is built. |
| Starter kit | Auto-given on first join. Items vanish after 5 min, on drop, on death, or on logout. Configurable in `plugins/CardboardStarterKit/config.yml` on the server. |
| Spawn protection | `spawn-protection=16` in server.properties — 16-block radius around 0,0,0 is build-protected (ops only). Full no-PvP spawn zone is not yet defined in WorldGuard. |
| Bedrock players | Appear with `.` prefix. Commands work the same — `/tp .PlayerName` is valid. |

---

## Common scenarios

### A player can't connect
**Step 1: ask for the exact error message.** That tells you the layer:
- *"Outdated client"* or *"Outdated server"* → they're on a version outside our Via range (extremely rare; we cover 1.7 through 1.21.11).
- *"Failed to verify username"* or *"Invalid session"* → cracked client on our online-mode server. They need a paid Java account.
- *"Connection timed out"* or *"Connection refused"* → network. Have them try `mc.mrcriter.com` again, check firewall.
- *Missing items / invisible blocks* (connected but broken) → **they're on an old client**, some items (grindstone added 1.14, netherite added 1.16) don't exist in their client's registry. They need to upgrade their launcher profile.

### A player is asking for items
Prefer **not** handing out items. If you do:
- Admin-grant items use `/minecraft:give <player> <item> <amount>`.
- Enchanted items: use component syntax — see the Inventory table above.
- Always log what you gave in the mod channel.

### A player is griefing / harassing
1. `/mute <player>` to stop chat harm immediately
2. Gather evidence: screenshots or `/co lookup` for block damage
3. If confirmed: `/tempban <player> 1d <reason>` (first offense) or `/ban <player> <reason>` (repeat)
4. Document in mod log channel with player name, timestamp, evidence

### A new player is confused
The welcome kit gives them armor + sword for 5 min. After that they're on their own. Direct them to:
- `/spawn` — get back to spawn
- `/sethome` — set a home location (limit configured in EssentialsX)
- `/home` — teleport back
- `/help` — full list of commands they can use
- `/rules` — server rules (if set; configure via EssentialsX `/rules` info file)

### Someone's build got griefed
1. Stand in the affected area.
2. `/co lookup r:50 t:24h` → shows last 24h of block actions in a 50-block radius.
3. Identify the griefer.
4. `/co rollback u:<griefer> t:24h r:50` to undo.
5. Ban/warn the griefer per the "griefing" scenario.

### Server feels laggy
1. Check TPS: `/tps` (should be ~20)
2. Check mspt (ms per tick): `/mspt` (anything over 50 is bad)
3. If lag: check `/chunky status` — pre-gen could be running. `/chunky pause` if active.
4. Check number of entities: `/lagg` (requires NoLagg) — we don't have it installed but Paper has a built-in `/paper entity list <world> <radius>` to find entity clusters.
5. Report to admin channel with the numbers.

### Crossplay (Bedrock) issues
- Bedrock players use the **same address** (`mc.mrcriter.com`) but **port 19132**.
- Console players (Xbox/PS/Switch) usually need BedrockConnect as a DNS workaround.
- If a Bedrock player can't see certain items/blocks: they're real-time translated by Geyser; some things just don't exist on Bedrock (check the [Geyser limitations list](https://geysermc.org/wiki/geyser/current-limitations)).

---

## When to escalate

Don't hesitate to escalate to admin (@mrcriter) if:
- You're unsure whether an action is reversible
- The issue involves real-world threats, doxing, or illegal content
- The server appears to be under attack (many accounts joining rapidly, all getting kicked)
- A plugin error appears in chat repeatedly
- You need to restart the server (mods can't do this; admin has `sudo cardboard-cmd ...` access)

---

## Appendix: what mods *can't* do (requires admin)

- Restart/stop the server (`systemctl`)
- Install, remove, or update plugins
- Edit plugin config files
- Change Paper/server configs (`server.properties`, `paper-global.yml`, etc.)
- Set up new WorldGuard regions
- Reset player data (delete `playerdata/<uuid>.dat`)
- Give themselves permanent creative or op flags
- Access backups or the 1Password vault

If you feel strongly one of these should be a mod-level power, open an issue in this repo and we'll discuss.
