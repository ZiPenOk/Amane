# 故障排查

## `ghcr.io/zipenok/amane` 拉取失败

先确认：

1. `Amane → Actions → Publish container` 已经成功运行。
2. Package visibility 是 `Public`。
3. 镜像名是小写：`ghcr.io/zipenok/amane`。
4. 没有混用其它项目的历史镜像名。

## 登录时不知道 API Token

Docker：

```bash
cat data/token
```

或：

```bash
docker compose logs amane | grep "api token"
```

桌面版从菜单栏或托盘图标选择“复制 API Token”。

## 容器无法写入数据或媒体目录

检查 `compose.yaml`：

- `UID` / `GID` 是否与宿主目录所有者一致
- `/data` 和媒体目录是否可写
- `AMANE_SAFE_DIRS` 是否包含所有已挂载的容器内路径
- 多个路径是否已按逗号分隔

## 媒体库扫描不到文件

检查：

- 文件扩展名是否在允许列表
- 子目录是否开启递归
- 文件是否命中黑名单
- 文件大小是否低于最小限制
- 路径是否位于 `.amane_trash`
- CloudDrive 挂载路径是否填写正确
- Docker 中的宿主路径是否已经映射到容器内，并与媒体库配置一致

## 刮削不到元数据

按任务记录查看各站点失败原因，常见原因：

- 站点不可达或要求代理
- 站点需要 Cookie / 登录
- 当前内容类型没有可用站点
- 字段优先级或排除规则过滤掉了结果
- 番号与站点记录不一致

## Emby / Jellyfin 看不到更新

确认：

- Amane 中的整理任务已成功
- NFO、图片与视频路径能被媒体服务器读取
- 媒体服务器已经刷新对应条目
- Amane 与媒体服务器没有同时写入 NFO

## 演员推送没有更新人物

演员推送只更新媒体服务器上已经存在的人物：

- 先让媒体服务器扫描影片并生成 Person 条目
- 在“设置 → 媒体服务器”填写 URL 和管理员 API Key
- 自动推送需要在“演员刮削”中单独开启
- 默认只补空字段和缺失头像；需要覆盖时使用“强制覆盖”
- 本地没有缓存头像时不会在推送任务中临时下载

## 提交问题

代码、构建和发布相关问题请提供：

- Amane 版本或 Docker 镜像 tag
- 部署平台
- 相关任务 ID
- 已脱敏的日志
- 复现步骤

问题入口：

```text
https://github.com/ZiPenOk/Amane/issues
```