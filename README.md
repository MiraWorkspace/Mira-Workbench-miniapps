# Mira Workbench Miniapps

Workbench 小程序开发文档与社区发现入口。小程序运行在 Mira 中的 Workbench，通过 Workbench 授权的 SDK 通道调用模型、联网搜索、存储、文件、主题和通知能力。

开发者在自己的仓库维护源码和安装包，本仓库维护开发指南、分发规范及小程序列表。

## 发现小程序

- [浏览全部小程序](https://github.com/topics/workbench-miniapp)
- [最近更新](https://github.com/search?q=topic%3Aworkbench-miniapp+archived%3Afalse&type=repositories&s=updated&o=desc)
- [最多 Star](https://github.com/search?q=topic%3Aworkbench-miniapp+archived%3Afalse&type=repositories&s=stars&o=desc)

## 开发文档

- [快速开始](docs/getting-started.md)：从示例入手，构建并在本地安装。
- [安装包与清单](docs/packaging.md)：`.miniapp` 格式、字段和限制。
- [SDK 与权限](docs/sdk.md)：连接、AI、存储、文件及能力边界。
- [发布与更新](docs/publishing.md)：HTTPS 分发清单、GitHub Pages 和版本管理。
- [常见问题](docs/troubleshooting.md)：安装、连接、权限和发布错误。
- [贡献指南](CONTRIBUTING.md)：提交小程序、修正文档。

底层客户端完整类型与参数见 [Mira-App-SDK](https://github.com/MiraWorkspace/Mira-App-SDK)。SDK 包含 Mira 子应用接口；小程序可用范围以本仓库的 [SDK 与权限](docs/sdk.md) 为准。

## 小程序列表

在 Mira 中打开 Workbench，选择“添加 → 小程序 → 从网址安装”，粘贴安装清单地址，检查来源和权限后确认安装。

### 官方示例

[Workbench SDK 能力演示台](https://github.com/MiraWorkspace/workbench-sdk-playground) — 面向开发者的 28 个可执行示例，展示参数、实际 SDK 调用代码、返回结果和错误，覆盖模型对话、联网、SQLite、文件、主题、通知及权限处理。

安装地址：

```text
https://miraworkspace.github.io/workbench-sdk-playground/manifest.json
```

网址安装需要支持该功能的 Workbench 与 Mira API 1.4。也可从示例仓库下载 `.miniapp` 后选择本地文件安装。完整兼容关系见[快速开始](docs/getting-started.md#运行要求)。

### 社区小程序

欢迎按[贡献指南](CONTRIBUTING.md)提交你的项目。GitHub Topic 搜索结果由开发者自行标注，不代表官方审核或推荐；安装前请核对发布者、源码与权限声明。

## 发布你的小程序

1. 在自己的公开仓库维护源码，添加 `workbench-miniapp` Topic。
2. 按[发布规范](docs/publishing.md)在同一 HTTPS 来源托管清单和版本包。
3. 在仓库 Website 填写可安装的 `manifest.json` 地址，并在 README 说明功能、权限、安装和反馈方式。
4. 如需收录到上面的列表，向本仓库提交 Pull Request。

## 相关项目

- [Mira-Harness](https://github.com/MiraWorkspace/Mira-Harness)：Mira 主应用，管理供应商与原生能力。
- [Mira-Workbench](https://github.com/MiraWorkspace/Mira-Workbench)：可选子应用，也是小程序宿主。
- [Mira-App-SDK](https://github.com/MiraWorkspace/Mira-App-SDK)：消息客户端与类型定义。
- [workbench-sdk-playground](https://github.com/MiraWorkspace/workbench-sdk-playground)：可运行的官方示例。
