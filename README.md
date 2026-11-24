# rockstar-lua

Steam depot pin files (`.lua`) for the Rockstar Games library, consumed by
[femstar](https://github.com/one-eyed-queen) / `femdl` so the launcher can install any
Rockstar title without a signed-in Steam account that owns it (the way
[SteamAutoCracks/DepotDownloaderMod](https://github.com/SteamAutoCracks/DepotDownloaderMod)
does).

## Layout

```
<steam-appid>/
  <steam-appid>.lua       -- primary pin file for the app
  <steam-appid>.anon.lua  -- optional variants, same shape
  ...
```

**One directory per Steam app id. No build sub-folders here** — a single `.lua`
holds every build this project cares about. The matching `.manifest` files live
in [`rockstar-manifest`](https://github.com/one-eyed-queen/rockstar-manifest) under
`<appid>/<buildid>/`.

## File format

Each `.lua` is the usual Steam pin file (SteamAutoCracks shape). The lines
`femdl` reads:

```lua
-- Grand Theft Auto V Legacy (app 271590)
-- Build 10188340 — 2024-06-18 — v1.0.3411.0
addappid(271590)
addappid(271591, 1, "AAAABBBBCCCCDDDDEEEEFFFF0011223344556677889900aabbccddeeff00112233")
setManifestid(271591, "6287320779057191210", 0)

-- Build 10090000 — 2024-04-02 — v1.0.3407.0
-- setManifestid(271591, "5011223344556677889", 0)
```

- `addappid(<depot>, 1, "<hex key>")` — depot id plus its 256-bit decryption
  key. One line per depot. `femdl` uses the key straight from here, so the
  Steam logon can be anonymous.
- `setManifestid(<depot>, "<gid>", 0)` — the active manifest for that depot.
  Lines commented out with `--` are older builds `femdl` can still pick in
  **advanced install** mode.
- Build-identifying comments on the line above any `setManifestid` give the
  launcher the **build id**, **version** and **release date** it shows in the
  install popup. The parser looks for `Build <id>`, a date like
  `YYYY-MM-DD`, and a version like `v<x.y.z>`.

## How the launcher uses it

- **Download only latest (default):** picks the first uncommented
  `setManifestid` per depot.
- **Advanced:** the launcher lists every `setManifestid` it finds (commented
  or not), tagged with the build id, version and date from the preceding
  comment, and the user picks which one.

Every line is cached locally after the first fetch so the launcher works
offline once a game has been installed.
