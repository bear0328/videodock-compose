# 91video 部署包

91porn.com 视频下载 + Web 播放。Docker 镜像公开，源码闭源（私有仓库）。

## 快速开始

```bash
mkdir -p /mnt/user/appdata/91video   # 按你的情况调整
docker compose pull -f compose.yaml
docker compose up -d -f compose.yaml
# 打开 http://<服务器IP>:8620
```

或直接把 `compose.yaml` 放到 Unraid 应用目录用 Docker Compose 管理。

## compose.yaml 说明

| 配置 | 默认值 | 说明 |
| ------ | -------- | ------ |
| 端口 | `8620:8620` | Web UI（下载 + 播放） |
| `/data` | `/mnt/user/media/ori/91` | 媒体库目录，改成你存储视频的位置 |
| `/appdata` | `/mnt/user/appdata/91video/data` | 任务状态等应用数据 |
| `PAGE_PROXY` | `http://192.168.6.111:7890` | **必填**：抓 91porn 页面的代理（大陆 IP 直连会被限/封），换成你自己的代理地址 |
| `CDN_PROXY` | 同 `PAGE_PROXY` | 视频下载走的代理，默认跟 PAGE_PROXY 一致（~1MB/s，直连仅 ~28KB/s）；代理线路差可设空串强制直连 |

## 使用说明

1. 打开 `http://<IP>:8620`，粘贴 91porn 视频链接（或 viewkey），回车开始下载
2. 下载在后台进行，进度条实时显示；完成后视频库自动出现卡片
3. 点击卡片直接播放（MP4 H.264，浏览器原生播放）
4. 媒体目录结构：`<标题>/video.mp4 + poster.jpg + folder.nfo`

## 镜像

- `bear0328/91video:latest`（Docker Hub 公开，无需登录即可 pull）

## 更新

```bash
docker compose pull -f compose.yaml
docker compose up -d -f compose.yaml
```
