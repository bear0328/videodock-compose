# videodock-compose

One-container deployment for [videodock](https://github.com/bear0328/videodock) — a general-purpose video downloader + web playback platform.

## Features

- **91porn** and the rest of the 91 family (91porn / 91pops / my91abc / tuchek / 69tuchk) via a built-in adapter — no cookies needed for most videos
- **YouTube, Bilibili, Vimeo and 1000+ other sites** via the built-in yt-dlp engine
- **Single container** with web UI + downloader; no external database, no separate workers
- Download queue with progress, instant playback in the browser (MP4), built-in media library UI (grid / title search / tab filters)
- In-app proxy and cookie management, video renaming

## Quick start (Docker Compose)

```bash
docker compose pull -f compose.yaml
docker compose up -d -f compose.yaml
# open http://<server-ip>:8620
```

Or place `compose.yaml` in an Unraid app directory and manage it with the Docker Compose plugin.

### First run on Unraid (appdata permissions)

The container runs as user `1000` (group `1001`). The **appdata** volume needs write access for that user — otherwise tasks will fail with a *permission denied* error on the data directory:

```bash
chmod 775 /mnt/user/appdata/videodock   # or: chown -R 1000:1001 <appdata dir>
```

After the first run, the media directory is created automatically (e.g. `/mnt/user/ori/91`); make sure your user shares (e.g. `media`) allow the same permissions.

## compose.yaml reference

| Setting | Default | Description |
| ------ | -------- | ------ |
| Port | `8620:8620` | Web UI (downloading + playback) |
| `/data` | `/mnt/user/ori/91` | Media library path — point it at where your videos live |
| `/appdata` | `/mnt/user/appdata/videodock/data` | Task state and other app data (survives image updates) |
| `PAGE_PROXY` | `http://192.168.6.111:7890` | Proxy for fetching video pages (91porn rate-limits/blocks direct access from mainland China; YouTube/Vimeo also frequently need one). **Replace with your own proxy.** |
| `CDN_PROXY` | same as `PAGE_PROXY` | Proxy for 91porn video streams (~1MB/s vs ~28KB/s direct). Set to `""` to force a direct connection |
| `YTDP_PROXY` | same as `PAGE_PROXY` | Proxy for the yt-dlp engine (YouTube, Bilibili, ...), can be set separately |
| `YTDP_COOKIES` | not set | Path to a Netscape-format `cookies.txt` for sites that require login (e.g. some YouTube/Vimeo videos, Bilibili member content) |

## Typical use

1. Open `http://<ip>:8620`, paste a video URL (91porn link/viewkey, YouTube/Bilibili/any yt-dlp-supported site) and press Enter
2. Downloads run in the background with live progress; completed videos appear in the library automatically
3. Click a card to play it in the browser (MP4, native playback)
4. Library layout per video: `<title>/video.mp4 + poster.jpg + folder.nfo`
5. Proxy, cookies and per-video renames can be managed from the settings page — changes take effect immediately, no restart needed

## Image

`bear0328/videodock:latest` — public on Docker Hub, pullable without login.

## Updating

```bash
docker compose pull -f compose.yaml
docker compose up -d -f compose.yaml
```

All existing videos remain in place (the library is scanned from disk); task state persists in the appdata volume.

## Tarball deployment (alternative)

For setups without docker compose: `docker load -i videodock.tar` followed by running the container with the same port/volume/env settings as above.

---

# videodock-compose 中文

[videodock](https://github.com/bear0328/videodock) 的单容器部署包——通用视频下载 + Web 播放平台。

## 功能

- **91porn** 及整个 91 系（91porn / 91pops / my91abc / tuchek / 69tuchk）内置适配器——大多数视频不需要 Cookie
- **YouTube、Bilibili、Vimeo 及 1000+ 站点**走内置 yt-dlp 引擎
- **单个容器**包含 Web UI + 下载器；无数据库、无独立 worker
- 后台下载队列（实时进度）、浏览器即点即播（MP4）、内置媒体库界面（网格 / 标题搜索 / Tab 筛选）
- 应用内代理与 Cookie 管理、视频改名

## 快速开始（Docker Compose）

```bash
docker compose pull -f compose.yaml
docker compose up -d -f compose.yaml
# 打开 http://<服务器IP>:8620
```

或把 `compose.yaml` 放进 Unraid 应用目录，用 Docker Compose 插件管理。

### Unraid 首次运行（appdata 权限）

容器以用户 `1000`（组 `1001`）运行。**appdata** 卷必须允许该用户写入，否则任务会报数据目录 *permission denied*：

```bash
chmod 775 /mnt/user/appdata/videodock   # 或：chown -R 1000:1001 <appdata目录>
```

首次运行后媒体目录会自动创建（如 `/mnt/user/ori/91`）；请确认对应共享盘（如 `media`）权限一致。

## compose.yaml 说明

| 配置 | 默认值 | 说明 |
| ------ | -------- | ------ |
| 端口 | `8620:8620` | Web UI（下载 + 播放） |
| `/data` | `/mnt/user/ori/91` | 媒体库目录，指向你的视频存储位置 |
| `/appdata` | `/mnt/user/appdata/videodock/data` | 任务状态等应用数据（镜像升级后保留） |
| `PAGE_PROXY` | `http://192.168.6.111:7890` | 抓取视频页面的代理（91porn 大陆 IP 直连会被限/封，YouTube/Vimeo 也常需要）。**换成你自己的代理地址** |
| `CDN_PROXY` | 同 `PAGE_PROXY` | 91porn 视频流下载走的代理（~1MB/s，直连仅 ~28KB/s）；代理线路差可设空串强制直连 |
| `YTDP_PROXY` | 同 `PAGE_PROXY` | 通用引擎（yt-dlp）走的代理，可单独设置 |
| `YTDP_COOKIES` | 未设置 | Netscape 格式 cookies.txt 路径，用于需要登录的站点（如部分 YouTube/Vimeo 视频、Bilibili 会员内容） |

## 使用说明

1. 打开 `http://<IP>:8620`，粘贴视频链接（91porn 链接或 viewkey、YouTube/Bilibili 等任意支持站点），回车开始下载
2. 下载在后台进行，进度实时显示；完成后视频库自动出现卡片
3. 点击卡片直接播放（MP4，浏览器原生播放）
4. 媒体目录结构：`<标题>/video.mp4 + poster.jpg + folder.nfo`
5. 代理、Cookie、视频改名均可在设置页操作——即时生效，无需重启

## 镜像

`bear0328/videodock:latest`（Docker Hub 公开，无需登录即可 pull）

## 更新

```bash
docker compose pull -f compose.yaml
docker compose up -d -f compose.yaml
```

已有视频全部保留（库从磁盘扫描）；任务状态保存在 appdata 卷中。

## tar 包部署（备选）

没有 docker compose 的环境：`docker load -i videodock.tar`，然后按上面的端口/卷/环境变量设置运行容器即可。
