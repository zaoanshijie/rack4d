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
- 编译产物（`img.xz` + `sha512sum` + `manifest` 等）自动发布到 [Releases](../../releases)

## 触发编译

- 推送修改 `.github/workflows/build.yml` 的提交到 `main` 分支时自动触发
- 或在 Actions 页面手动运行 workflow（`workflow_dispatch`），可指定 rsdk ref 与 release tag

## 烧录

```bash
xzcat radxa-rk3576_bookworm_kde_*.output_512.img.xz | sudo dd of=/dev/sdX bs=4M conv=fsync status=progress
```

也可使用 balenaEtcher / Rufus 写入解压后的 `img` 文件。

## SSH

镜像内默认安装 `openssh-server` 并随系统启动。首次开机按提示完成 rsetup 初始配置（RadxaOS 默认账号 `radxa` / `radxa`，以官方文档为准），随后即可连接：

```bash
ssh radxa@<板子IP>
```
