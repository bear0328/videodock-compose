**English** | [简体中文](README_CN.md)

Deployment presets for videodock — a multi-site video
downloader + web playback platform in a single Docker container (built for Unraid).

**Supported sites:** 91porn · Pornhub · Bilibili · Douyin(抖音) · YouTube · TikTok · X/Twitter and 500+ more via yt-dlp — a self-hosted media server & video downloader for Unraid.

- **91-series** (91porn / bilithings, …) via a dedicated adapter — poster + metadata included
- **YouTube, Bilibili, TikTok, X/Twitter and 1000+ other sites** via the built-in yt-dlp engine
- **No database, no workers**: one container with web UI, task queue, and playback
- **Libraries auto-classified** into an *adult* and a *general* mount; in-app proxy & cookie
  management with live verification; rename / search / delete from the UI

## Quick start (Docker Compose)

```bash
docker compose pull -f compose.yaml
docker compose up -d -f compose.yaml
# open http://<server-ip>:8620
```

On Unraid, place `compose.yaml` in `/boot/config/videodock/` and manage it with the Docker
Compose plugin:

```bash
mkdir -p /boot/config/videodock /mnt/user/appdata/videodock/data
cp compose.yaml /boot/config/videodock/
```

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
  - /mnt/user/downloads/youtube:/general   # general-site library (yt-dlp)
  - /mnt/user/appdata/videodock/data:/appdata
environment:
  - VIDEODB_DATA=/
```

Both volumes must exist and be writable. To keep everything in one tree, map both mounts to
the same host path (they will create `adult/` and `general/` subdirectories under it).

## compose.yaml reference

**Volumes**

| Mount | Purpose |
| --- | --- |
| `/adult` | Adult library (91-series) |
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

## Cookies & login-gated content

- **91-series**: public videos work without cookies; private videos need a login. Paste
  the browser's raw `Cookie` header into *Settings → 91 series cookies*.
- **YouTube**: public videos work anonymously; age-gated / members-only need a logged-in
  cookie.
- **Bilibili**: member-only / high-quality streams need `SESSDATA` (copy the full cookie).
- **Douyin**: public videos download with no cookies; the web API sits behind a bot
  challenge, so the adapter uses the mobile share page instead. Only paste cookies if a
  private or age-gated video ever fails.

The in-app **Verify** button checks each site from the container and reports
*Valid / Invalid*, so you can confirm a paste before downloading.

### How to get a cookie

videodock wants the **raw browser `Cookie` header** — exactly the string the browser
sends (`name=value; name2=value2; …`). Extract it with DevTools:

1. Log in to the site in your browser.
2. Open DevTools (F12) → **Network**, reload the page, then click any request to the site.
3. In the request details, open **Request Headers** and copy the entire value of the
   `Cookie:` line.
4. Paste it into the matching field under *Settings*, then press **Verify**.

For Bilibili you can instead grab just the three essentials from *Application → Cookies*
and join them:

```text
SESSDATA=…; bili_jct=…; dediuserid=…
```

`SESSDATA` is the key token — without it you only get guest-level access.

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
docker compose pull -f compose.yaml
docker compose up -d -f compose.yaml
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

