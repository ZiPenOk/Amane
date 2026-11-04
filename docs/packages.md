# GitHub Packages 与镜像发布

本文说明 GitHub Releases、GitHub Packages 和本发行仓库之间的关系，以及如何得到正确的 GHCR 镜像。

## 先区分 Releases 和 Packages

| 对象 | 用途 | Amane 中的内容 |
|---|---|---|
| **Releases** | 版本页面与可下载文件 | Windows zip、macOS zip、Android APK、CLI 压缩包、校验文件 |
| **Packages** | 包注册表中的可拉取制品 | Docker / GHCR 容器镜像 |

桌面版和 Android APK 不适合作为 Package，继续放在 Releases。Docker 镜像使用 GitHub Container Registry。

## 当前状态

当前没有名为 `amane` 的 GHCR Package。用户 Packages 页面中的其它容器包与本项目无关。

`https://github.com/users/ZiPenOk/packages?repo_name=<repository>` 中的 `repo_name` 只是 Packages 列表的仓库筛选参数，不是独立的存储位置。Package 本身属于用户命名空间 `ZiPenOk`，再关联到一个仓库。

## 推荐方案

公开面只保留本仓库：

```text
ZiPenOk/Amane
```

容器镜像使用：

```text
ghcr.io/zipenok/amane
```

开发源码和内部构建仓库不对外公开。它们通过本仓库的 Actions Secret 提供给发布 workflow，不出现在 README、用户文档或 workflow 文件中。

## 发布前配置 Secrets

进入：

```text
ZiPenOk/Amane → Settings → Secrets and variables → Actions
```

添加两个 Repository secrets：

| Secret | 内容 |
|---|---|
| `SOURCE_REPOSITORY` | 内部源码仓库的 `owner/repo` |
| `SOURCE_REPO_TOKEN` | 有该源码仓库 Contents: Read 权限的细粒度 Token |

`SOURCE_REPOSITORY` 使用 Secret 而不是 Repository variable，避免仓库名直接出现在公开配置中。Token 不要提交到仓库，也不要写进文档。

## 发布 Workflow

仓库已经包含：

```text
.github/workflows/publish-container.yml
```

该 workflow：

1. 在 `ZiPenOk/Amane` 中运行
2. 从 Secret 读取内部源码仓库和访问 Token
3. 检出指定分支或 tag
4. 使用源码中的 Dockerfile 构建镜像
5. 发布到 `ghcr.io/zipenok/amane`
6. 生成版本、次版本和 `latest` 标签

这样既不会把源码复制到分发仓库，也不会在公开 workflow 中暴露内部仓库地址。

## 首次发布

进入：

```text
ZiPenOk/Amane → Actions → Publish container → Run workflow
```

建议第一次填写：

| 输入 | 示例 |
|---|---|
| `source_ref` | `dev` |
| `version` | `0.25.0` |
| `platforms` | `linux/amd64` |
| `latest` | `true` |

成功后镜像地址为：

```text
ghcr.io/zipenok/amane:latest
ghcr.io/zipenok/amane:0.25.0
ghcr.io/zipenok/amane:0.25
ghcr.io/zipenok/amane:0
```

也可以在本仓库发布 `v*` Release 时自动触发。注意：由另一个 workflow 使用默认 `GITHUB_TOKEN` 创建的 Release 不会再次触发 workflow；这种情况应直接手动运行 `Publish container`。

## 发布后检查

1. 打开 `ZiPenOk/Amane` 的 Packages 页面，确认出现 `amane`。
2. 打开 Package Settings，确认它关联到 `ZiPenOk/Amane`。
3. 将 Package visibility 设为 `Public`，否则匿名用户无法拉取。
4. 检查镜像可拉取：

```bash
docker pull ghcr.io/zipenok/amane:latest
```

如果 workflow 使用本仓库的 `GITHUB_TOKEN` 发布，通常会默认关联当前仓库。没有自动关联时，在 Package Settings 中手动选择 `Amane`。

## Releases 的处理原则

所有用户可见的发行内容应统一从本仓库发布：

- Windows / macOS 桌面版
- Android APK
- CLI 压缩包与校验文件
- 版本说明和更新日志

不要在公开 README 或安装文档中链接内部源码仓库。内部构建仓库可以继续负责构建，但用户下载入口应保持为：

```text
https://github.com/ZiPenOk/Amane/releases
```

GitHub Releases 不能直接从一个仓库迁移到另一个仓库。如果已有内部仓库 Release，需要在发布流程中把产物重新上传到本仓库，而不是把用户引导到内部仓库。

## 不建议的做法

- 不要期待重命名仓库后 Package 会自动跟着改名或迁移。
- 不要把 `/users/ZiPenOk/packages?repo_name=...` 当作镜像地址；镜像地址必须写成 `ghcr.io/zipenok/<name>`。
- 不要在公开 workflow 中硬编码内部源码仓库地址。
- 不要把桌面版和 Android APK 上传成 npm、NuGet 等 Package；它们属于 Releases。
- 不要在安装文档中暴露内部构建地址、Token 或仓库结构。

## 清理策略

当前没有 Amane 容器包需要删除。首次使用新命名后：

- 保留 `ghcr.io/zipenok/amane`
- 如以后产生旧镜像，确认新镜像可用后再删除旧 Package
- 与本项目无关的其它容器包不要改动