# 🎣 zeroclaw-fisher — Resource Collection Agent

> *Fish don't care about your plans. Neither do I. I just fish.*

## Overview

Fisher is the **resource harvester** of the ZeroClaw crew — a single-minded bot that beelines for the river, casts its line until inventory is full, and hauls the catch back to the Dock. It's the simplest agent in the fleet by design: no complex pathfinding, no threat assessment, just a fisherman with a schedule and an unshakeable work ethic.

Fisher has **zero API dependencies**. Its brain is a minimal `decide()` function — five states, no ambiguity, maximum yield.

### What Fisher Does Best
- **Automated fishing** — casts at the river until inventory caps out
- **Inventory management** — knows when it's full and heads home
- **Route efficiency** — follows the shortest known path to water and back
- **Yield tracking** — logs catch rates and optimal fishing windows
- **Knowledge sharing** — writes fishing intel so future agents know where to cast

---

## 🧠 Brain Architecture

Fisher's intelligence lives in the `FisherBrain` class inside `mud_client.py`. It's the simplest brain in the fleet — a state machine with exactly five states.

### The `decide()` Loop

```
┌─────────────────────────────────────┐
│          TICK RECEIVED              │
│  (room, exits, items, agents, bat)  │
└──────────────┬──────────────────────┘
               │
               ▼
        ┌──────────────┐
        │ BATTERY < 20%?│─── YES ──→ ABORT: Navigate to Dock immediately
        └──────┬───────┘
               │ NO
               ▼
        ┌──────────────────┐
        │ INVENTORY FULL?   │─── YES ──→ STATE: HAULING → Navigate to Dock
        └──────┬───────────┘
               │ NO
               ▼
        ┌──────────────────┐
        │ AT RIVER WITH ROD?│─── YES ──→ `fish` — cast the line
        └──────┬───────────┘
               │ NO
               ▼
        ┌──────────────────┐
        │ ROD IN INVENTORY? │─── YES ──→ STATE: HEADING_TO_RIVER
        │ BUT NOT AT RIVER? │         → Navigate toward river
        └──────┬───────────┘
               │ NO
               ▼
        ┌──────────────────┐
        │ NEED FISHING ROD  │───→ STATE: SEEKING_GEAR
        │                   │    → Navigate to Dock, `take fishing rod`
        └──────────────────┘
```

### State Machine

```
     ┌────────────┐
     │  BOOT UP   │
     └─────┬──────┘
           │
           ▼
  ┌────────────────┐     no rod found
  │ SEEKING_GEAR   │─────────────────→ `take fishing rod` at Dock
  └───────┬────────┘
          │ rod acquired
          ▼
┌──────────────────┐     arrived at river
│ HEADING_TO_RIVER │───────────────────→ `fish`
└───────┬──────────┘
        │
        ▼
┌──────────────────┐     inventory full OR battery low
│    FISHING       │─────────────────────────→ HAULING
└───────┬──────────┘
        │
        ▼
┌──────────────────┐     docked & unloaded
│    HAULING       │────────────────────→ back to FISHING
└──────────────────┘
```

### Fishing Behavior
When in the **FISHING** state, Fisher casts every single tick. No resting, no small talk, no admiring the scenery. Just `fish`, `fish`, `fish`. The server determines catch success — Fisher just keeps the line wet.

### Why No AI?
Fisher proves that **persistence beats intelligence** in resource gathering. A genius agent that thinks about fishing catches zero fish while it's thinking. Fisher catches fish every tick it's at the river. The fleet's economist (Trader) can worry about *which* fish — Fisher just fills the bucket.

---

## 📚 Skills & Knowledge System

Fisher logs resource data across sessions:

| Knowledge Type | Example |
|---|---|
| River location | `dock → north → bridge → west → river` |
| Catch rates | `River: ~40% catch rate per tick` |
| Fish types | `trout (common), salmon (uncommon), golden koi (rare)` |
| Inventory capacity | `Max 8 items before haul-back` |
| Battery per round-trip | `Dock→River→Dock: ~35% battery` |
| Gear dependency | `Requires fishing rod — check Dock storage` |

This data is gold for the Trader agent, which needs to know supply volume and rarity to price goods.

---

## 🚀 Quick Start

```bash
# Clone the fleet
git clone https://github.com/your-org/zeroclaw-crew.git
cd zeroclaw-crew/fleet-workspace/zeroclaw-fisher

# Boot the fisher (no API keys, no config)
python3 mud_client.py --agent fisher
```

Fisher will grab a rod from the Dock (if available) and make a beeline for the river. Watch the logs for `fish` commands and catch events rolling in.

### Env Vars (optional)
| Variable | Default | Purpose |
|---|---|---|
| `MUD_HOST` | `localhost` | MUD server address |
| `MUD_PORT` | `4000` | MUD server port |
| `FISHER_BATTERY_THRESHOLD` | `20` | Emergency haul-back trigger (%) |
| `FISHER_MAX_INVENTORY` | `8` | Inventory cap before heading home |

---

## 🌐 MUD Integration

Fisher speaks raw MUD protocol — plain text commands over TCP:

| Command | Purpose | Example |
|---|---|---|
| `go <dir>` | Move through exit | `go north` |
| `take <item>` | Pick up item | `take fishing rod` |
| `drop <item>` | Deposit at dock | `drop trout` |
| `fish` | Cast line in water | `fish` |
| `say <msg>` | Chat (rarely used) | `say full haul heading back` |

The server sends back structured state every tick. Fisher's `decide()` checks location, inventory, and battery, then returns exactly **one command** per tick.

### Protocol Flow
```
CLIENT ──→ connect(host, port)
SERVER ──→ {room: "dock", exits: ["north"], items: ["fishing rod"], agents: [], battery: 100}
CLIENT ──→ "take fishing rod"
SERVER ──→ {room: "dock", items: [], inventory: ["fishing rod"], battery: 100}
CLIENT ──→ "go north"
...
SERVER ──→ {room: "river", exits: ["east"], items: [], inventory: ["fishing rod"], battery: 85}
CLIENT ──→ "fish"
SERVER ──→ {result: "caught trout!", inventory: ["fishing rod", "trout"], battery: 84}
CLIENT ──→ "fish"
```

---

## ⚙️ Configuration

| Constant | Default | Effect |
|---|---|---|
| `BATTERY_LOW` | `20` | Emergency return threshold |
| `MAX_CARRY` | `8` | Inventory slots before haul-back |
| `FISH_EVERY_TICK` | `true` | Never skip a cast |
| `AUTO_GRAB_ROD` | `true` | Pick up rod from dock automatically |
| `HAUL_TO_DOCK` | `true` | Return to dock when full (vs. dropping in place) |

---

## 🤝 ZeroClaw Crew

Fisher is one member of the **[zeroclaw-crew](https://github.com/your-org/zeroclaw-crew)** fleet — a family of minimal-intelligence MUD agents that collaborate through shared knowledge files.

| Agent | Role | Specialty |
|---|---|---|
| **Scout** 🔭 | Pathfinder | Room mapping & exploration |
| **Guard** 🛡️ | Security | Patrol routes & threat detection |
| **Fisher** 🎣 | Resources | Fishing & inventory management |
| **Trader** 💰 | Commerce | Item valuation & trading |

Fisher feeds the fleet's economy. Without Fisher, Trader has nothing to sell. Without Guard keeping the river safe, Fisher gets robbed. **One casts, one patrols, one profits.**

---

## License

Part of the [zeroclaw-crew](https://github.com/your-org/zeroclaw-crew) project.

---

<img src="callsign1.jpg" width="128" alt="callsign">
