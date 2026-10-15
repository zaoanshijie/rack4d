# rack4d

使用 GitHub Actions 基于 [rsdk](https://github.com/RadxaOS-SDK/rsdk)（RadxaOS 官方 SDK）编译 [Radxa ROCK 4D](https://docs.radxa.com/rock4/rock4d) 的 RadxaOS 镜像。

## 镜像

| 镜像 | 说明 |
| ---- | ---- |
| `radxa-rk3576_bookworm_kde` | KDE Plasma 桌面版，与官方发布的桌面镜像一致 |
| `radxa-rk3576_bookworm_cli` | CLI 精简版，无桌面环境，适合服务器用途 |

- 均基于 Debian 12（bookworm）
- 均默认安装并启用 SSH（`openssh-server`），开机即可连接
- 标准 512 字节扇区镜像（SD / eMMC 启动），首次开机自动扩容根分区
- 使用 Radxa APT test 通道软件源构建（rk3576 内核目前仅发布于 test 通道，与 rsdk 官方 CI 构建方式一致）

## 版本命名与每日同步

Release 以**编译所用 rsdk 提交的前 12 位 hash** 命名（如 `6340851ff7d9`），镜像文件名同理。对应的完整提交见 Release 说明。

每日 00:00（北京时间）定时执行：

1. 检测 [radxa-build/radxa-rk3576](https://github.com/radxa-build/radxa-rk3576) 的最新官方 release
2. 检测 [RadxaOS-SDK/rsdk](https://github.com/RadxaOS-SDK/rsdk) main 分支最新提交
3. 只要 rsdk main 出现尚未编译过的新提交，就编译一份 KDE + CLI 镜像并发布为新 Release（tag = 提交 hash）；如果该提交已有对应 Release，则跳过（已在 Release 说明中标注当时的官方最新版本）

> 说明：官方仓库构建时始终使用 rsdk main，且 release 元数据不固定 rsdk 版本，因此“同步官方”即编译当前 rsdk main（与官方 CI 行为一致），并在 Release 说明中标注当时的官方最新 release。

## 触发编译

- **每日定时**：北京时间 00:00 自动检测
- **推送**：修改 `.github/workflows/build.yml` 并推送到 `main` 时，强制用 rsdk main 编译一次（验证 workflow）
- **手动**：Actions 页面运行 workflow，可指定 rsdk ref；勾选 `force` 可在已有 Release 时强制重编

> 注意：GitHub 会在仓库连续 60 天无任何提交时自动禁用定时任务；如发现定时任务停了，随便提交一次或手动运行即可恢复。

## 烧录

```bash
xzcat radxa-rk3576_bookworm_kde_*.output_512.img.xz | sudo dd of=/dev/sdX bs=4M conv=fsync status=progress
```

也可使用 balenaEtcher / Rufus 写入解压后的 `img` 文件。

## SSH

镜像内默认安装 `openssh-server` 并随系统启动（构建时会校验 manifest 中包含 `openssh-server`）。首次开机按提示完成 rsetup 初始配置（RadxaOS 默认账号 `radxa` / `radxa`，以官方文档为准），随后即可连接：

```bash
ssh radxa@<板子IP>
```

## 故障诊断

构建失败时，workflow 会自动在 [issues](../../issues) 中记录失败日志（标题 `[diagnostics] build-failure-log`），便于排查。
