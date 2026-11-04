# 浏览器脚本与 Amane 入库

浏览器脚本把 JavBus、JavDB、JavLibrary 和 115 页面中的资源提交到 Amane，由 Amane 创建入库任务。安装脚本后，既可以在 115 文件页导入已有文件或目录，也可以让脚本把磁力链接或 ed2k 文件链接提交为 Amane 入库任务。

脚本由 [ZiPenOk/scripts](https://github.com/ZiPenOk/scripts) 维护。

## 安装

浏览器需要先安装 Tampermonkey、Violentmonkey 或兼容的用户脚本管理器。

安装地址：

```text
https://github.com/ZiPenOk/scripts/raw/refs/heads/main/laosiji-amane.js
```

在已安装脚本管理器的浏览器中打开该地址，确认安装。安装完成后刷新 JavBus、JavDB、JavLibrary 或 115 页面。

脚本更新时重新打开安装地址并覆盖安装即可。

## 准备 Amane

Amane 需要满足以下条件：

1. 服务已经启动，浏览器能够访问 Amane 地址。
2. 115 连接已经完成认证。
3. 至少有一个用于入库的媒体库。
4. 已取得 `javxy` 集成 Token。

脚本只提交入库请求和查询提交回执，任务状态仍由 Amane 管理。

### 获取 javxy 集成 Token

`javxy` 集成 Token 与登录 Web UI 使用的主 API Token 不同。它只能访问 `/api/integrations/javxy/` 集成接口。

按安装方式查找：

| 安装方式 | Token 位置 |
|---|---|
| Docker | 数据目录的 `secrets/javxy-token`，默认即 `./data/secrets/javxy-token` |
| Windows 桌面版 | `%LOCALAPPDATA%\Amane\secrets\javxy-token` |
| macOS 桌面版 | `~/Library/Application Support/Amane/secrets/javxy-token` |

设置了环境变量 `AMANE_JAVXY_TOKEN` 时，以环境变量为准。值为 `off` 时关闭集成。

不要填写主 API Token，也不要填写 Web UI 登录 Cookie。

## 配置脚本

打开脚本管理器菜单，选择 **Amane 入库设置**。

需要填写：

| 配置 | 说明 |
|---|---|
| 推送至 Amane | 开启后，脚本将入库请求提交给 Amane，而不是直接走普通 115 流程 |
| 每次推送都询问媒体库 | 开启后每次提交都选择媒体库；关闭后使用已保存媒体库 |
| Amane 地址 | Amane 根地址，例如 `http://192.168.1.10:8016` 或 `http://127.0.0.1:18000` |
| javxy 集成 Token | 上节取得的 `javxy` 集成 Token |
| 媒体库 | 点击“加载”后从列表选择，或直接填写媒体库 ID |

地址只填写 Amane 根地址，不要附加 `/api/integrations/javxy` 或其它接口路径。

Docker 默认端口为 `8016`，桌面版默认端口为 `18000`。如果浏览器不在 Amane 所在主机上，不能使用 `127.0.0.1`，应填写局域网地址、VPN 地址或反向代理地址。

点击“加载”后，脚本会读取可选媒体库。只生成 STRM、不执行入库的媒体库不会出现在列表中。

## 使用

### 导入 115 已有文件或目录

1. 打开 115 网盘文件页。
2. 选择一个或多个文件或目录。
3. 右键选择 **Amane 入库**。
4. 如果开启了“每次推送都询问媒体库”，选择目标媒体库。

脚本会为选中项创建 Amane 入库任务。多选时逐项创建任务。页面右下角会显示提交状态，并提供 Amane 入库任务页入口。

### 推送磁力链接或 ed2k 文件链接

在 JavBus、JavDB、JavLibrary 或 115 页面使用脚本的 115 离线/入库操作时，如果 **推送至 Amane** 已开启，脚本会把链接提交给 Amane，由 Amane 创建入库任务并继续后续的离线下载、整理、STRM 生成与 Emby 刷新。

支持：

- `magnet:` 链接
- `ed2k://|file|...` 文件链接

关闭 **推送至 Amane** 后，下一次操作恢复为普通 115 流程。

## 查看任务

提交成功后，在 Amane 的 **Amane 入库** 页面查看任务阶段、进度、事件和文件明细：

```text
http://<Amane 地址>/amane?tab=acquisitions
```

脚本的右下角任务卡片也会显示提交状态和任务入口。

## 常见问题

### 提示 Token 无效

确认填写的是 `secrets/javxy-token` 的内容，不是数据目录中的主 `token`，也不是主 API Token。

### 媒体库列表为空

检查：

- Amane 地址可以访问；
- `javxy` Token 正确；
- 115 连接已经认证；
- 已创建完整入库媒体库；
- 该媒体库不是仅生成 STRM 的模式。

### 提交失败或无法连接 Amane

先在浏览器中访问：

```text
http://<Amane 地址>/api/health
```

如果无法访问，检查端口、局域网路由、防火墙、VPN 或反向代理。远程使用时优先通过 VPN、Tailscale 或 HTTPS 反向代理访问，不要直接暴露管理端口。

### 右键菜单没有“Amane 入库”

确认：

- 脚本管理器已启用脚本；
- 当前页面为 115 文件页；
- 已刷新页面；
- 脚本版本已更新；
- 已至少选择一个文件或目录。

### 想恢复普通 115 流程

在 **Amane 入库设置** 中关闭 **推送至 Amane**。下一次操作会回到脚本原有的 115 流程。

## 安全提示

- `javxy` Token 能直接提交 Amane 入库任务，应妥善保管。
- 不要把 Token 写入公开配置、截图或日志。
- Token 泄露后，替换数据目录中的 `secrets/javxy-token`，或修改 `AMANE_JAVXY_TOKEN`，并同步更新脚本配置。
- 不要把 Amane 管理端口直接暴露到公网；远程访问使用 VPN、Tailscale 或带认证的 HTTPS 反向代理。
