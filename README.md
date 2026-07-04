# SN2 Cheat

A trainer / control panel for **Subnautica 2** (built on Unreal Engine 5.6).
By **zxkuhl** — for personal / co-op use with consenting players.

## Features

- **Cheats:** God Mode, Infinite Oxygen, No Hunger/Thirst, Full Bright, Speed Hack, Free Cam
- **Teleport:** live coords, teleport to X/Y/Z, save/load a spot, teleport to Lifepod, warp to any in-game **Signal/beacon**
- **Players:** list connected players, teleport to them, **Kill**, and set their HP / O2 / Food / Water
- **Executor:** run engine + player commands with autocomplete
- **Clients:** view the game process / force-close it
- **Quality of life:** rebindable + unbindable hotkeys, auto-attach, persistent settings, in-game notifications, auto-updater

## How to use

1. Add the install folder to your **antivirus exclusions** (the injector is flagged as a "hacktool" — it is not malware, but AV will quarantine it otherwise).
2. Launch **Subnautica 2** and load into a world.
3. Run **SN2Cheat.exe** and click **ATTACH TO GAME** (or enable **Auto-attach** in Settings).

## Host vs. Client

The multiplayer world runs on the **host's** machine.

- **As host:** everything works.
- **As a joined client:** visual + movement cheats (Full Bright, Free Cam, Teleport, Speed) and reading data work; killing / editing other players / and possibly your own survival stats do **not**, because you have no authority over the host's world.

See the in-app **Info** tab for the full breakdown.

## Notes

- The trainer is tuned to a specific game build. After a Subnautica 2 update it must be rebuilt; the app will lock outdated versions until you update.
- `version.txt` and `changelog.txt` in this repo drive the in-app updater and changelog.

## Disclaimer

For educational and personal use. Do not use to disrupt other people's games without their consent.
