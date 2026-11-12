# 发布与更新

[返回首页](../README.md)

开发者负责自己的源码仓库、静态托管、版本和问题反馈。本仓库提供文档与发现入口，不替第三方托管安装包。

## 准备项目仓库

- 使用自己的小程序 ID，保持公开源码可阅读，并说明许可证。
- 添加 `workbench-miniapp` Topic。
- README 说明功能、截图、申请权限、安装地址和问题反馈入口。
- 仓库 Website 填写可安装的 `manifest.json` 地址。

目录可以独立放在任意位置；不需要复制到本仓库或 Mira 源码内。

## 在线分发清单

将稳定的在线清单和按版本划分的包发布到同一 HTTPS origin：

```text
https://example.com/my-miniapp/
├── manifest.json
└── releases/
    └── 1.0.0/
        └── my-miniapp-1.0.0.miniapp
```

在线 `manifest.json` 包含完整[包内清单](packaging.md)字段，再增加 `package`：

```json
{
  "manifestVersion": 1,
  "id": "my-miniapp",
  "name": "我的小程序",
  "version": "1.0.0",
  "entry": "index.html",
  "requiredCapabilities": ["appearance:read"],
  "update": { "url": "https://example.com/my-miniapp/manifest.json" },
  "package": {
    "url": "https://example.com/my-miniapp/releases/1.0.0/my-miniapp-1.0.0.miniapp",
    "sha256": "填写最终安装包的64位十六进制SHA256摘要",
    "size": 12345
  }
}
```

上面的摘要与大小是示意值，必须根据最终文件生成。`package` 只属于在线清单；包内清单也包含相同 `update.url`，不包含 `package`。两份清单支持的字段（ID、名称、版本、入口、图标、权限、更新地址）必须一致。

## 生成摘要与大小

使用 Node.js 根据最终 `.miniapp` 文件生成，避免手工录入：

```js
import { readFileSync, writeFileSync } from 'node:fs'
import { createHash } from 'node:crypto'

const manifest = JSON.parse(readFileSync('dist/miniapp-manifest.json', 'utf8'))
const name = `${manifest.id}-${manifest.version}.miniapp`
const bytes = readFileSync(`release/${name}`)
const distribution = {
  ...manifest,
  package: {
    url: new URL(`releases/${manifest.version}/${name}`, manifest.update.url).href,
    sha256: createHash('sha256').update(bytes).digest('hex'),
    size: bytes.length,
  },
}
writeFileSync('manifest.json', JSON.stringify(distribution, null, 2) + '\n')
```

将生成的 `manifest.json` 与对应包放到上面的静态托管路径。运行此片段前须已构建 `dist/`、生成 `release/` 下的包，并配置有效 `update.url`。

## 托管要求

- 清单地址、`update.url` 和 `package.url` 都是 HTTPS 绝对地址，不含账号密码或 URL 片段。
- 规范化后的 `update.url` 必须与用户粘贴的安装地址一致；不支持镜像地址替代或重定向。
- 包和清单必须同 origin，即协议、域名与端口相同，不只是同属一个组织。
- 文件直接返回 HTTP 200，不跳转到登录、预览页或 CDN 地址。GitHub 仓库的文件网页不是安装地址；GitHub Release 附件通常重定向，不可直接用作 `package.url`。
- 清单最多 64 KiB，包最多 64 MiB；`size` 是原始包字节数，`sha256` 为最终包的 64 位十六进制 SHA-256。Mira 下载超时分别为 15 秒和 60 秒，并拒绝已识别的本机/内网地址。
- 旧版本包使用不可变路径。不要覆盖已发布的相同版本；新发布时增加 SemVer，更新稳定的清单地址，保留旧包。

## 使用 GitHub Pages

[官方示例](https://github.com/MiraWorkspace/workbench-sdk-playground)提供可复用的[构建脚本](https://github.com/MiraWorkspace/workbench-sdk-playground/blob/main/scripts/build.mjs)与[工作流](https://github.com/MiraWorkspace/workbench-sdk-playground/blob/main/.github/workflows/pages.yml)。

1. 改成自己的 ID、名称、能力列表和 `update.url`，如 `https://your-account.github.io/your-repository/manifest.json`，同步项目和清单版本。
2. `npm ci && npm run build` 生成 `site/manifest.json`、`site/releases/<version>/<id>-<version>.miniapp`。
3. 仓库 Settings → Pages → Source 选择 **GitHub Actions**，推送主分支并等待发布成功。首次未启用 Pages 导致工作流失败时，启用后重新运行即可。
4. 验证线上清单与包直接返回成功，大小和摘要正确；在新版 Mira / Workbench 中粘贴清单地址，确认来源与权限，安装后打开。

示例工作流把版本包保留在 `gh-pages` 分支。已有版本字节发生变化时发布失败，避免同版本被悄悄覆盖。工作流需要仓库内容写入、Pages 写入与部署的 ID token 权限；只复用你理解的工作流，并按自己的仓库调整。

## 更新与验证

当前更新由用户再次粘贴清单地址或选择新包手动发起，不自动检查更新。`update.url` 是稳定的分发入口，不表示已有后台自动升级服务。

同 ID 仅接受严格更高的 SemVer。正式发布更新前才增加版本号；本地开发每次改功能不要求增加 SDK 或小程序版本。不要移动已发布标签、替换旧包或仅改 build metadata 试图覆盖安装。

更新展示版本、来源和所需权限，确认后安装；失败保留之前提交的版本，独立业务存储保留。更新会重新确认清单全部能力，不应假设旧权限选择自动继承。

发布前至少验证：

1. 在干净环境安装，取消确认不留下已安装记录，错误后可以重试。
2. 从上一公开版本升级，核对业务数据保留、视图关闭与新版本打开。
3. 按实际权限测试拒绝行为、目录只读限制与撤销后访问。
4. 核对线上摘要、大小、包内清单和资源路径；模型和系统通知在目标桌面平台单独验收。

收录到社区列表按[贡献指南](../CONTRIBUTING.md)提交。安装或运行报错先看[常见问题](troubleshooting.md)。
