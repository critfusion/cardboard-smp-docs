# Chariot SMP — Moderator Guide

Welcome. This is everything a moderator needs to know about the plugins running on Chariot SMP, what commands you have access to, and how to handle common situations.

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
| **ChariotStarterKit** | 0.1.0 | First-join starter kit (auto-expires) | *(automatic; no commands yet)* |
| **ChariotBuildAI** | 0.1.0 | Natural-language builder (Claude API) | `/ai <prompt>`, `/ai <model> <prompt>`, `/ai undo`, `/ai models` |
| **ChariotSafeRooms** | 0.1.0 | Permanent bedrock-layer safe rooms + decor box + admin-vault chest tooling | `/safe`, `/saferoom info`, `/saferoom assign\|unassign\|list\|reload\|decortemplate\|givedecor`, `/adminvault place\|list` |
| **ChariotKeyShop** | 0.1.0 | Key-tier loot shop, chars currency, chest-linked tier shops | `/keys`, `/keyshop`, `/chars`, `/buykey`, `/keyshop <tier>template`, `/keyshop link\|unlink` |
| **ChariotGoldenEgg** | 0.1.0 | Random treasure-hunt event — chest spawns somewhere on the map every 30-90 min | `/goldenegg status\|drop\|cancel\|reload` |
| **ChariotProtect** | 0.1.0 | Op-friendly per-block protection toggle | `/protect on\|off\|area\|remove\|status` |

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

### AI builder (`/ai`)

Describe a structure in English; the server calls the Claude API and places blocks around you. Builds drop relative to where you're standing.

| Command | What it does |
|---|---|
| `/ai <description>` | Build using the **default model** (Sonnet 4.6 — cheap, good for small builds) |
| `/ai opus <description>` | Use **Claude Opus 4.7** — best quality, best for complex/large builds, ~5× the cost |
| `/ai sonnet <description>` | Force Sonnet explicitly |
| `/ai haiku <description>` | Use **Haiku 4.5** — fastest & cheapest, lower quality |
| `/ai undo` | Revert your most recent build (keeps a 5-deep history) |
| `/ai models` | Show the current default model + alias list |

**Examples:**
```
/ai a small oak cabin with a porch and a chimney
/ai opus a detailed medieval watchtower 15 blocks tall with battlements
/ai haiku a 20x20 cobblestone patio
/ai undo
```

**Limits:**
- Max build size: 48 blocks on each axis (the model is told this).
- Per-player cooldown: 30 seconds between builds.
- Only 2 builds can run concurrently across the whole server.
- Ops bypass cooldown + concurrency cap (but not size).

**Costs (rough):**
| Model | Per small build | Per detailed build |
|---|---|---|
| Haiku | ~$0.005 | ~$0.02 |
| Sonnet *(default)* | ~$0.02–0.08 | ~$0.10 |
| Opus | ~$0.10 | ~$0.20–0.40 |

**When builds look bad:** `/ai undo`, reword the prompt more specifically, try a different model. Opus is notably better for anything involving structure (roofs, stairs, multi-room interiors). Sonnet is fine for simple geometric builds. Prompts with **dimensions** (e.g. "7 blocks wide, 5 tall") produce tighter results than vague ones.

**Safety:** banned-block list is enforced server-side (bedrock, barriers, command blocks, TNT, lava, spawners — all rejected). Model is instructed not to generate these, but placements are filtered either way.

**Costs are tracked on the Anthropic console** (console.anthropic.com → Usage). If you see runaway spend, tell admin to cap the key.

### Chariot SMP specifics

| Feature | Mod-relevant |
|---|---|
| Day cycle | Locked (`gamerule advance_time false`). Don't change; it's intentional while the map is built. |
| Starter kit | Auto-given on first join. Items vanish after 5 min, on drop, on death, or on logout. Configurable in `plugins/ChariotStarterKit/config.yml` on the server. |
| AI builder | `/ai` calls the Claude API to place blocks. Ops-only by default. See the AI builder section above. |
| Spawn protection | `spawn-protection=16` in server.properties — 16-block radius around 0,0,0 is build-protected (ops only). Full no-PvP spawn zone is not yet defined in WorldGuard. |
| Bedrock players | Appear with `.` prefix. Commands work the same — `/tp .PlayerName` is valid. |

### Safe Rooms (`/safe`, `/saferoom`, `/createdecorchest`)

ChariotSafeRooms manages the 10×10×10 private rooms players buy on the Tebex store. Rooms live at the bedrock layer (y=−63..−52). Walls are indestructible (BlockBreak/Place/Explode/Piston/LiquidFlow all cancelled by the plugin — even ops can't break them). Owners can build inside their own room; non-owners cannot place blocks there. PvP is disabled inside any safe room.

There's only one purchasable size (10×10×10). The plugin still supports `medium`/`large` tiers internally for special hand-crafted rooms (e.g., `custom_ethan`), but those aren't sold.

| Command | Who | What it does |
|---|---|---|
| `/safe` | Anyone | Teleport to your owned room. Tells you "buy one at the store" if you don't own one. |
| `/saferoom info` | Anyone | Print your room id and spawn coords. |
| `/saferoom list` | Admin | Tier counts: how many rooms exist and how many are owned. |
| `/saferoom assign <player>` | Admin | Pull the next free safe room and bind it to the player. This is what the Tebex on-purchase command runs. |
| `/saferoom unassign <player>` | Admin | Release a player's room back to the free pool. |
| `/saferoom reload` | Admin | Re-read `rooms.yml` + `owners.yml` from disk. |
| `/saferoom decortemplate` | Admin | Look at a chest within 8 blocks; saves its contents as the Decor Box recipe (`plugins/ChariotSafeRooms/decor_template.yml`). Re-run anytime to update. |
| `/saferoom givedecor <player> [--force]` | Admin | Mint a tagged Decor Box shulker (populated from the template) and add it to the player's inventory. Refuses if they don't own a safe room or already received one (override with `--force`). |
| `/createdecorchest` | Admin | **Deprecated** — alias for `/adminvault place decorchest`. Kept for back-compat. |

**Decor box restrictions** (auto-enforced once a player has one):
- The Decor shulker can only be **placed inside the owner's own safe room interior** — anywhere else, BlockPlaceEvent is cancelled.
- Decor items **cannot be dropped** (`PlayerDropItemEvent` cancelled).
- Decor items **cannot enter chests, hoppers, the Auction House, shops, anvils, or crafting** — only the owner's inventory and their Decor shulker GUI accept them.
- Every 5 seconds the plugin scans for decor items in players who are outside their room and silently strips them (the box itself is kept so they can relocate).

**Custom-spawn rooms** (e.g. `custom_ethan`): a player can have their `/safe` point to any coordinate by hand-editing `plugins/ChariotSafeRooms/rooms.yml`. Used when the owner built their own private room aboveground and we just want `/safe` to teleport them there.

**Admin sky vault** at `0, 309, 0`: a 12×12×12 stone-brick room at world ceiling holding the keystore template chests. Marked as room id `admin_vault` with a 1×1 dummy interior, so the entire envelope is treated as wall and is unbreakable by anyone.

### Key Shop (`/keys`, `/keyshop`, `/chars`, `/buykey`)

ChariotKeyShop runs the four-tier crate-key economy: **Common / Gold / Prime / Mythical**. Keys are tagged tripwire-hook items in player inventories; chars (`₵`) are the in-game currency that converts to keys.

| Command | Who | What it does |
|---|---|---|
| `/keys` | Anyone | Show your key balance |
| `/keys give <player> <tier> [n]` | Admin | Give n keys of a tier |
| `/keys reload` | Admin | Re-read `config.yml` |
| `/keyshop` | Anyone | Open the tier picker GUI |
| `/keyshop <common\|gold\|prime\|mythical>` | Anyone | Open one tier directly |
| `/buykey <tier> [amount]` | Anyone | Convert chars → keys (confirmation GUI) |
| `/chars` | Anyone | Show your ₵ balance |
| `/chars give\|take\|set <player> <amount>` | Admin | Adjust balances |
| `/chars top` | Anyone | Top-10 leaderboard |

**Loot templates** — define what a tier rewards by snapshotting a chest:

1. Fill any chest, trapped chest, barrel, or shulker with the items you want in that tier.
2. Look at the container (within 8 blocks) and run **one** of:
   - `/keyshop commontemplate`
   - `/keyshop goldtemplate`
   - `/keyshop primetemplate`
   - `/keyshop mythicaltemplate`
3. Saved to `plugins/ChariotKeyShop/templates/<tier>.yml` and reloaded immediately. The next time someone opens `/keyshop <tier>` they see your snapshot — full ItemStack with enchants, trims, and components preserved.

To revert a tier to the legacy `tiers:` config in `config.yml`: delete the corresponding `templates/<tier>.yml` and `/keys reload`.

**Chest-linked tier shops** — turn a physical chest into a portal that opens the keyshop GUI:

| Command | What it does |
|---|---|
| `/keyshop link <tier>` | Admin: while looking at a chest, links its block coords to the tier. Right-clicking that chest now opens `/keyshop <tier>` instead of the chest's vanilla inventory. Persisted in `chest_links.yml`. |
| `/keyshop unlink` | Admin: removes the link from the chest you're looking at. |

Used at the admin sky vault to make the four template chests double as a working storefront preview.

### Admin Sky Vault — at `0, 309, 0`

A 12×12×12 stone-brick room at the world ceiling that holds the canonical template chests. **Indestructible** — no player or op can break it (registered as `admin_vault` in `rooms.yml` with a 1×1 dummy interior, so the entire envelope is treated as a wall and the ChariotSafeRooms wall-protection handlers cancel every break/place/explode/piston/liquid-flow event inside).

**Get there:** `/tp 0 309 0` (op) — no warp set up yet, ask if you want one.

**What's inside:**

| Chest | Purpose | Linked to |
|---|---|---|
| **COMMON** at (-4, 309, -3) | Loot template for the Common key | `/keyshop common` (right-click opens shop GUI) |
| **GOLD** at (-2, 309, -3) | Loot template for the Gold key | `/keyshop gold` |
| **PRIME** at (0, 309, -3) | Loot template for the Prime key | `/keyshop prime` |
| **MYTHICAL** at (2, 309, -3) | Loot template for the Mythical key | `/keyshop mythical` |
| **DECOR** (when added) at (4, 309, -3) | Loot template for the Decor Box shulker | Used by `/saferoom givedecor` |

**Day-to-day workflow inside the vault — use `/adminvault`:**

| Command | What it does |
|---|---|
| `/adminvault place <type>` | Look at a floor block; chest+sign appear above it. Existing chest+sign of the same type are removed first (so it acts like a "move"). Tier types auto-link to `/keyshop <tier>`. Aliased `/av`. |
| `/adminvault list` | Show currently tracked locations of all admin-vault chests. |

Types: `decorchest` · `goldenegg` · `common` · `gold` · `prime` · `mythical`. Contents come from the matching template (`templates/<tier>.yml`, `decor_template.yml`, or the goldenegg `item-pool` config).

Other workflows:

1. **Edit a tier's loot:** open the relevant chest, change items, look at it, run `/keyshop <tier>template` (e.g. `/keyshop goldtemplate`). Saves to `plugins/ChariotKeyShop/templates/<tier>.yml` and reloads instantly. Right-clicking the linked chest now serves the new loot.
2. **Edit decor box loot:** open the decor chest, change items, run `/saferoom decortemplate`. Then `/saferoom givedecor <player>` mints a tagged shulker for them.
3. **Manual chest linking** (rare — `/adminvault place` does this automatically): look at a chest, run `/keyshop link <tier>` to wire its right-click into the keyshop GUI. `/keyshop unlink` to undo. Stored in `chest_links.yml`.

### Golden Egg event (`/goldenegg`)

A random treasure-hunt event. Every 30-90 minutes a chest spawns at a random surface location somewhere within ±5000 of origin (excluding the central ±600 safe-rooms exclusion zone). World chat broadcasts the coords + contents. First player to right-click claims it.

**Mechanics:**
- Initial loot pool: one of `egg`, `cooked beef (renamed "Hearty Steak")`, or `netherite pickaxe`. Item is randomly picked at the start of a cycle.
- Box starts with 10 chars in addition to the item.
- Right-click claim → player gets the item + chars (granted via ChariotKeyShop's `addChars` API). Cycle resets.
- If unclaimed for 30 minutes, the chest disappears, +10 chars accumulate (item persists), 30-min cooldown, then redrops at a new location.
- Chest is unbreakable until claimed (ops included). Pistons/explosions/lava can't move or destroy it.
- State persists across server restarts (`plugins/ChariotGoldenEgg/state.yml`).

**Admin commands:**

| Command | What it does |
|---|---|
| `/goldenegg status` | Show whether the egg is active, current location/chars/item if so |
| `/goldenegg drop` | Force an immediate drop (ignored if already active) |
| `/goldenegg cancel` | Remove the active egg without paying out chars; schedules next drop |
| `/goldenegg reload` | Reload `plugins/ChariotGoldenEgg/config.yml` |

The item pool is editable in `plugins/ChariotGoldenEgg/config.yml` → `item-pool`. A `/goldenegg savetemplate` command (snapshot a chest to populate the pool) and `/goldenegg place` (spawn a reference chest in the admin vault) are planned but not implemented yet.

### Block protection (`/protect`)

ChariotProtect tracks individual block coordinates as "protected" — break / explosion / piston / lava-flow events on those blocks are cancelled at HIGH event priority (so even ops have to `/protect remove` first). Two ways to mark blocks:

| Command | What it does |
|---|---|
| `/protect on` | Toggle ON: every block YOU place from now on is auto-protected. Action-bar shows `✓ block protected` on each placement. |
| `/protect off` | Toggle OFF — placements stop auto-protecting. |
| `/protect area` | Protects every non-air block inside your current WorldEdit selection. Make a selection first: hold a wooden axe (`//wand`), left-click corner 1, right-click corner 2, then `/protect area`. |
| `/protect remove` | Un-protect the block you're looking at (within 8 blocks). |
| `/protect status` | Show current toggle state + total protected-block count. |

Persisted in `plugins/ChariotProtect/protected_blocks.yml`. Permission: `chariot.protect.use` (op default). For larger admin-managed areas (entire builds, claims), prefer **WorldGuard regions** (`//wand` → `/rg define <name>` → `/rg flag <name> build deny`) — that's what the spawn region, the Colosseum, the floating islands, and the chariot monument all use.

### Random teleport (`/rtp`)

Aliased to Essentials' native `/tpr`. Center: (0, 0). Min range: 600 blocks (clears the safe-rooms exclusion). Max range: 5000 (well inside the 10,100-block world border). Excludes ocean/river biomes by default. Permission `essentials.tpr` is granted to the `default` group, so anyone can use it.

To change the bounds, run in-game: `/settpr center <x> <z>`, `/settpr minrange <n>`, `/settpr maxrange <n>`. Console can't run `/settpr` (it requires a player context).

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
- You need to restart the server (mods can't do this; admin has `sudo chariot-cmd ...` access)

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
