# 快速开始

[返回首页](../README.md)

小程序是 HTML、CSS、JavaScript 等静态资源组成的 ZIP 包，扩展名为 `.miniapp`。包内网页通过消息通道连接 Workbench，Workbench 根据用户授权调用 Mira 能力。安装包不包含模型服务、供应商配置或密钥。

## 运行要求

| 场景 | 要求 |
| --- | --- |
| 小程序运行 | Mira 桌面应用中启用 Workbench，支持小程序 API 1.2 |
| 模型选择、多轮对话、联网 | Mira API 1.3 或更新版本 |
| 从网址安装 | 支持网址安装的 Workbench（使用 SDK 0.4.0），Mira API 1.4 或更新版本 |
| 本地构建官方示例 | Node.js 22+、npm、Git，可访问 GitHub 和依赖仓库 |

当前公开 SDK 通过 GitHub 标签分发：`github:MiraWorkspace/Mira-App-SDK#v0.4.0`。官方示例的小程序端仍使用兼容的 `v0.3.0`；新版客户端不会自动增加 Workbench 向小程序开放的方法。

## 先运行官方示例

可直接使用[首页的安装地址](../README.md#官方示例)。需要查看或修改代码时，克隆独立项目：

```sh
git clone https://github.com/MiraWorkspace/workbench-sdk-playground.git
cd workbench-sdk-playground
npm ci
npm run build
```

构建生成 `dist/` 静态资源、`release/` 中的 `.miniapp` 和摘要文件，以及用于线上分发的 `site/`。项目可放在任意目录，不要求嵌套在 Mira 源码或 `test` 目录中。

1. 在 Workbench 点击“添加 → 小程序 → 选择文件”。
2. 选择 `release/` 中的 `.miniapp`，检查能力申请并确认安装。
3. 打开演示台，确认显示“已连接 Workbench”。
4. 先执行上下文和存储示例，再按需运行模型、联网、文件等示例。

演示台不会在打开时自动调用模型或搜索。AI 示例需要先在 Mira 中配置并启用聊天供应商与模型。

## 开发自己的小程序

从示例中选取需要的调用，也可以从空的静态网页开始。修改为自己的 `id`、名称、版本、入口及实际需要的能力，不要沿用示例的全部权限。

```text
my-miniapp/
├── miniapp-manifest.json
├── index.html
├── app.js
└── vendor/
    └── mira-app-sdk.js
```

安装客户端：

```sh
npm install github:MiraWorkspace/Mira-App-SDK#v0.4.0
```

如果使用 Vite 等构建工具，可直接从 `@mira/app-sdk` 导入并打包；纯静态网页可参考示例的[构建脚本](https://github.com/MiraWorkspace/workbench-sdk-playground/blob/main/scripts/build.mjs)，将依赖中构建好的 `dist/index.js` 和许可证复制到运行产物的 `vendor/`。不要手工维护 SDK 副本。

最小清单：

```json
{
  "manifestVersion": 1,
  "id": "my-miniapp",
  "name": "我的小程序",
  "version": "0.1.0",
  "entry": "index.html",
  "requiredCapabilities": ["appearance:read"]
}
```

`index.html`：

```html
<!doctype html>
<html lang="zh-CN">
<meta charset="utf-8">
<title>我的小程序</title>
<p id="status">正在连接 Workbench…</p>
<script type="module" src="./app.js"></script>
</html>
```

`app.js`（纯静态文件示例）：

```js
import { connectMiraApp } from './vendor/mira-app-sdk.js'

const status = document.querySelector('#status')
try {
  const sdk = await connectMiraApp()
  window.addEventListener('pagehide', () => sdk.dispose(), { once: true })
  const context = await sdk.context.get()
  if (context.host !== 'workbench') throw new Error('请在 Workbench 中打开')
  const theme = await sdk.appearance.get()
  document.documentElement.dataset.theme = theme
  status.textContent = `已连接：${context.appId}，主题：${theme}`
} catch (error) {
  status.textContent = error instanceof Error ? error.message : '连接失败'
}
```

SDK 是消息客户端；服务在 Mira 执行，小程序请求先由 Workbench 授权。普通浏览器没有此通道，不能以浏览器预览结果代替安装验证。

## 打包与下一步

把最终静态资源与清单放在归档根目录，不要把整个项目目录再包一层。可使用 ZIP 工具，或复用官方示例的 `fflate` 构建脚本，生成 `.miniapp`。不要打入 `node_modules`、源码测试文件、密钥或开发依赖。

- 先按[安装包与清单](packaging.md)检查产物，再本地安装。
- 新增能力前阅读[SDK 与权限](sdk.md)。
- 本地跑通后按[发布与更新](publishing.md)提供线上安装地址。
