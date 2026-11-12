# 安装包与清单

[返回首页](../README.md)

## 安装包结构

`.miniapp` 是 ZIP 格式，也接受 `.zip`。根目录必须包含 `miniapp-manifest.json`，兼容旧名称 `app-manifest.json`；新项目统一使用前者。只打包最终静态资源，不执行安装脚本、后台进程或原生模块。

```text
my-miniapp.miniapp
├── miniapp-manifest.json
├── index.html
├── icon.svg
└── assets/
    └── app.js
```

## 包内清单

```json
{
  "manifestVersion": 1,
  "id": "my-miniapp",
  "name": "我的小程序",
  "description": "说明小程序的用途，帮助用户在安装前了解功能。",
  "version": "1.0.0",
  "entry": "index.html",
  "icon": "icon.svg",
  "requiredCapabilities": ["storage:app", "appearance:read"],
  "update": {
    "url": "https://example.com/my-miniapp/manifest.json"
  }
}
```

| 字段 | 必填 | 规则 |
| --- | --- | --- |
| `manifestVersion` | 是 | 固定为数字 `1` |
| `id` | 是 | 1–64 字符，以小写字母开头，只含小写字母、数字和连字符；发布后保持不变 |
| `name` | 是 | 非空字符串，最多 80 字符，不含控制字符 |
| `description` | 否 | 安装确认中展示的简介；去除首尾空白后非空，原始长度最多 500 字符，允许换行和制表符，不含其他控制字符；旧包可省略 |
| `version` | 是 | SemVer，如 `1.0.0`、`1.1.0-beta.1` |
| `entry` | 是 | 包内入口文件的相对路径，文件必须存在 |
| `icon` | 否 | 包内图标文件的相对路径，文件必须存在；推荐不超过 1 MiB 的 PNG、JPG、SVG、WebP 或 ICO |
| `requiredCapabilities` | 是 | 能力字符串数组，可为空，不能重复；仅支持[能力列表](sdk.md#能力列表)中的值 |
| `update.url` | 本地安装可省略；网址安装必填 | 稳定的 HTTPS 发布清单地址，必须与用户输入的安装地址一致 |

`mira-workbench`、`mira-workbench-test`、`mira-harness` 是保留 ID。不能通过 `test: true` 安装测试应用，不能声明 Mira 管理权限。字段类型与格式必须遵循上表。

`package` 字段只放在[在线清单](publishing.md#在线分发清单)，不放进安装包内。

## 路径与大小

| 项目 | 限制 |
| --- | --- |
| 安装包 | 非空，最多 64 MiB |
| 包内清单 / 在线清单 | 每份最多 64 KiB |
| 解压总大小 | 最多 256 MiB |
| 归档条目数 | 最多 10,000，含目录 |
| 包内路径 | 最多 512 字符，相对路径，使用 `/` |
| 发布 URL | 最多 8,192 字符，HTTPS，无凭证或片段 |

拒绝绝对路径、反斜杠、冒号、空路径段、`.`、`..`、控制字符、以点或空格结尾的路径段，以及 Windows 保留设备名。不接受重复路径，包括仅大小写不同的路径。资源路径需要在 Windows 和 macOS 上都可用。

图标路径合法且存在不等于一定显示：Workbench 只缓存受支持格式且不超过 1 MiB 的图标。入口、脚本、样式等用相对引用，确保构建后资源都在包中。

## 安装和更新行为

安装前检查清单与包，用户确认申请能力后写入独立版本目录，再提交注册表。网址安装还会验证来源、大小、摘要和包内声明。

同 ID 只能安装 SemVer 更高的版本；同版本、较低版本以及仅修改 build metadata 的版本都不能覆盖安装。更新重新确认能力，关闭旧视图并保留业务存储；写入或注册表提交失败时保留之前提交的版本。

当前不提供自动检查更新或手动选择历史版本的回滚入口。卸载关闭视图、撤销外部目录句柄并删除安装资源，保留独立业务数据；重新安装不恢复已撤销目录授权。
