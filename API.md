# SN2 Cheat — DLL API

`SN2Cheat.dll` exposes a simple, language-agnostic **local IPC API** over a Windows
named pipe. Once the DLL is injected into Subnautica 2, you can build your own
frontend / tool (Python, C#, Node, a Discord bot, etc.) that drives it — no need to
touch the DLL internals.

Built and maintained by **zxkuhl**. You may build tools on this API, but credit is
required and you may not claim `SN2Cheat.dll` or this API as your own original work.
(Every `status` response includes `"author":"zxkuhl"`.)

---

## Connecting

- **Pipe:** `\\.\pipe\sn2cheat`
- **Mode:** byte stream, UTF-8, newline (`\n`) delimited
- The pipe only exists **after the DLL is injected** and the game is running.
- Only **one** client at a time (single instance).
- On connect, the server immediately sends one `status` JSON line.

**Protocol:** send a command as a single `\n`-terminated line. The server replies with
exactly **one JSON line** per command:
- `pings` → a pings object
- `players` → a players object
- `esp` → an ESP object (on-screen player boxes)
- everything else → a fresh `status` object

Poll `status\n` (e.g. once per second) to keep live state.

---

## Commands

### State
| Command | Description |
|---|---|
| `status` | Returns the status object (below). |
| `players` | Returns the connected-players list. |
| `pings` | Returns the signal/beacon list. |
| `esp` | Returns projected on-screen boxes for all other players (below). |

### Cheat toggles
| Command | Description |
|---|---|
| `set god 0\|1` | God mode (self). |
| `set o2 0\|1` | Infinite oxygen. |
| `set food 0\|1` | No hunger/thirst. |
| `set speed 0\|1` | Speed hack. |
| `set bright 0\|1` | Full bright (zero fog). |
| `set fly 0\|1` | Fly / noclip. |
| `toggle <name>` | Flip any of the above (`god`,`o2`,`food`,`speed`,`bright`,`fly`). |
| `speedmult <1..30>` | Speed multiplier. |

### Camera
| Command | Description |
|---|---|
| `fov <30..170>` | Lock the camera field of view to this value. |
| `fov off` | Release the FOV lock back to the game. |

### Teleport
| Command | Description |
|---|---|
| `tp <x> <y> <z>` | Teleport self to world coordinates. |
| `savepos` | Save your current position. |
| `tpsaved` | Teleport to the saved position. |
| `tppod` | Teleport to the lifepod. |

### Players (host-authoritative)
| Command | Description |
|---|---|
| `tpplayer <name\|id>` | Teleport yourself to a player. |
| `kill <name\|id>` | Deal lethal damage to a player. |
| `health <name\|id> <amount>` | Set a player's health. |
| `oxygen <name\|id> <amount>` | Set a player's oxygen. |
| `food <name\|id> <amount>` | Set a player's food. |
| `water <name\|id> <amount>` | Set a player's water. |

### Misc
| Command | Description |
|---|---|
| `run <console command>` | Run a game engine console command (e.g. `run stat fps`). |
| `notify <text>` | Show an in-game notification toast. |
| `bind <god\|o2\|food\|all\|speed> <vkCode>` | Rebind a hotkey (0 = unbound). |
| `console 0\|1` | Hide/show the cheat's console window. |

### Name spoof
| Command | Description |
|---|---|
| `spoofname <text>` | Replace **your** displayed name with `text`. |
| `spoofname all <text>` | Replace **every** player's displayed name (your view). |
| `spoofname off` | Restore the real name(s). |

> The spoof is written into the existing name buffer, so `text` is truncated to the
> length of the real name. It's client-side unless you're the host; some UI elements
> cache the name and may not refresh immediately.

---

## Response schemas

### `status`
```json
{
  "author": "zxkuhl",
  "version": "1.1.0",
  "latest": "1.1.0",
  "outdated": false,
  "ready": true,
  "god": false, "o2": true, "food": false,
  "speed": false, "speedmult": 3.0,
  "bright": false, "fly": false,
  "fov": false, "fovval": 90,
  "px": -337218.3, "py": 433434.7, "pz": 234.6,
  "saved": false,
  "binds": { "god": 112, "o2": 113, "food": 114, "all": 115, "speed": 116 }
}
```
- `ready` — true once you're in a world (attribute classes resolved).
- `outdated` — true when the DLL version doesn't match the remote version; while
  outdated the DLL blocks all mutating commands.
- `fov` — whether a custom FOV lock is active; `fovval` — the locked value.
- `px/py/pz` — your live world position (Unreal units).
- `binds` — hotkey virtual-key codes.

### `players`
```json
{ "players": [ { "name": "JaydenMavric", "id": 0 }, { "name": "Wu", "id": 1 } ] }
```

### `pings`
```json
{ "pings": [ { "name": "Lifepod", "x": -337218.3, "y": 433434.7, "z": 234.6 } ] }
```

### `esp`
```json
{
  "esp": {
    "vw": 1920, "vh": 1080,
    "players": [
      { "name": "Wu", "hp": 0.85, "dist": 42, "x": 910, "y": 400, "w": 60, "h": 150 }
    ]
  }
}
```
- `vw/vh` — the game's viewport size the boxes were projected against; scale the
  coordinates to your own canvas by `yourWidth/vw` × `yourHeight/vh`.
- Per player: `name`, `hp` (0–1 fraction of max health), `dist` (metres), and the
  screen-space box `x,y` (top-left) `w,h`. Only players currently on screen appear.

---

## Example (Python)

```python
import json, time

pipe = open(r'\\.\pipe\sn2cheat', 'r+b', buffering=0)

def send(cmd):
    pipe.write((cmd + '\n').encode())
    return json.loads(pipe.readline().decode())

print(send('status'))          # current state
send('set god 1')              # god mode on
send('tp 1000 2000 300')       # teleport
print(send('players'))         # list players
```

> The DLL must already be injected (use the injector / the SN2 Cheat app) before
> connecting. Mutating commands are ignored while `outdated` is true.
