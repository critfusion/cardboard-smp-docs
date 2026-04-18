# Cardboard SMP — Admin Runbook

Ops-level reference. Mods don't need this; admins (@mrcriter) do.

For the mod-facing doc see [MODERATORS.md](MODERATORS.md).

---

## Host & access

- **VPS:** Contabo, IP `154.38.167.238` (`vmi2512680.contaboserver.net`)
- **SSH:** `ssh -i ~/.ssh/scout-infrastructure scout@154.38.167.238`
- **Spec:** 6 vCPU AMD EPYC · 11 GB RAM · 96 GB disk
- **Cost:** see 1P item `Minecraft Server (mc.mrcriter.com)` in `CritFusion Infra`

Apache is also on this box serving `mrcriter.com` on 80/443 — unrelated to MC, leave alone.

---

## Service control

```bash
# status
sudo systemctl status cardboard.service
sudo journalctl -u cardboard.service -f      # follow live logs
sudo journalctl -u cardboard.service -n 500  # last 500 lines

# lifecycle
sudo systemctl restart cardboard.service
sudo systemctl stop cardboard.service
sudo systemctl start cardboard.service

# disable/enable at boot
sudo systemctl disable cardboard.service
sudo systemctl enable cardboard.service
```

SIGTERM triggers Paper's shutdown hook (save-all + stop). `TimeoutStopSec=120` gives it 2 minutes to flush cleanly.

---

## Running console commands

The service runs with no stdin attached. Commands are sent via a named-pipe FIFO at `/run/cardboard/in`:

```bash
sudo cardboard-cmd <any minecraft console command>
# e.g.
sudo cardboard-cmd say Hello everyone
sudo cardboard-cmd time set day
sudo cardboard-cmd op SomePlayer
sudo cardboard-cmd lp user SomePlayer parent set admin
```

The wrapper is at `/usr/local/bin/cardboard-cmd`.

---

## Paths

```
/home/minecraft/cardboard/                 install root
├── paper.jar                              Paper 1.21.11 build 69
├── start.sh                               launcher with Aikar flags (6G heap)
├── .paper-build                           version pin (used by updater)
├── server.properties
├── ops.json                               operator list
├── whitelist.json
├── world{,_nether,_the_end}/              world data
├── config/                                Paper config files
│   └── paper-world-defaults.yml           anti-xray lives here
├── plugins/                               17 plugins
│   ├── CardboardStarterKit.jar            our custom plugin
│   ├── BlueMap.jar
│   └── ...
├── libraries/                             plugin-shared JARs
├── logs/                                  file logs (also in journalctl)
└── backups/                               nightly local world backups (7-day retention)

/etc/systemd/system/cardboard.service      systemd unit
/usr/local/bin/cardboard-cmd               FIFO command wrapper
/usr/local/bin/cardboard-backup.sh         nightly backup script
/run/cardboard/in                          console stdin FIFO (created per boot)
```

---

## JVM settings

Paper starts with Aikar's flags at **6 GB heap**. Host has 11 GB — leaves ~5 GB for OS, Apache, Geyser non-heap, JVM metaspace, disk cache. Under load this is comfortable for ~30-40 CCU. For the PRD target of 100 CCU, the box needs to be resized to 16 GB and heap bumped to 10 GB.

Flags live in `/home/minecraft/cardboard/start.sh`. Standard Aikar set, change `HEAP=6G` to bump.

---

## Plugins

### Installing a new plugin

```bash
# as scout on the VPS:
cd /home/minecraft/cardboard/plugins
sudo -u minecraft curl -fsSL -o MyPlugin.jar "<download url>"
sudo systemctl restart cardboard.service
# verify it loaded:
sudo journalctl -u cardboard.service -n 200 | grep -E "MyPlugin|Enabling"
```

### Removing a plugin

Stop the server first (some plugins leave state otherwise):

```bash
sudo systemctl stop cardboard.service
sudo rm /home/minecraft/cardboard/plugins/<Plugin>.jar
# optional: remove its config/data dir
sudo rm -rf /home/minecraft/cardboard/plugins/<Plugin>/
sudo systemctl start cardboard.service
```

### Updating a plugin

Same as install — replace the jar and restart. Keep the old jar as `<Name>.jar.old` for rollback until you confirm the new version works.

### Custom plugins (this repo)

`cardboard-starter-kit` and future custom plugins live here:
https://github.com/critfusion/cardboard-smp-plugins

Build locally:
```bash
cd ~/repos/cardboard-smp-plugins
mvn -B package
# jar lands at cardboard-starter-kit/target/cardboard-starter-kit-<version>.jar
```

To deploy:
```bash
scp cardboard-starter-kit/target/cardboard-starter-kit-*.jar scout@154.38.167.238:/tmp/
ssh scout@154.38.167.238 'sudo mv /tmp/cardboard-starter-kit-*.jar /home/minecraft/cardboard/plugins/CardboardStarterKit.jar && sudo chown minecraft:minecraft /home/minecraft/cardboard/plugins/CardboardStarterKit.jar && sudo systemctl restart cardboard.service'
```

GitHub Actions builds every push; tagged releases attach the jar to the GitHub Release.

---

## Backups

**Local, nightly:** cron at 04:30 runs `/usr/local/bin/cardboard-backup.sh`, tars the three world dirs, keeps 7 days at `/home/minecraft/cardboard/backups/`.

**Pre-migration:** the 4 Bedrock servers + old Vanilla Java are archived at `~/backups/mc-mrcriter/pre-craftsmp-20260418/` on the admin local machine (`/home/scout/`).

**Not yet done:** offsite backups. B2/S3 push is a good next step — see PRD.

---

## Monitoring

No dedicated observability yet. Ad-hoc:
- TPS: `sudo cardboard-cmd tps` or `/tps` in-game
- MSPT: `/mspt` in-game
- Memory: `free -h` on host
- Disk: `df -h /` on host
- Live logs: `sudo journalctl -u cardboard.service -f`
- Player count: `sudo cardboard-cmd list`

---

## Secrets

All project secrets go in **1Password → CritFusion Infra**:
- `Minecraft Server (mc.mrcriter.com)` — connection info, admin notes
- *(planned)* Discord bot token for DiscordSRV
- *(planned)* Tebex API secret
- *(planned)* RCON password (if we enable RCON)

**Policy:** pre-create 1P items before asking users to paste values into third-party dashboards. Don't ever put secrets in git, config comments, or chat.

---

## Deploying changes

Pre-authorized (no confirmation needed):
- Restarting `cardboard.service`
- Editing config files under `/home/minecraft/cardboard/`
- Dropping new plugin jars into `plugins/`
- Running `cardboard-cmd` for console admin actions
- Creating/pushing commits to this repo

Explicitly requires asking the user:
- Changing plugin set that affects gameplay (adding anticheat, removing a plugin players depend on)
- Stopping the server during active playtime
- Wiping world data
- Resizing the VPS
- Changing DNS (`mc.mrcriter.com` A record)

---

## Troubleshooting cheats

| Symptom | First check |
|---|---|
| Server won't start | `sudo journalctl -u cardboard.service -n 200` for the stack trace |
| Plugin won't load | Usually a version mismatch. Look for `UnsupportedClassVersionError` (plugin built for newer Java than host JVM) or `InvalidPluginException` (missing dep) |
| Port not accepting connections | `sudo ss -tlnp \| grep 25565` — Paper should own it. If not, the JVM crashed. |
| Paper says "Incorrect argument for command" on known commands | In 1.21 some commands were renamed to snake_case with `minecraft:` namespace. E.g. `doDaylightCycle` → `minecraft:advance_time`. |
| BlueMap map is stale | `sudo cardboard-cmd bluemap reload` or wait — BlueMap renders in background on chunk updates |
| Chunky pre-gen is slow / stuck | `/chunky status`. Pause with `/chunky pause` if it's lagging live players. |

---

## Known issues / TODO

- [ ] WorldGuard spawn safe zone not yet defined (needs WorldEdit selection from an in-game op)
- [ ] DiscordSRV not configured (bot token pending)
- [ ] Grim AntiCheat not installed (no reliable mirror found; try again later)
- [ ] Citizens not installed (CI was flaky on install day)
- [ ] Memory bumping to 10G heap requires VPS resize to 16 GB before 100 CCU
- [ ] Offsite backup not set up
- [ ] Custom PRD plugins (CraftSpawners, CraftCombat, CraftKills) not yet built

