# 安装与升级

本文说明 Amane 支持的安装方式、首次登录、升级和数据备份。桌面版与 Docker 运行同一个服务，功能一致；Android 只是远程客户端，需要连接一台已经部署的 Amane。

## 选择安装方式

| 方式 | 适用场景 | 数据目录 |
|---|---|---|
| Docker | NAS、家庭服务器、需要 24 小时运行 | 容器内 `/data`，建议映射到宿主机 |
| Windows 桌面版 | 在 Windows 电脑上单独运行 | `%LOCALAPPDATA%\Amane` |
| macOS 桌面版 | 在 macOS 13+ 上单独运行 | `~/Library/Application Support/Amane` |
| Android 客户端 | 访问已经部署的服务 | 不保存服务端数据 |

需要长期运行定时任务、订阅和文件监控时，优先使用 Docker。

## Docker 安装

> [!NOTE]
> Docker 镜像：`ghcr.io/zipenok/amane`。

### 1. 准备目录

```bash
git clone https://github.com/ZiPenOk/Amane.git
cd Amane
mkdir -p data
```

### 2. 设置运行用户

Linux 下建议让容器用户与宿主机用户一致，避免写入的数据库、图片和媒体文件变成 root 所有：

```bash
printf 'UID=%s\nGID=%s\n' "$(id -u)" "$(id -g)" > .env
```

生成的 `.env` 内容示例：

```dotenv
# 容器运行用户的 UID/GID；Linux 下建议与宿主机用户一致
UID=1000
GID=1000
```

Windows Docker Desktop 可以直接使用默认的 `1000:1000`，或按实际挂载权限调整。

### 3. 创建 `compose.yaml`

仓库中的 `compose.yaml` 内容如下：

```yaml
services:
  amane:
    image: ghcr.io/zipenok/amane:latest
    container_name: amane
    user: "${UID:-1000}:${GID:-1000}"
    ports:
      - "8016:8016"
      # 可选的 Emby 302 网关
      - "8017:8017"
    volumes:
      - ./data:/data
      - /path/to/media:/media
    environment:
      - AMANE_PORT=8016
      - AMANE_DATA_DIR=/data
      - AMANE_SAFE_DIRS=/media
      - AMANE_SUPERVISED=1
    restart: unless-stopped
    healthcheck:
      test:
        - CMD
        - python
        - -c
        - >-
          import http.client, os, sys;
          port = int(os.environ.get("AMANE_PORT", "8016"));
          conn = http.client.HTTPConnection("127.0.0.1", port, timeout=3);
          conn.request("GET", "/api/health");
          sys.exit(0 if conn.getresponse().status == 200 else 1)
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 10s
```

至少检查以下内容：

- `./data:/data`：Amane 数据库、配置、缓存和日志
- `/path/to/media:/media`：本地媒体目录
- `AMANE_SAFE_DIRS=/media`：允许 Amane 访问的容器内路径
- `8016:8016`：Web UI 与 API
- `8017:8017`：可选的 Emby 302 网关

有多个媒体目录时逐个挂载，并把所有容器内路径写入 `AMANE_SAFE_DIRS`，以逗号分隔。

只使用元数据、不管理本地文件时，可以删除媒体目录挂载与 `AMANE_SAFE_DIRS`。

### 4. 启动

```bash
docker compose pull
docker compose up -d
docker compose ps
```

查看日志：

```bash
docker compose logs -f amane
```

### 5. 获取 API Token

首次启动会生成 Token，并保存在：

```text
./data/token
```

也可以从日志中查找：

```bash
docker compose logs amane | grep "api token"
```

浏览器访问：

```text
http://<主机地址>:8016
```

粘贴 Token 登录。

## Windows 桌面版

1. 打开 [Amane Releases](https://github.com/ZiPenOk/Amane/releases)。
2. 下载 `Amane-v*-windows-x64.zip`。
3. 解压到任意目录。必须保留完整目录结构，`Amane.exe` 依赖同目录下的 `onedir` 与 `web` 文件夹。
4. 运行 `Amane.exe`，系统托盘会出现 Amane 图标。
5. 从托盘菜单选择“打开 Web UI”，或访问 `http://127.0.0.1:18000`。

数据目录为 `%LOCALAPPDATA%\Amane`。

## macOS 桌面版

1. 打开 [Amane Releases](https://github.com/ZiPenOk/Amane/releases)。
2. 下载 `Amane-v*-macos.app.zip`。
3. 解压后将 `Amane.app` 拖入“应用程序”。
4. 启动 Amane，从菜单栏图标选择“打开 Web UI”，或访问 `http://127.0.0.1:18000`。

需要 macOS 13 或更高版本。数据目录为 `~/Library/Application Support/Amane`。

## Android 客户端

Android 应用不包含服务端，必须先部署 Amane 服务。

从 [Amane Releases](https://github.com/ZiPenOk/Amane/releases) 中选择 `app-*` 开头的发布，安装 APK 后填写服务地址和 API Token。

在局域网外访问时，建议通过 VPN、Tailscale 或带 HTTPS 的反向代理连接到服务，不要直接把管理端口暴露到公网。

## 升级

### Docker

```bash
docker compose pull
docker compose up -d
```

升级前建议备份整个 `data` 目录。Amane 启动时会自动执行数据库迁移，并在迁移前生成备份。

### 桌面版

下载新版本压缩包，退出旧版本后替换程序目录。不要删除数据目录。

## 备份

最重要的是 `data` 目录，其中包含：

- SQLite 数据库
- API Token 与配置
- 图片、头像和资源缓存
- 日志

媒体文件本体不在 `data` 中。备份时建议停止容器，或至少使用 SQLite 一致性备份方式：

```bash
docker compose stop amane
tar -czf amane-data-backup.tgz data
docker compose start amane
```

## 远程访问

默认建议仅监听本机或局域网地址。需要远程访问时：

1. 优先使用 VPN / Tailscale 等私有网络。
2. 或使用 HTTPS 反向代理。
3. 不要在公网直接暴露无认证的 API、媒体目录或 Docker Socket。
4. 保管好 API Token；持有 Token 即拥有服务的主要管理权限。