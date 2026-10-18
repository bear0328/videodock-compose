# Changelog

All notable changes to videodock-compose. Dates use the format YYYY-MM-DD.

Version numbers track the sibling app repo [videodock](https://github.com/bear0328/videodock)
in lockstep. Entries here cover deployment / `compose.yaml` / README changes; when an app
release does not touch the deployment preset, the version still bumps to stay aligned but
there is no separate entry.

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
