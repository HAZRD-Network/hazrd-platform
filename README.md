<p align="center">
  <img src="docs/banner.webp" alt="HAZRD" width="100%">
</p>

<h1 align="center">HAZRD Platform</h1>
<p align="center">
  <b>A web & ops platform layer on top of dedicated survival game servers.</b><br>
  Player portal · live map · auctions · web inventory · Discord-as-code · monitoring
</p>

<p align="center">
  <a href="https://7days.hazrd.net"><img src="https://img.shields.io/badge/7_Days_to_Die-live-9acd32?style=for-the-badge" alt="live"></a>
  <img src="https://img.shields.io/badge/source-private-0b0d0b?style=for-the-badge" alt="source private">
</p>

> This is a **showcase repository**. The platform runs live game economies, so the source code is private.
> Here you'll find what HAZRD is, how it's built and the engineering decisions behind it.

---

## 🧟 The idea

Survival games like *7 Days to Die* are great — until the endgame: grind, crowded maps, wipes, one server as the whole world.
HAZRD adds a **second layer on top of the game**: a persistent player portal where your identity, items and progress live outside a single session.

- 🗺️ **Live map** — players, claims and zones rendered on the real world map
- 💰 **Economy** — live auctions and a shop that deliver items straight into the game
- 🎒 **Web inventory** — buy or win items on the web, claim them in-game when you're ready
- 🏆 **Seasons, rewards & leaderboards** — progression beyond one wipe
- 💬 **One Discord for the whole network** — generated from config, not clicked together

<p align="center">
  <img src="docs/live-7days.jpg" alt="7days.hazrd.net" width="90%"><br>
  <sub>Live: <a href="https://7days.hazrd.net">7days.hazrd.net</a></sub>
</p>

---

## 🏗️ Architecture

```mermaid
flowchart LR
  subgraph Game["Dedicated game servers"]
    GS1["7DTD Server<br/>(Harmony mods)"]
    GS2["Realm Lab<br/>(test server)"]
  end

  subgraph Platform["HAZRD platform"]
    BR["Game Bridge<br/>Telnet · WebAPI · events"]
    DB[("Wallets · auctions<br/>inventory · claims")]
    DASH["Player portal<br/>Next.js"]
    OBS["Observer<br/>uptime & economy"]
  end

  subgraph Community
    WEB["hazrd.net<br/>network site"]
    BOT["Discord bot<br/>guild-as-code"]
  end

  GS1 <--> BR
  GS2 <--> BR
  BR <--> DB
  DASH <--> DB
  DASH --> BR
  OBS -. polls .-> BR
  OBS -. alerts .-> BOT
  WEB --> DASH
```

| Component | What it does | Stack |
| --- | --- | --- |
| **Player portal** | Map, auctions, shop, web inventory, profiles, leaderboards, wiki | Next.js · TypeScript · Leaflet |
| **Game bridge** | Talks to the game server (Telnet/WebAPI), delivers items, streams game events | Node.js |
| **Server mods** | Server-side game hooks for chat, delivery and events — no client mod required | C# · Harmony |
| **Discord bot** | Generates roles, channels and game spaces from one YAML registry with plan → apply | Node.js · Discord.js |
| **Observer** | Uptime + economy history, Discord alerts, small control panel | Node.js · SQLite |
| **Network site** | Masterbrand site for all games | Next.js · Cloudflare |

---

## 🔧 Engineering highlights

A few problems that were more interesting than they looked:

**Live map without melting bandwidth.**
Map markers are live, but the terrain tiles only change when players build. A naive "redraw every N seconds" looked like a live map in code and did *nothing* — Leaflet rebuilds the `<img>` and the browser serves identical URLs from memory cache, so no request ever leaves.
Solution: ETag/304 on tiles plus a tiny version endpoint the client polls once a minute. Only when it changes does the tile URL get a new cache-buster — instead of ~400 KB per tick per tab.

**Discord as infrastructure.**
The whole guild (roles, categories, channels, permissions, onboarding) is declared in config. Adding a game is a registry entry → validate → plan → apply on staging → apply on production. Destructive changes need an explicit flag.

**Game updates are a deployment, not a click.**
All server mods patch game methods at runtime. A game update can change a signature and crash the server on start instead of failing cleanly — so updates go through the test realm first and are never auto-applied to the live server.

**Bridge topology.**
Today: one bridge sidecar per game server (simple, isolated). Next: one bridge for the whole cluster as the single source of truth for wallets, auctions and zones — which also unlocks cluster-wide chat.

---

## 🎨 Design concepts

<p align="center">
  <img src="docs/concept-map.png" alt="Map & travel concept" width="49%">
  <img src="docs/concept-auctions.png" alt="Auctions concept" width="49%">
</p>
<p align="center"><sub>Concept designs for the player portal — map & travel, auctions. Not all shown features are live yet.</sub></p>

---

## 🧭 Roadmap

- [x] 7 Days to Die player portal, live map & markers
- [x] Discord network with guild-as-code
- [x] Monitoring & alerting (Observer)
- [ ] Shop → in-game item delivery
- [ ] Warps, homes & map overlays
- [ ] Cluster-wide bridge & chat
- [ ] Palworld as the second game

---

<p align="center">
  <sub>Built by <a href="https://github.com/mixu-94">@mixu-94</a> · part of the <a href="https://github.com/HAZRD-Network">HAZRD Network</a> · <a href="https://hazrd.net">hazrd.net</a></sub>
</p>
