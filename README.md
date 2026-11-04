<div align="center">

<img src="assets/logo.svg" width="96" alt="Amane logo">

# Amane

**面向个人影音库的多源元数据管理、整理与自动化服务**

多源刮削 · 字段级择优 · 本地数据所有权 · STRM · 任务与订阅 · AI 智能体 · Emby / Jellyfin

[![Version](https://img.shields.io/badge/version-0.25.0-blue)](https://github.com/ZiPenOk/Amane/releases)
[![Container](https://img.shields.io/badge/container-ghcr.io%2Fzipenok%2Famane-2496ED?logo=docker&logoColor=white)](https://github.com/ZiPenOk/Amane/pkgs/container/amane)
[![License](https://img.shields.io/badge/license-GPLv3-blue.svg)](LICENSE)

[安装说明](docs/installation.md) · [快速开始](docs/quickstart.md) · [文档目录](docs/README.md) · [下载](https://github.com/ZiPenOk/Amane/releases)

</div>

---

> [!IMPORTANT]
> 本仓库是 Amane 的**发行与用户文档仓库**。开发源码、内部构建文档和发布基础设施不对外公开。
>
> - 上游开源项目：[sqzw-x/amane](https://github.com/sqzw-x/amane)
> - 容器镜像目标名称：`ghcr.io/zipenok/amane`
>
> 镜像需要先在本仓库 Actions 中运行一次 `Publish container` 工作流。桌面版、Android 与 CLI 产物统一通过本仓库 Releases 发布。

## Amane 是什么

Amane 是一个以影片条目为核心的个人影音库元数据服务。

它负责从多个来源抓取、合并和管理元数据，并可按需把结果整理为 NFO、海报、封面、剧照和字幕；Emby / Jellyfin 继续负责媒体文件管理、播放与串流。

Amane **不是** Emby / Jellyfin 的替代品，而是它们的元数据补充，也可以完全独立地作为番号资料库使用。

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/screenshots/meta-wall-dark.webp">
  <img src="assets/screenshots/meta-wall.webp" alt="Amane 片库海报墙">
</picture>

## 主要能力

| 能力 | 说明 |
|---|---|
| **多源字段聚合** | 按内容类型配置站点顺序，每个字段可单独设置优先级或排除来源 |
| **保留原始数据** | 站点原始快照、图片和译文保存在本地 SQLite 与资源目录，来源下线后仍可查看历史数据 |
| **可审计的元数据管理** | 对比来源、字段锁定、分类治理、评论、用户标签、逐字段修订 |
| **丰富的番号来源** | 手动提交、模板批量生成、RSS / Atom 订阅、定时重新刮削、媒体库扫描、智能体与 CLI |
| **媒体库与 STRM** | 本地文件扫描、115 入库、STRM 生成、批量命名、路径模板、整理与删除任务 |
| **Emby / Jellyfin 衔接** | 整理完成后写入 NFO、图片与字幕；支持刷新媒体库、演员资料与头像推送 |
| **AI 智能体** | 在应用内用自然语言查询、整理标签、提交刮削；外部智能体可通过 CLI 操作 |
| **语义检索** | 使用本地向量索引检索封面与剧照 |
| **多端访问** | macOS / Windows 桌面应用、Docker 服务端、Android 客户端与浏览器访问 |

## 典型使用方式

### 纯元数据管理

直接提交番号即可获得多源合并后的影片资料、演员信息和图片，不需要本地视频文件。

### 本地媒体库整理

为影片目录建立媒体库，扫描文件名识别番号，刮削后按模板整理并写出 NFO、图片和字幕，供 Emby / Jellyfin 使用。

### 115 / STRM 播放

管理 115 网盘内容，生成 STRM 指针，并通过 Emby 网关或播放代理访问远端媒体。

## 安装方式

| 平台 | 推荐场景 | 获取方式 |
|---|---|---|
| Docker | NAS、家庭服务器、需要长期运行 | `ghcr.io/zipenok/amane` |
| Windows 桌面版 | 在 Windows 电脑上单独运行 | 本仓库 Releases 中的 `Amane-v*-windows-x64.zip` |
| macOS 桌面版 | 在 macOS 13+ 上单独运行 | 本仓库 Releases 中的 `Amane-v*-macos.app.zip` |
| Android 客户端 | 访问已经部署的 Amane 服务 | 本仓库 Releases 中 `app-*` 开头的发布 |

完整步骤见 **[安装说明](docs/installation.md)**。

### Docker 快速开始

```bash
git clone https://github.com/ZiPenOk/Amane.git
cd Amane
cp .env.example .env
# 编辑 .env 与 compose.yaml 中的媒体目录
docker compose pull
docker compose up -d
```

启动后访问 `http://<主机地址>:8016`。API Token 位于宿主机数据目录的 `data/token`，也可以从容器日志获取。

## 文档

- [安装与升级](docs/installation.md)
- [首次使用](docs/quickstart.md)
- [Packages 与镜像发布](docs/packages.md)
- [故障排查](docs/troubleshooting.md)

## 项目关系

- **本仓库**：发行说明、用户文档、`compose.yaml`、容器发布工作流
- **上游项目**：`sqzw-x/amane` 是项目最初的开源实现

该发行版包含个人维护者新增或调整的功能，因此行为可能与本仓库以外的版本不同。

## 许可与声明

本项目使用 [GNU General Public License v3.0](LICENSE)。

Amane 不提供、托管或分发任何影片内容。请仅管理你自行取得且有权访问的内容，并遵守所在地法律及服务条款。