# Changelog

All notable changes to videodock-compose. Dates use the format YYYY-MM-DD.

Version numbers track the sibling app repo [videodock](https://github.com/bear0328/videodock)
in lockstep. Entries here cover deployment / `compose.yaml` / README changes; when an app
release does not touch the deployment preset, the version still bumps to stay aligned but
there is no separate entry.

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
