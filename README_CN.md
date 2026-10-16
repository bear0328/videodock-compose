**中文** | [English](README.md)

# videodock-compose（中文）

videodock 部署预设——多站点视频下载 + Web 播放平台,
单容器(为 Unraid 设计)。

**支持站点:** 91porn · Pornhub · Bilibili · 抖音(Douyin) · YouTube · TikTok · X/Twitter,以及通过 yt-dlp 支持的 500+ 站点——一个自托管媒体服务器与视频下载器,为 Unraid 设计。

- **91 系(91porn / bilithings,…)走专用适配器,自带海报 + 元数据
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

界面 **验证** 按钮会从容器实际访问各站点,返回 *有效 / 无效*,粘完即可确认,不用等
下载失败才知道不对。

### 怎么获取 Cookie

videodock 要的是浏览器**原始 `Cookie` 请求头**——即浏览器实际发出的那串
(`name=value; name2=value2; …`)。用 DevTools 提取:

1. 在浏览器登录目标站点。
2. 打开 DevTools(F12)→ **Network**(网络),刷新页面,点击任意一个访问该站点的请求。
3. 在请求详情里找到 **Request Headers**(请求头),复制 `Cookie:` 这一行的完整值。
4. 粘到 *设置* 里对应站点的框,按 **验证**。

B 站也可以直接从 *Application → Cookies* 只复制这三个关键字段再拼起来:

```text
SESSDATA=…; bili_jct=…; dediuserid=…
```

`SESSDATA` 是最关键的 token;没有它只能拿到游客权限。

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
