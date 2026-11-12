# SDK 与权限

[返回首页](../README.md)

## 调用关系

```text
小程序中的 SDK 消息客户端
  → Workbench 校验小程序身份与用户授权
  → Workbench 通过 Mira-App-SDK 调用 Mira
  → Mira 执行能力并返回结果
```

Mira 与 Harness 是同一个主应用；Workbench 是可选子应用，也是这些小程序的宿主。小程序不注册为 Mira 子应用，不能直接获取 Mira 身份或供应商密钥。

小程序可以打包 `@mira/app-sdk` 客户端，连接对端仍为 Workbench。SDK 提供的全部 Mira 接口不会因此全部向小程序开放。

## 连接与释放

```js
import { connectMiraApp } from '@mira/app-sdk'
const sdk = await connectMiraApp()
window.addEventListener('pagehide', () => sdk.dispose(), { once: true })
const context = await sdk.context.get()
// { appId, host: 'workbench', apiVersion: { major: 1, minor: 2 }, capabilities }
```

`capabilities` 是实际已授权能力，不等于清单申请列表。`context.get()` 无额外权限要求。客户端 `dispose()` 释放真实连接后 Workbench 会关闭小程序视图，需从 Workbench 重新打开；页面卸载时也应清理订阅与连接。

## 能力列表

| 清单声明 | 小程序可调用方法 |
| --- | --- |
| `appearance:read` | `appearance.get/subscribe` |
| `models:generate` | `models.listProviders/listModels/generate/chat` |
| `web:search` | `web.search`；与模型权限共同允许 `models.chat({ webSearch: true })` |
| `storage:app` | `storage.migrate/transaction` |
| `files:read` | `files.read/list/directories` |
| `files:write` | `files.write` |
| `notifications:send` | `notifications.send` |

每次调用都会检查授权。权限调整或目录撤销会关闭运行视图；迟到的模型结果不会继续交给已撤权小程序，但已经发送给供应商的请求可能继续执行。

以下 Mira 客户端接口不向小程序开放：`views.*`、`workbench.*`、旧数据迁移、`files.selectDirectory/remove/revoke`，以及任选身份或 namespace。包括 `workbench.fetchMiniappResource` 在内的安装运输接口仅供 Workbench 调用。

## 供应商、模型和对话

```js
const providers = await sdk.models.listProviders()
// 由界面让用户选择 providerId。
const models = await sdk.models.listModels(providerId)
// 由界面让用户选择 modelId。
const messages = [{ role: 'user', content: '我叫小明。' }]
const first = await sdk.models.chat({ providerId, modelId, messages })
messages.push(
  { role: 'assistant', content: first.text },
  { role: 'user', content: '我叫什么？' },
)
const second = await sdk.models.chat({ providerId, modelId, messages })
```

供应商和模型列表只返回已配置可用、已启用聊天项的 `{ id, name }`，不返回密钥或 API 地址，也不发起生成请求。`chat` 必须显式选择供应商和模型，界面需要处理空列表。

每次提交完整历史，由小程序维护。消息只支持非空文本的 `user/assistant`，首尾必须是用户消息；可另传 `system`。最多 100 条消息，含系统提示的文本合计最多 100,000 字符。返回 `{ text }`，目前无流式、附件和自定义工具注册。

单次生成也可使用 `sdk.models.generate({ prompt, providerId, modelId })`。两个 ID 一起省略时使用 Mira 默认模型；指定时必须同时提供。模型调用可能产生费用，应由用户操作触发。

## AI 自主联网与独立搜索

```js
const answer = await sdk.models.chat({
  providerId, modelId,
  webSearch: true,
  messages: [{ role: 'user', content: '查阅 TypeScript 官方文档后解释类型收窄。' }],
})
// answer.text 为最终回答；answer.sources 的 index 对应 [[source:N]]。
```

需要同时申请并获准 `models:generate` 与 `web:search`，并选择支持工具调用的模型。AI 决定查询词、搜索、阅读和是否继续，宿主自动选择搜索源、并发回退，普通界面不用选择搜索引擎。允许联网不表示每个问题都会搜索。

每批最多 3 个搜索/读取任务并行，一次对话最多 6 次模型调用、8 次工具执行，总预算 60 秒，SDK 等待至少 75 秒。来源编号按请求生成，需绑定到相应回答；发现来源不表示已读过全部正文。网页与摘要是外部资料，不能当作指令执行。

只需要结构化搜索结果时，单独使用：

```js
const result = await sdk.web.search({ query: 'TypeScript 官方文档', limit: 5 })
// result.results: [{ title, url, snippet? }]
```

独立搜索只需要 `web:search`，不需要模型配置。query 最长 2,000 字符，limit 为 1–10 的整数、默认 5。返回还含 `query`、`engine`、`attempts`，单次预算 25 秒、SDK 等待至少 35 秒。可选 `engine` 供诊断使用，不是 AI 联网必填参数。

## SQLite 存储

```js
await sdk.storage.migrate([{ version: 1, tables: [{
  name: 'items', columns: [
    { name: 'id', type: 'TEXT', primaryKey: true },
    { name: 'text', type: 'TEXT' },
  ],
}] }])
await sdk.storage.transaction([{ type: 'insert', table: 'items', values: { id: '1', text: '你好' } }])
const [rows] = await sdk.storage.transaction([{ type: 'select', table: 'items', limit: 20 }])
```

namespace 由 Workbench 固定为当前小程序，不接受自行切换。迁移版本递增、同版本不重复应用；事务提供声明式增删改查，批次失败整体回滚，不是任意 SQL 接口。完整操作类型见 SDK 和[可运行示例](https://github.com/MiraWorkspace/workbench-sdk-playground/tree/main/src/examples)。

## 私有文件与外部目录

```js
await sdk.files.write({ path: 'documents/hello.txt', data: new TextEncoder().encode('你好') })
const bytes = await sdk.files.read({ path: 'documents/hello.txt' })
const text = new TextDecoder().decode(bytes)
const entries = await sdk.files.list({ path: '' })
```

无 `grant` 时访问自己的私有文件区，单文件最多 64 MiB。路径均为相对路径，写入会覆盖同路径文件；`read` 返回 `Uint8Array`。仅 `list` 可使用空路径表示根目录。

外部目录必须由用户在 Workbench“设置 → 小程序权限”添加只读或读写目录，再从 `sdk.files.directories()` 获取句柄。调用时传 `{ path: 'report.txt', grant: grantId }`；不能传绝对路径。文件能力声明不授予全盘访问权，只读目录拒绝写入，撤销后的旧句柄拒绝访问。

## 主题、通知与错误

```js
const theme = await sdk.appearance.get()
const unsubscribe = sdk.appearance.subscribe(value => {
  document.documentElement.dataset.theme = value
})
// 页面卸载前 unsubscribe()，然后 sdk.dispose()。
await sdk.notifications.send({ title: '处理完成' })
```

通知成功表示宿主接受请求，不等于操作系统已显示横幅；macOS 需要正式安装版及系统授权，Windows 和 macOS 分别验收。

Workbench 拒绝请求时通常返回 `APPLICATION_DENIED`，其 `message` 包含可展示原因；本地客户端已释放时为 `DISCONNECTED`。界面应显示实际错误并允许重试，不用假数据替代结果。更多参数、错误类型见 [SDK 文档](https://github.com/MiraWorkspace/Mira-App-SDK/blob/main/doc/api.md)；完整示例见[官方演示台](https://github.com/MiraWorkspace/workbench-sdk-playground)。
