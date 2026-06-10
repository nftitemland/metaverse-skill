# metaverse-agent skill — DEPRECATED

> **⚠️ Deprecated / not indexed.** This is an internal, **staging-only** dev/test harness
> for driving the kryptofun metaverse during development — kept here for reference only.
> It is intentionally **not** published or maintained as a public agent skill: the skill
> file is named [`deprecated_SKILL.md`](./deprecated_SKILL.md) (not `SKILL.md`), so skill
> aggregators such as skillsmp.com don't index it. The production AI surface is a
> separate, higher-level "standard usage" interface, not this low-level dev harness.

A reference doc for **driving, inspecting, and verifying the kryptofun 3D metaverse**
(the nftmine-minter app) from an automation agent — e.g. via the **chrome-devtools MCP**,
or any Playwright/CDP script — using the self-describing `window.metaverseAgent` API.

The agent API is installed **only in staging mode** (`isStagingMode()` — localhost,
`*.cloudfront.net`, `?staging`), never in production. The surface is client-side only
(it drives the local camera/player and reads/writes local UI state; multiplayer presence
writes stay HMAC-token-gated on the server), so it can't bypass any server authority.

```js
const a = window.metaverseAgent;   // present on localhost/staging only
a.describe();                       // self-describing manifest of every method
a.meta.ready();                     // true once in-world
a.player.teleport(196, 5, 0);       // drive the player
a.camera.lookAt(186, 1, 14);        // frame a world point for a screenshot
a.world.count(o => o.geometry?.type === 'BoxGeometry');  // assert a render
a.presence.inject([{ playerId:'p1', username:'a', char:'alien', pos:[190,1,14], yaw:0 }]);
```

See **[`deprecated_SKILL.md`](./deprecated_SKILL.md)** for the full workflow, golden
rules, and recipes.

## License

MIT — see [`LICENSE`](./LICENSE).
