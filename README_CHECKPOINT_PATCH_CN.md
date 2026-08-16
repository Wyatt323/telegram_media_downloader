# Telegram Media Downloader — checkpoint patch

本仓库基于 [tangyoha/telegram_media_downloader](https://github.com/tangyoha/telegram_media_downloader)，修复了 Docker 常驻运行时下载完成却不立即写入 `last_read_message_id` 的问题。

## 修复内容

上游版本仅会在进程退出（例如收到 `SIGINT`）时执行 `app.update_config()`。本版本会在所有频道的当前下载批次完成后，立即将：

- `config.yaml` 中的 `chat[].last_read_message_id`
- `data.yaml` 中的失败重试消息列表

写入挂载在容器外的文件。程序退出时仍会再次保存，作为额外保护。

## 镜像

GitHub Actions 会将镜像发布至 GitHub Container Registry（GHCR）：

```bash
docker pull ghcr.io/wyatt323/telegram_media_downloader:latest
```

首次拉取私有 GitHub Package 时需要 `docker login ghcr.io`，或在 GitHub 仓库的 Packages 设置中将该镜像改为 Public。公开后可直接拉取，不需要 Docker Hub。

## Docker Compose 使用

将服务镜像改为：

```yaml
services:
  telegram_media_downloader:
    image: ghcr.io/wyatt323/telegram_media_downloader:latest
    restart: unless-stopped
    volumes:
      - ./downloads/:/app/downloads/
      - ./config.yaml:/app/config.yaml
      - ./data.yaml:/app/data.yaml
      - ./log/:/app/log/
      - ./sessions/:/app/sessions
      - ./temp/:/app/temp
```

更新：

```bash
docker compose pull
docker compose up -d
```

> CloudDrive2 上传方案按当前需求未改动。
