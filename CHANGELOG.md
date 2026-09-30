# Changelog

## v2.2.5 — 2026-10-01

### Added

- **Resolution badge on thumbnails**: video resolution (4K/2.5K/1080p) shown in bottom-right corner of poster thumbnails. Backend extracts from `video.info.json` or falls back to ffprobe for older videos.

## v2.1.0 — 2026-07-20

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

## v2.0.0 — 2026-07-19

### Features

- **Multi-site engine**: yt-dlp based adapter for YouTube, Bilibili, Vimeo and 1000+ more sites (auto-classified by domain, `general/` subdirectory)
- In-container ffmpeg (no external volume mount needed)
- Per-engine proxy settings (`PAGE_PROXY` / `CDN_PROXY` / `YTDP_PROXY`) and cookie support via `YTDP_COOKIES`

### Notes

- Existing 91 videos are auto-migrated into `adult/` on first start; the web UI stays fully functional while an optional migration runs in the background

## v1.0.0 — 2026-07-18

### Features

- One-container videodock (web UI + downloader), 91porn downloader with CDN stream acceleration
- Unraid compose profile: media library, appdata volume, proxy env vars, tarball deployment
