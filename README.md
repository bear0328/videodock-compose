# videodock-compose

Deployment presets for [videodock](https://github.com/bear0328/videodock) — a multi-site video
downloader + web playback platform in a single Docker container (built for Unraid).

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

The in-app **Verify** button checks each site from the container and reports
*Valid / Invalid*. Step-by-step extraction guides (DevTools screenshots & token lists)
live in the [main repo README](https://github.com/bear0328/videodock#cookies-how-to-get-them).

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

# videodock-compose（中文）

[videodock](https://github.com/bear0328/videodock) 部署预设——多站点视频下载 + Web 播放平台,
单容器(为 Unraid 设计)。

- **91 系**(91porn / bilithings,…)走专用适配器,自带海报 + 元数据
- **YouTube、B 站、TikTok、X 及 1000+ 站点**走内置 yt-dlp 引擎
- **无数据库、无 worker**:一个容器搞定 Web UI、任务队列、播放
- **媒体库自动分类**到 成人 / 通用 两个挂载;应用内代理与 Cookie 管理(带实时验证)、
  改名 / 搜索 / 删除

## 快速开始（Docker Compose）

```bash
docker compose pull -f compose.yaml
docker compose up -d -f compose.yaml
# 打开 http://<服务器IP>:8620
```

Unraid 用户把 `compose.yaml` 放进 `/boot/config/videodock/`,用 Docker Compose 插件管理:

```bash
mkdir -p /boot/config/videodock /mnt/user/appdata/videodock/data
cp compose.yaml /boot/config/videodock/
```

把 `compose.yaml` 里两个媒体库路径改成你自己的目录(见下),然后按上面 pull & start。

### Unraid 首次运行（appdata 权限）

容器以用户 `1000`（组 `1001`）运行。**appdata** 卷必须允许该用户写入,否则任务报
*permission denied*:

```bash
chmod 775 /mnt/user/appdata/videodock   # 或:chown -R 1000:1001 /mnt/user/appdata/videodock
```

## 媒体库（双挂载）

videodock 会把每个下载自动分类到两个媒体库之一:**91 系**链接进 *adult* 挂载,其余
(yt-dlp 站点)进 *general* 挂载。`VIDEODB_DATA=/` 时正好对应 `/adult` 与 `/general`:

```yaml
volumes:
  - /mnt/user/media/ori/91:/adult          # 91 系媒体库
  - /mnt/user/downloads/youtube:/general   # 通用站点媒体库(yt-dlp)
  - /mnt/user/appdata/videodock/data:/appdata
environment:
  - VIDEODB_DATA=/
```

两个卷都必须存在且可写。想全部放一个目录树:把两个挂载指到同一宿主机路径即可(会自动
在其下建 `adult/` 与 `general/` 子目录)。

## compose.yaml 说明

**卷**

| 挂载 | 用途 |
| --- | --- |
| `/adult` | 成人媒体库(91 系) |
| `/general` | 通用媒体库(所有 yt-dlp 站点) |
| `/appdata` | 元数据、任务队列、设置、日志——镜像升级后保留 |

**环境变量**

| 变量 | 默认 | 说明 |
| --- | --- | --- |
| `PORT` | `8620` | Web 端口 |
| `VIDEODB_DATA` | `/data` | 媒体库根目录;视频落入 `<root>/adult` 或 `<root>/general`(预设为 `/`) |
| `APPDATA` | `/appdata` | 应用数据目录 |
| `PAGE_PROXY` | *(空)* | 抓 91 系页面与海报的代理。**服务器直连不到站点时必填。** 所有代理也可在界面设置页修改(免重启) |
| `CDN_PROXY` | 跟随 `PAGE_PROXY` | 91 CDN 视频流代理——通常远快于直连;设 `""` 强制直连 |
| `YTDP_PROXY` | 跟随 `PAGE_PROXY` | yt-dlp 引擎(YouTube、B 站…)的代理 |
| `YTDP_COOKIES` | *(空)* | Netscape `cookies.txt` 路径——应用内 Cookie 字段的可选替代 |

## Cookie 与登录内容

- **91 系**:公开视频免 Cookie;私密视频需登录。把浏览器原始 `Cookie` 请求头粘到
  *设置 → 91 系 Cookie*。
- **YouTube**:公开视频匿名可下;年龄限制 / 会员内容需登录 Cookie。
- **B 站**:会员 / 高清源需要 `SESSDATA`(复制完整 cookie)。

界面 **验证** 按钮会从容器实际访问各站点,返回 *有效 / 无效*。逐步提取指南(DevTools 与
关键 token 清单)见 [主仓库 README](https://github.com/bear0328/videodock#cookie-怎么获取)。

## 使用说明

1. 打开 `http://<IP>:8620`,粘贴链接(91porn、YouTube、B 站…),回车开始下载
2. 后台队列下载,实时进度与速度;完成后自动出现在对应媒体库(已分类)
3. 点卡片即播——HTTP Range 直出、进度条可拖、无转码
4. 媒体库:成人 / 通用双 Tab、标题搜索、改名(媒体 + 元数据一起迁移)、删除
5. 代理、Cookie、91 频道(CN/EN/AI…)都在设置页管理——即时生效,无需重启

## 更新

```bash
docker compose pull -f compose.yaml
docker compose up -d -f compose.yaml
```

媒体库从磁盘扫描,已有视频全部保留;任务状态与设置保存在 appdata 卷中。

## tar 包部署（备选）

无 docker compose 的环境:`docker load -i videodock.tar`,然后按上面的端口/卷/环境变量
运行容器即可。

## 常见问题

| 症状 | 解决 |
| --- | --- |
| 启动报数据目录 *permission denied* | `chmod 775 /mnt/user/appdata/videodock`(容器用户 1000) |
| 91 系页面慢 / 失败 | 设置 `PAGE_PROXY`(或设置页的页面代理)为可用代理 |
| 91 下载只有几 KB/s | 保持 `CDN_PROXY` 跟随 `PAGE_PROXY`;检查代理线路质量 |
| YouTube/B 站 403 或跳登录页 | 配好下载代理;受限内容粘贴 Cookie 后点 **验证** |
| 代理 `407 / Bad Gateway` | 换代理地址(或补认证)——用设置页 **验证** 确认连通 |
| 视频能播但海报慢 | 海报经应用代流、走页面代理——在设置页调整 |
