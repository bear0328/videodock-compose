**English** | [简体中文](README_CN.md)

Deployment presets for videodock — a multi-site video
downloader + web playback platform in a single Docker container (built for Unraid).

**Supported sites:** 91porn · Pornhub · Bilibili · Douyin(抖音) · Kuaishou(快手) · Xiaohongshu(小红书) · YouTube · TikTok · X/Twitter and 500+ more via yt-dlp — a self-hosted media server & video downloader for Unraid.

- **91-series** (91porn / bilithings, …) via a dedicated adapter — poster + metadata included
- **YouTube, Bilibili, TikTok, X/Twitter and 1000+ other sites** via the built-in yt-dlp engine
- **No database, no workers**: one container with web UI, task queue, and playback
- **Libraries auto-classified** into an *adult* and a *general* mount; in-app proxy & cookie
  management with live verification; rename / search / delete from the UI

## Quick start (Docker Compose)

```bash
docker compose -f compose.yaml pull
docker compose -f compose.yaml up -d
# open http://<server-ip>:8620
```

On Unraid, place `compose.yaml` in `/boot/config/docker/compose/videodock/` and manage it with the Docker
Compose plugin:

```bash
mkdir -p /boot/config/docker/compose/videodock /mnt/user/appdata/videodock/data
cp compose.yaml /boot/config/docker/compose/videodock/
```

Keep the file on the boot flash and run compose there. Do **not** run `docker compose`
from inside `/mnt/user` — that wedged the share's FUSE mount on 2026-11-01 (every path
under `/mnt/user` stopped responding until the server was restarted).

Adjust the two library paths in `compose.yaml` to your own directories (see below), then
pull & start as shown.

### First run on Unraid (appdata permissions)

The container runs as user `1000` (group `1001`). The **appdata** volume must be writable
by that user, or tasks fail with *permission denied*:

```bash
chmod 775 /mnt/user/appdata/videodock   # or: chown -R 1000:1001 /mnt/user/appdata/videodock
```

## Media libraries (two mounts)

videodock classifies every download into one of two libraries: **91-series** links land in
the *adult* mount, everything else (yt-dlp sites) lands in the *general* mount. With
`VIDEODB_DATA=/`, those are exactly `/adult` and `/general`:

```yaml
volumes:
  - /mnt/user/media/ori/91:/adult          # 91-series library
  - /mnt/cache/strm/happy/91:/strm/happy/91:ro  # same 91 set, read-only strm
  - /mnt/user/downloads/youtube:/general   # general-site library (yt-dlp)
  - /mnt/user/appdata/videodock/data:/appdata
environment:
  - VIDEODB_DATA=/
```

Both library volumes must exist and be writable. To keep everything in one tree, map both mounts to
the same host path (they will create `adult/` and `general/` subdirectories under it).

`/strm/happy/91` is not a third library. It is the read-only strm directory for the adult
library, mounted from `/mnt/cache/strm/happy/91`. `/mnt/user/strm` is a symlink to
`/mnt/cache/strm`, so the mount uses the real directory. The container path is
`VD_EMBY_STRM_PREFIX`. A local adult item matches only when a `.strm` file of the same
name is in that directory.

## compose.yaml reference

**Volumes**

| Mount | Purpose |
| --- | --- |
| `/adult` | Adult library (91-series) |
| `/strm/happy/91` | Read-only strm directory for that adult library |
| `/general` | General library (all yt-dlp sites) |
| `/appdata` | Metadata, task queue, settings, logs — survives image updates |

**Environment**

| Variable | Default | Description |
| --- | --- | --- |
| `PORT` | `8620` | Web UI port |
| `VIDEODB_DATA` | `/data` | Library root; videos land in `<root>/adult` or `<root>/general` (preset uses `/`) |
| `APPDATA` | `/appdata` | App-data directory |
| `PAGE_PROXY` | *(empty)* | Proxy for fetching 91-series pages & posters. **Required if your server cannot reach the site directly.** All proxy vars can also be set in the UI (Settings page, no restart). |
| `CDN_PROXY` | follows `PAGE_PROXY` | Proxy for 91 CDN video streams — often much faster than direct. Set to `""` to force direct |
| `YTDP_PROXY` | follows `PAGE_PROXY` | Proxy for the yt-dlp engine (YouTube, Bilibili, …) |
| `YTDP_COOKIES` | *(empty)* | Netscape `cookies.txt` path — optional alternative to the in-app cookie fields |
| `VD_EMBY_URL` | *(empty)* | Emby base URL. Empty disables Emby playback and keeps local files |
| `VD_EMBY_STREAM_URL` | *(empty, uses `VD_EMBY_URL`)* | Proxy that 302s `/Videos/{id}/stream` after PlaybackInfo. The media source id is not the item id. No redirect keeps VOD HLS |
| `VD_EMBY_KEY` | *(empty)* | Emby API key. Put it in the compose directory `.env`, not in `compose.yaml` |
| `VD_EMBY_USER_ID` | *(empty)* | Emby user id. Empty selects an administrator |
| `VD_EMBY_STRM_PREFIX` | `/strm/happy/91/` | Container path of the read-only strm mount. It must match the Emby item path. Only `.strm` names in this directory match the adult library |

## Cookies & login-gated content

- **91-series**: public videos work without cookies. There is no 91 cookie setting; page
  access uses the configured page proxy and the `cn_CN` locale.
- **YouTube**: public videos work anonymously; age-gated / members-only need a logged-in cookie.
- **Bilibili**: member-only / high-quality streams need `SESSDATA` in the exported cookies.
- **Douyin**: a valid login cookie is recommended for public and private videos because it
  restores the mobile share page's server-rendered data. The default image has no Playwright fallback.
- **Xiaohongshu**: public notes need no cookies; the setting is only for private notes.

The in-app **Verify** button checks each site from the container and reports
*Valid / Invalid*, so you can confirm a paste before downloading.

### How to get cookies

videodock accepts **Netscape `cookies.txt` exports or Cookie-Editor JSON**, not a raw
`name=value; …` Cookie request header.

1. Log in to the site in your browser.
2. Export its cookies with "Get cookies.txt LOCALLY" or Cookie-Editor.
3. Paste the exported contents into the matching field under *Settings*, then press **Verify**.

For Bilibili, `SESSDATA` is the key token — without it you only get guest-level access.

## Typical use

1. Open `http://<ip>:8620`, paste a link (91porn, YouTube, Bilibili, …) and press Enter
2. Downloads queue in the background with live progress & speed; finished videos appear
   in the library automatically (already classified into the right mount)
3. Click a card to play — direct HTTP Range streaming, seekable, no transcoding
4. Library: adult / general tabs, title search, rename (media + metadata moved together),
   delete
5. Proxy, cookies and the 91 channel (CN/EN/AI/…) are managed on the Settings page —
   changes apply instantly, no restart

## Updating

```bash
docker compose -f compose.yaml pull
docker compose -f compose.yaml up -d
```

The library is scanned from disk, so all existing videos stay in place; task state and
settings persist in the appdata volume.

### Tarball deployment (alternative)

For hosts without docker compose: `docker load -i videodock.tar`, then run the container
with the same port / volume / environment settings as above.

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| *permission denied* on the data directory at startup | `chmod 775 /mnt/user/appdata/videodock` (user 1000) |
| 91-series pages slow / failing | Set `PAGE_PROXY` (or the page proxy in Settings) to a working proxy |
| 91 downloads crawl at a few KB/s | Leave `CDN_PROXY` following `PAGE_PROXY`; check the proxy route quality |
| YouTube/Bilibili 403 or login redirects | Set the download proxy, and paste cookies if the content is gated (then **Verify**) |
| Proxy `407 / Bad Gateway` | Your proxy needs a different host/port or auth — update Settings; `verify` confirms reachability |
| Video plays but posters are slow | Posters stream through the app using the page proxy — adjust it in Settings |

---

