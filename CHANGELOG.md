# Changelog

All notable changes to videodock-compose. Dates use the format YYYY-MM-DD.

Version numbers track the sibling app repo in lockstep. Entries here cover deployment / `compose.yaml` / README changes; when an app
release does not touch the deployment preset, the version still bumps to stay aligned but
there is no separate entry.

## [2.14.8] — 2026-11-08

### Changed

- **Synced to app v2.14.8** — Emby / 115 direct playback now recovers from short-lived cloud 403s (fresh signed link, one retry, HLS fallback) and the player offers a visible retry. `compose.yaml` needs no change.

## [2.14.7] — 2026-11-08

### Changed

- **Synced to app v2.14.7** — Xiaohongshu cookie validation now checks the site's login state instead of always failing, and the first paste no longer errors when the cookie directory is absent. `compose.yaml` needs no change.

## [2.14.6] — 2026-11-08

### Changed

- **Synced to app v2.14.6** — the player now waits for a click before playing, keyboard shortcuts work regardless of focus, and failed playback shows a visible retry. `compose.yaml` needs no change.

## [2.14.5] — 2026-11-08

### Changed

- **Synced to app v2.14.5** — GitHub quality CI now uses the Chrome path supplied by `setup-chrome`, so the UI self-test runs on Linux runners. `compose.yaml` needs no change.

## [2.14.4] — 2026-11-08

### Changed

- **Synced to app v2.14.4** — the download queue removes a task and its unfinished files, cloud-backed Emby playback is proxied with Range support on mobile, and the quality CI self-test runs without a local virtualenv. `compose.yaml` needs no change.

## [2.14.3] — 2026-11-07

### Changed

- **Synced to app v2.14.3** — failed and cancelled downloads keep their partial files and can continue from the library history. `compose.yaml` needs no change.

## [2.14.2] — 2026-11-07

### Changed

- **Synced to app v2.14.2** — touch devices without hover open an adult poster full screen before playback. Desktop hover-to-reveal and click-to-play are unchanged. `compose.yaml` needs no change.

## [2.14.1] — 2026-11-07

### Changed

- **Synced to app v2.14.1** — the adult strm directory is mounted read-only next to the adult library. Matching uses the `.strm` names in that directory. Optional `VD_EMBY_STREAM_URL` points at a proxy that redirects `/Videos/{id}/stream` to the cloud file. Leave it empty to keep VOD HLS through `VD_EMBY_URL`.

## [2.14.0] — 2026-11-07

### Changed

- **Synced to app v2.14.0** — the adult library can match Emby strm items and play them as
  VOD HLS through videodock. Optional `VD_EMBY_URL`, `VD_EMBY_KEY`, `VD_EMBY_USER_ID` and
  `VD_EMBY_STRM_PREFIX` are read from the compose directory `.env`. Leave them empty to keep
  the previous local-file playback. The key is not written into `compose.yaml`.

## [2.13.4] — 2026-11-07

### Changed

- **Synced to app v2.13.4** — the app release hardens deletion, settings and subscription
  failure handling, makes metadata and directory creation safer, pins runtime dependencies
  and adds repository quality checks. `compose.yaml` needs no change.

## [2.13.3] — 2026-11-01

### Changed

- **Synced to app v2.13.3** — the app release reduces task-polling payloads, batches playback
  progress writes, avoids duplicate library I/O and bounds batch parsing and progress-update
  overhead. `compose.yaml` needs no change.

## [2.13.2] — 2026-11-01

### Changed

- **Synced to app v2.13.2** — the app release fixes category preloading and stale-response
  handling, improves grid and queue rendering, and polishes theme and keyboard accessibility.
  `compose.yaml` needs no change.

## [2.13.1] — 2026-11-01

### Changed

- **Synced to app v2.13.1** — the app release contains review fixes for media path validation,
  download defaults, cancellation and secret-safe logging, and migrates project skills to Kimi Code.
  `compose.yaml` needs no change.
- Corrected the bilingual cookie guide: the app accepts Netscape `cookies.txt` or Cookie-Editor
  JSON, has no 91-series cookie setting, and recommends a login cookie for Douyin.

## [2.13.0] — 2026-11-01

### Changed

- **Synced to app v2.13.0** — the app release adds Kuaishou (快手) and Xiaohongshu
  (小红书) adapters, including image notes that now show up in the library with a
  cover and can be viewed and deleted from the UI.
- **Compose lives in Unraid's standard location** — `compose.yaml` goes to
  `/boot/config/docker/compose/videodock/` (boot flash) and compose runs there, not
  inside `/mnt/user`: running compose in the share wedged its FUSE mount on
  2026-11-01. README quick-start commands and the compose file's comments were
  updated to match, and the supported-sites line now lists 快手 / 小红书.

## [2.12.0] — 2026-11-01

### Changed

- **Synced to app v2.12.0** — the app release adds download option groups with named
  presets, subtitles / chapters / SponsorBlock marks, batch operations with tags, and a
  library disk-usage readout with an optional quota warning. `compose.yaml` needs no
  change — everything new lives in the existing `/appdata` mount and the media library.

## [2.11.0] — 2026-11-01

### Changed

- **Synced to app v2.11.0** — the app release adds subscriptions (auto-follow): watch a YouTube
  channel or a Bilibili UP space and its new uploads are queued automatically on a per-source
  interval. `compose.yaml` needs no change — the subscription state lives in the existing
  `/appdata` mount.

## [2.10.0] — 2026-11-01

### Changed

- **Synced to app v2.10.0** — the app release adds playback progress (“continue watching”),
  a download preview, automatic retries for failed downloads and a share / quick-submit entry.
  `compose.yaml` needs no change — the preset was already correct: the new playback state
  lives in the existing `/appdata` mount, and `VD_MAX_RETRIES` is an optional override
  (default `3`).

## [2.9.5] — 2026-11-01

### Changed

- **Synced to app v2.9.5** — media URLs (video/poster) are no longer public: they now need the
  session cookie or a short-lived signed token (`?k=…`), the way Emby passes `api_key` to
  `/Videos/{id}/stream`. Anyone with a copied media link can no longer pull a video without the
  access password. `compose.yaml` needs no change — the preset was already correct.

## [2.9.4] — 2026-10-31

### Changed

- **Synced to app v2.9.4** — the app release closes an unauthenticated-deletion hole, fixes
  rename/move/delete on deployments that mount the libraries at `/adult` + `/general`, and
  makes the deploy-test E2E actually assert (it had been reporting green while every probe
  got a 302/401). `compose.yaml` needs no change — the preset was already correct.
- **Removed a stale `VD_API_PASSWORD` line from the production compose** — the app stopped
  reading a password from the environment in v2.9.0 (the password lives in the app's own
  `settings.json`), so the variable was dead weight in the deployed file. It was never part
  of this repository's `compose.yaml`.

## [2.9.3] — 2026-10-26

### Changed

- **Timezone config moved into `compose.yaml`** — added `TZ=Asia/Shanghai` to the
  `environment` block. The timezone is now set per-deployment instead of being baked into the
  image, so changing it no longer requires a rebuild. `compose.yaml` itself is the only
  deployment-preset change; the app image keeps only `tzdata`.

## [2.9.2] — 2026-10-26

### Changed

- **Synced to app v2.9.2** — container logs (`docker logs`) now carry Shanghai local
  timestamps (uvicorn `--log-config` + `TZ=Asia/Shanghai`); `PyYAML` added to the image.
  `compose.yaml` needs no change — both are baked into the image.

## [2.9.1] — 2026-10-25

### Changed

- **Synced to app v2.9.1** — container now runs in Asia/Shanghai timezone and logs
  carry Shanghai local time. The timezone is baked into the image (`TZ=Asia/Shanghai`),
  so `compose.yaml` needs no change.


### Changed

- **Synced to app v2.9.0 — access password now local.** The app no longer reads
  `VD_API_PASSWORD` from the compose environment; the password is set in the UI
  (Settings → Access password) and persisted locally, defaulting to `123456`.
  Removed the `VD_API_PASSWORD=${VD_API_PASSWORD}` line from `compose.yaml`, so the
  password can no longer leak through the deployment env. The UI still requires a
  password on every route except `/login` (a 30-day httpOnly session cookie).

## [2.8.4] — 2026-10-25

### Docs

- **Synced to app v2.8.4:** Desktop fullscreen now fills the video itself instead
  of the whole player overlay (which previously looked like the entire page went
  fullscreen). Deployment preset unchanged.

## [2.8.3] — 2026-10-24

### Docs

- **Synced to app v2.8.3:** Desktop player gained a fullscreen button and the `f`
  keyboard shortcut; iOS native playback now launches full-screen. Deployment preset
  unchanged.

## [2.8.2] — 2026-10-22

### Docs

- **Synced to app v2.8.2:** PC "All" view was empty and duplicated the category
  buttons — the sidebar now remains the single category-navigation entry point on
  desktop (content-area Adult/General buttons hidden at viewport ≥ 641px), and the
  previously-empty "All" view now merges all sites. Deployment preset unchanged.

## [2.8.1] — 2026-10-22

### Docs

- **Synced to app v2.8.1**: "Completed" count now reflects only successfully
  downloaded tasks — the sidebar "已完成" counter previously counted every entry
  in the download history (including errored/cancelled tasks and duplicate
  re-downloads); it now counts only tasks that reached the `done` state.
  `compose.yaml` deployment preset unchanged (app-only change).

## [2.8.0] — 2026-10-22

### Docs

- **Synced to app v2.8.0**: 91porn embed / short links now supported — links of the
  form `91.9p9.xyz/ev.php?VID=<token>` (and bare encrypted embed tokens) resolve to
  the same video as their `91porn.com/view_video.php?viewkey=...` counterpart; the
  adapter reads the numeric VID from the poster and reuses the main-site download
  path, deduping both forms to one library entry. `compose.yaml` deployment preset
  unchanged (app-only change).

## [2.7.6] — 2026-10-19

### Docs

- **Synced to app v2.7.6**: downloaded filenames no longer keep URL percent-encoding —
titles that yt-dlp / adapters still returned as UTF-8 URL-encoded strings (e.g.
`%E8%B1%86...`) were saved verbatim; now they URL-decode to the real title before the
on-disk name is built. `compose.yaml` deployment preset unchanged (app-only change).

## [2.7.5] — 2026-10-18

### Docs

- **Synced to app v2.7.5**: PWA install button restored (added missing `/sw.js` and
  `/icons/{name}` routes) and home-screen icon fixed — maskable icon full-bleed white so
  the Mac Dock icon has no black border and iOS home screen shows no white-sticker
  overlay. `compose.yaml` deployment preset unchanged (frontend-only app change).

## [2.7.4] — 2026-10-18

### Docs

- **Synced to app v2.7.4**: mobile "Download to Local" now saves to the Files app via file
  share; a new download-feedback toast; PWA service worker made network-first so updates push
  to phones again; theme light/dark toggle moved from the top bar into Settings (single-row
  mobile layout). `compose.yaml` deployment preset unchanged (frontend-only app change).

## [2.7.3] — 2026-10-18

### Docs

- **Synced to app v2.7.3**: mobile player control bar fixes (bottom controls no longer pushed
  off-screen on iOS Safari via `flex: 1 1 0` + `100dvh`) and removal of the redundant stop /
  fullscreen buttons from the mobile player. `compose.yaml` deployment preset unchanged
  (frontend-only app change).

## [2.7.2] — 2026-10-18

### Docs

- **Removed Emby STRM mount**: dropped the `videodock` `/mnt/cache/strm/happy:/strm/happy:ro`
  volume — the app no longer carries the Emby STRM library browse/play feature (reverted to plain
  download + playback). Deployment preset now ships only media + appdata mounts.

## [2.7.1] — 2026-10-09

- **Synced to app 2.7.1** (PWA installable to Mac Dock). No change to `compose.yaml`, env
  vars, volumes, or deployment steps — version bumped to stay aligned with the app.

## [2.2.5] — 2026-10-01

### Added

- **Resolution badge on thumbnails**: video resolution (4K/2.5K/1080p) shown in bottom-right corner of poster thumbnails. Backend extracts from `video.info.json` or falls back to ffprobe for older videos.

## [2.1.0] — 2026-07-20

### Features

- **Settings page** in the web UI: edit proxy settings (page / CDN stream / yt-dlp) live with connection test; paste raw browser cookies for 91-family, Bilibili and YouTube with one-click verification
- **Video library**: title search; tab filters (All / No cover / No info / 91)
- **Rename videos** from the web UI (per video or in bulk, 4-character short titles become full ones)
- In-app proxy changes apply instantly — no container restart

### Fixes

- Cross-device video move (re-download) now keeps the task in "download complete" state and does not re-run post-processing
- Cookie verification correctly reports 91-family cookie-less mode vs "not logged in"
- Fixed a UI glitch where the library grid stopped refreshing after downloads finished

### Docs

- Full EN/CN README with cookie retrieval guide (91 / Bilibili / YouTube)

## [2.0.0] — 2026-07-19

### Features

- **Multi-site engine**: yt-dlp based adapter for YouTube, Bilibili, Vimeo and 1000+ more sites (auto-classified by domain, `general/` subdirectory)
- In-container ffmpeg (no external volume mount needed)
- Per-engine proxy settings (`PAGE_PROXY` / `CDN_PROXY` / `YTDP_COOKIES`) and cookie support via `YTDP_COOKIES`

### Notes

- Existing 91 videos are auto-migrated into `adult/` on first start; the web UI stays fully functional while an optional migration runs in the background

## [1.0.0] — 2026-07-18

### Features

- One-container videodock (web UI + downloader), 91porn downloader with CDN stream acceleration
- Unraid compose profile: media library, appdata volume, proxy env vars, tarball deployment
