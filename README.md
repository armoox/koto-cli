<div align="center">
  <img src="assets/logo.png" alt="koto-cli" height="128">
  <p><strong>Watch anime from your terminal.</strong></p>

  <p>
    <a href="https://github.com/VVAT3R/koto-cli">GitHub</a> •
    <a href="https://github.com/VVAT3R/koto-cli/issues">Issues</a>
  </p>

  <p>
    <img src="https://img.shields.io/badge/license-GPLv3-blue" alt="License">
    <img src="https://img.shields.io/badge/shell-POSIX-green" alt="POSIX">
  </p>
</div>

---

A standalone POSIX shell script to browse, search and watch anime from the command-line, inspired by [pystardust/ani-cli](https://github.com/pystardust/ani-cli). Built by reverse-engineering [anikototv.to](https://anikototv.to) — decoding how it serves HLS streams from its obfuscated embeds — to extract playable `.m3u8` links without any crypto dependencies or API keys.

### Quick start

**Install:**
```sh
curl -fsSL https://raw.githubusercontent.com/armoox/koto-cli/main/install.sh | sudo sh
```

**Uninstall:**
```sh
curl -fsSL https://raw.githubusercontent.com/armoox/koto-cli/main/uninstall.sh | sudo sh
```

<details>
<summary>Features</summary>

- **anikoto** source — direct `.m3u8` streams, no crypto dependencies
- Hard-sub (hsub), sub, soft-server and dubbed playback
- Watch history with resume position tracking (`-c`, `--resume`)
- Batch download with progress indicator (`--batch`)
- Single & multi-episode selection, including ranges (`-e`, `-r`)
- Quality selection (`-q`)
- Interactive menus (fzf / rofi / dmenu)
- Jump between seasons of a show from the post-play menu
- Skip intros with ani-skip (`--skip`, mpv only)
- Player support: mpv, vlc, iina, android, flatpak, syncplay and more
- Next-episode countdown (`-N`)
- Install & uninstall scripts included

</details>

<details>
<summary>Notes on episode availability</summary>

- **Episode not yet uploaded**: If the latest episode of an airing anime has been released but koto-cli shows it as unavailable, it means anikototv.to has not yet added it to their database. Try again later.
- **No valid sources**: If the episode exists on anikoto but koto-cli fails to play it, none of the supported servers (hsub, sub, soft-server, dub) are available for that episode.

</details>

<details>
<summary>How koto-cli came to be</summary>

koto-cli started as a fork of ani-cli whose provider missed some anime I wanted to watch. I reverse-engineered [anikototv.to](https://anikototv.to) and built an **anikoto** provider, then spun it out into this standalone repo. Everything runs in plain POSIX shell — no API keys, no crypto, no external dependencies.

The pipeline: search the site's AJAX endpoints → extract anime ID and episode list → resolve hsub/sub/soft-server/dub servers → grab `.m3u8` streams from megaplay embeds → auto-detect and strip obfuscated segment prefixes → feed clean MPEG-TS to the player.

</details>

<details>
<summary>How anikoto scraping works</summary>

koto-cli scrapes **anikototv.to** by following these steps:

1. **Search** — Queries the AJAX search endpoint and extracts anime slugs and titles from the HTML response.
2. **Anime ID** — Fetches the watch page for a slug and extracts the numeric ID from the `data-id` attribute.
3. **Episode list** — Fetches the episode list via AJAX. Each episode entry has a `data-num` (episode number) and `data-ids` (server group ID).
4. **Server list** — Uses the server group ID to fetch available servers. Each server has a **type** (`hsub`, `sub`, `soft-server`, or `dub`) and a **link ID**.
5. **Server type priority** — Servers are tried in order depending on the mode:
   - **Sub mode** (default): `hsub` → `sub` → `soft-server`
   - **Dub mode** (`--dub`): `dub` only

   Not all server types are available for every anime — it depends on what anikototv.to has uploaded. If `hsub` isn't available for a particular anime, koto-cli falls back to `sub`, then `soft-server`. This is normal site behavior, not a bug.
6. **Embed resolution** — Each link ID is resolved to an embed URL (e.g. megaplay, vidtube, akirax). The embed page is parsed for a player element with a `data-id` attribute.
7. **Stream extraction** — The file ID is sent to the embed host's `getSources` API along with the server type. The API returns an `.m3u8` HLS stream URL.
8. **Playback** — Direct hosts (vidtube, akirax) stream straight to mpv. Obfuscated hosts (megaplay etc.) go through a local feed that strips the junk prefix from each segment and writes clean MPEG-TS files for mpv to read.

</details>

<details>
<summary>Usage</summary>

```sh
koto-cli [options] [query]
```

Run `koto-cli -h` for the full list of options.

</details>
