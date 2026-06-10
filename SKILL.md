---
name: metaverse-agent
description: Drive, inspect, and verify the nftmine-minter (kryptofun) 3D metaverse from an automation agent. Use when testing or screenshotting the metaverse, controlling the player or camera, asserting on what's actually rendered (avatars, roads, world-builder pieces, shapes), driving in-world UI/state, or verifying features like multiplayer presence, the World Builder, or road pieces. Triggers on requests to "test/verify/drive/screenshot the metaverse", "control the camera in the 3D world", "check what renders in-world", or to confirm an in-metaverse change works.
---

# Metaverse Agent

The metaverse exposes a single, self-describing API on `window.metaverseAgent` (and a
back-compat `window.__metaverseTest`) for automation. Drive it through the
**chrome-devtools MCP** `evaluate_script` tool.

It is installed in **every environment** — localhost, staging, and production
(kryptofun.com) — because the surface is **client-side only**: it drives the local
camera/player and reads/writes local UI state; multiplayer presence writes stay
HMAC-token-gated on the server. So the same scripts that verify a change on localhost
also verify it live on kryptofun.com, and the API can never bypass any server authority.

## Golden rules (learned the hard way)

1. **localhost & staging hit the staging backend; kryptofun.com is production.**
   `isStagingMode()` is true on any non-production host (localhost, `*.cloudfront.net`,
   `?staging`) and routes the app's APIs to the staging backend — so you can exercise
   the full stack locally without a deploy. The agent API itself is present on **every**
   host (incl. prod), so the same scripts work against a live change on kryptofun.com.
2. **The camera is `lookAt(tx, ty, tz, dist=12, height=6)`** = frame the world point
   (tx,ty,tz) with the eye placed `dist` back and `height` up, facing it. For an
   explicit eye+target, use `camera.moveTo(x,y,z, tx,ty,tz)`. (It is NOT `look(yaw,pitch)`.)
3. **Production builds don't expose `canvas.__r3f`.** Never reach the THREE scene via
   the canvas — use `world.scene()` / `world.count()` / `world.meshes()` instead.
4. **Always call `describe()` first** to discover the current API (args + docs) — don't
   assume method names from memory.
5. `meta.ready()` must be true (player controller + scene mounted) before driving.

## Setup

Dev server (from `.../dockernftitem/app`):
```bash
npm run start:singleplayer              # serves http://localhost:8000
npm run build:singleplayer:production   # type/bundle check after edits
```
Then with chrome-devtools MCP: `navigate_page` → `http://localhost:8000/`
(add `?buildBypass` to force the 🔨 World Builder chip without a builders login).
To drive **production**, navigate to `https://kryptofun.com/` instead — the same API is there.

Browser gotchas: a `beforeunload` dialog may block navigation/screenshot — `handle_dialog`
→ accept. If "browser already running", `pkill -9 -f "chrome-devtools-mcp/chrome-profile"`
then retry.

## Discover the API

```js
const a = window.metaverseAgent;
a.version;          // "1.0.0"
a.describe();       // { version, usage, namespaces: { ns: { method: {args, doc} } } }
a.meta.ready();     // true once in-world
```

Namespaces: `meta · player · camera · world · state · ui · presence · nav`.

## Common recipes

**Frame + verify a render** (the reliable pattern — no guessing at the camera):
```js
const a = window.metaverseAgent;
a.camera.lookAt(186, 1, 14);                               // frame a world spot
const n = a.world.count(o => o.geometry?.type === 'BoxGeometry'); // assert a count
const items = a.world.meshes(m => m.color === '#ff0000');  // inspect pos/color/visible
// then take_screenshot
```
> `world.meshes/count` predicates run **in the page**, so write them inline.

**Drive the player:**
```js
a.player.pos();                 // [x,y,z] eye position
a.player.teleport(196, 5, 0);
a.player.hectare();             // [hx,hz]
a.nav.toSpawn(); a.nav.toArena();   // toArena exists only in staging (the Test Arena)
```

**Drive UI / world state:**
```js
a.ui.openBuilder();                                  // open World Builder palette
a.state.set('metaverseBuildPaletteOpen', true);      // any registered ReduxX key
a.state.get('metaverseWorldActive');
```
Unregistered state keys throw (ReduxX validates) — read an existing key to learn names,
or check `reactStateManagement.js`.

**Multiplayer presence (test data):**
```js
// Each entry renders the REAL character model for its `char`
// (pepe-gx / pepe / alien / massimo), or a vehicle when char:'car'.
a.presence.inject([
  { playerId:'p1', username:'alice', char:'alien', pos:[190,1,14], yaw:0 },
  { playerId:'p2', username:'bob',   char:'car', carType:'sports', carColor:'#e74c3c', pos:[196,1,14], yaw:1.5 },
]);
a.presence.players();   // current remote players (metaverseRemotePlayers)
a.presence.inject([]);  // clear when done
```
Remote avatars are **real character/car models** (not placeholder capsules), so assert
by the avatar/car meshes — not `CapsuleGeometry`. Each avatar faces its reported `yaw`
(it turns to the look direction) and lerps toward its latest position.

## Verifying a feature end-to-end (example: an in-world render)

1. `navigate_page` → `:8000/?buildBypass` (or `kryptofun.com`); wait for `a.meta.ready()`.
2. Put the world in the state you need (`state.set`, `ui.openBuilder`, place pieces, or
   `presence.inject` test data).
3. Assert structurally with `world.count` / `world.meshes` (counts, colors, positions) —
   more reliable than eyeballing a screenshot.
4. `camera.lookAt(target...)` then `take_screenshot` for a visual confirm.
5. Clean up injected/test state (`presence.inject([])`).

## Extending

The API lives in `metaverseAgent.js` (`SKILL` co-located). Add methods to its `SPEC`
object (`{ args, doc, fn }`) — the runtime API and `describe()` both derive from it, so
new methods are automatically discoverable. Host-specific jumps go through `navExtras`
when `installMetaverseAgent(...)` is called, so the core API stays generic.

## Portability

This `SKILL.md` is self-contained — copy it into any agent's skills directory
(e.g. `~/.claude/skills/metaverse-agent/`). It assumes only: the app running (localhost
`:8000` or `kryptofun.com`) and the chrome-devtools MCP.
