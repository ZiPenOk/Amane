# Amane 文档

本目录面向安装和使用 Amane 发行版的用户。开发源码、内部架构和构建文档不对外公开。

| 文档 | 内容 |
|---|---|
| [安装与升级](installation.md) | Docker、Windows、macOS、Android、首次登录、升级与备份 |
| [首次使用](quickstart.md) | 配置媒体库、提交刮削、查看结果、接入 Emby / Jellyfin |
| [与上游原版的主要区别](differences.md) | 115 入库、STRM、字幕、数据源和界面差异 |
| [Packages 与镜像发布](packages.md) | GitHub Packages、GHCR 镜像、仓库关联与发布流程 |
| [故障排查](troubleshooting.md) | 镜像、登录、权限、刮削和媒体服务器问题 |

## 推荐阅读顺序

1. 按 [安装与升级](installation.md) 启动服务。
2. 使用 API Token 登录 Web UI。
3. 按 [首次使用](quickstart.md) 完成第一次刮削。
4. 需要 Docker 镜像发布时阅读 [Packages 与镜像发布](packages.md)。