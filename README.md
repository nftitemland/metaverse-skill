# metaverse-agent skill

An [agent skill](https://skillsmp.com/) for **driving, inspecting, and verifying the
kryptofun 3D metaverse** (the nftmine-minter app) from an automation agent — e.g. Claude
via the **chrome-devtools MCP**, or any Playwright/CDP script.

The metaverse ships a single, versioned, self-describing automation API on
`window.metaverseAgent`. This skill teaches an agent how to use it: control the player
and camera, assert on what's actually rendered (avatars, roads, World-Builder pieces),
read/write in-world UI state, and inject test data for multiplayer presence.

## What it's for

- "Test / verify / screenshot the metaverse"
- "Control the camera in the 3D world"
- "Check what renders in-world"
- Confirm an in-metaverse change actually works (multiplayer presence, the World
  Builder, road pieces, avatars, …)

## How it works

`window.metaverseAgent` is **installed in every environment** — localhost, staging, and
production (`kryptofun.com`). The surface is **client-side only**: it moves the local
camera/player and reads/writes local UI state; multiplayer presence writes stay
HMAC-token-gated on the server. So the same scripts verify a change on localhost *and*
on the live site, and the API can never bypass server-side authority.

```js
const a = window.metaverseAgent;
a.describe();                       // self-describing manifest of every method
a.meta.ready();                     // true once in-world
a.player.teleport(196, 5, 0);       // drive the player
a.camera.lookAt(186, 1, 14);        // frame a world point for a screenshot
a.world.count(o => o.geometry?.type === 'BoxGeometry');  // assert a render
a.presence.inject([{ playerId:'p1', username:'a', char:'alien', pos:[190,1,14], yaw:0 }]);
```

See **[`SKILL.md`](./SKILL.md)** for the full workflow, golden rules, and recipes.

## Install

Copy `SKILL.md` into your agent's skills directory:

```bash
mkdir -p ~/.claude/skills/metaverse-agent
cp SKILL.md ~/.claude/skills/metaverse-agent/SKILL.md
```

It's self-contained — it assumes only the app running (localhost `:8000` or
`kryptofun.com`) and the chrome-devtools MCP.

## License

MIT — see [`LICENSE`](./LICENSE).
