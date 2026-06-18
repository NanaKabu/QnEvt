# QnEvt

![Platform](https://img.shields.io/badge/Platform-Windows-blue)
![Network](https://img.shields.io/badge/Network-SOCKS5-24d6a0)
![Distribution](https://img.shields.io/badge/Source-Closed-lightgrey)

QnEvt 是一款 Windows 指定应用 SOCKS5 网络通道工具。它面向开发、测试和网络诊断场景，帮助你把选中的本机应用接入独立 SOCKS5 通道，其它软件继续使用原有网络环境。

官网：[https://qnevt.com](https://qnevt.com)

下载：[GitHub Releases](https://github.com/NanaKabu/QnEvt/releases)

## 适合谁

- 你只想让某个应用走代理，不想改系统全局代理。
- 你遇到全局代理导致个别应用卡顿、延迟升高或网络异常。
- 你需要对 IDE、命令行工具、开发客户端或特定业务软件做独立网络调试。
- 你在自有或已获授权的设备、应用和网络环境中做连接诊断。

## 核心能力

- 指定应用代理：只对选中的应用生效，减少对其它软件的影响。
- SOCKS5 专用通道：当前版本专注 SOCKS5，支持地址、端口和可选账号密码。
- 多进程应用配置：一个应用配置可以关联多个相关 `.exe`，例如 IDE 主程序、扩展进程和语言服务进程。
- 连接诊断：提供启动状态、代理可信度、连接验证和运行日志。
- 免安装包形态：下载 ZIP 后解压，按说明启动即可使用。

## 系统要求

- Windows 10/11 x64
- 管理员权限
- 合法、可信的 SOCKS5 服务
- 正式分发版本应使用已认证签名的驱动

## 快速使用

1. 从官网或 GitHub Releases 下载最新 `QnEvt-<version>-x64.zip`。
2. 解压到一个固定目录，例如 `D:\Apps\QnEvt`。
3. 以管理员身份运行 `QnEvt.exe`。
4. 在“SOCKS5 配置”中填写服务器地址、端口和认证信息。
5. 在“应用管理”中启用需要走独立通道的应用。
6. 点击“启动网络调试”，再启动目标应用进行验证。

配置文件位于程序同级的 `QnEvt-V2-assets` 目录。应用图标以 `QnEvt-V2-assets\app` 文件夹为准，应用配置以 `QnEvt-V2-assets\config` 文件夹中的 JSON 为准。

## 配置示例

```json
{
  "Name": "Antigravity IDE",
  "ExecutableName": "Antigravity IDE.exe,language_server_windows_x64.exe,node.exe,antigravity.exe,msedgewebview2.exe",
  "ExecutablePath": null,
  "IsSelected": true,
  "IconPath": "",
  "BypassRules": []
}
```

`ExecutableName` 可以只写一个进程名，例如 `"codex.exe"`；也可以用英文逗号写多个相关进程。应用卡片的“运行中”状态以第一个进程名为准。

## 发布与更新

当前推荐发布版本：`3.0.0`

服务器后台发布时上传 ZIP 包，版本号填写 `3.0.0`。官网后台会自动生成公开下载文件和 `update.json`，桌面端会通过 HTTPS 检查更新。

如果官网开启“研发中”模式，`update.json` 会返回 `available:false`，桌面端会显示暂无可下载版本，而不是更新失败。

## 合规说明

QnEvt 仅适用于你本人所有或已获得合法授权管理的设备、应用、服务和网络环境。请遵守适用法律法规、网络服务协议和组织安全制度。不得将本软件用于侵入、干扰未经授权的系统或网络，也不得用于任何违法违规活动。

## Source Availability

QnEvt is currently distributed as a closed-source Windows application. This repository is used for product information, release notes, downloads, and update metadata only. The source code is not published.

## English Summary

QnEvt is a Windows per-application SOCKS5 network channel tool. It helps selected local applications use an independent SOCKS5 route while the rest of the system keeps its normal network environment.

Use it only on devices, applications, services, and networks that you own or are authorized to manage.
