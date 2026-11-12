# 常见问题

[返回首页](../README.md)

| 现象 | 排查方法 |
| --- | --- |
| 提示需要 Mira API 1.4 | 更新并重启 Mira，加载支持网址安装的 Workbench；只替换小程序中的 SDK 文件无法升级 Mira 服务 |
| 安装地址不是有效 JSON | 使用 `manifest.json` 的 HTTPS 直接地址，不能填仓库网页、HTML 预览或登录地址 |
| 提示来源不能重定向 | 检查 HTTP 响应；GitHub Release 附件、跳转短链、未补齐路径的地址不适用，改用直接返回文件的静态托管 |
| 提示同源或 `update.url` 不一致 | 核对输入地址与 `update.url`，以及包地址的协议、域名和端口；使用同一个公开入口 |
| 大小或 SHA-256 不符 | 根据最终包重新生成在线清单，排查上传或缓存问题；已发布版本字节变化应发布新版本，不能覆盖旧包 |
| 包内声明不一致 | 在线清单需由包内相同清单生成，仅增加 `package`；核对名称、ID、版本、入口、图标、权限、`update.url` |
| 入口或图标不存在 | 归档根目录直接放清单和静态资源，不再嵌套 `dist/` 或项目目录；核对大小写和相对引用 |
| 更新版本必须高于已安装版本 | 相同或更低 SemVer 不能覆盖；更新时使用更高版本。同版本本地复验可卸载后再安装，卸载保留业务数据并撤销外部目录授权 |
| 普通浏览器显示未连接 | 浏览器预览只能阅读页面；SDK 能力必须在 Workbench 安装并打开后使用 |
| 模型列表为空 | 在 Mira 中配置并启用聊天供应商与模型，并检查小程序获准 `models:generate` |
| 联网对话失败或没有搜索 | 同时需要模型和搜索权限；选择支持工具调用的模型。`webSearch: true` 允许 AI 自主搜索，不强制每个问题都搜索 |
| `APPLICATION_DENIED` 或方法不支持 | 查看实际 message 和 `context.get().capabilities`；清单声明、Workbench 用户授权与开放方法必须同时满足 |
| 目录列表为空或写入被拒绝 | 在 Workbench 权限设置中添加目录；不能从小程序弹出授权器。使用有效 grant，确认文件能力和目录读写标记 |
| 通知返回成功但没有横幅 | 只证明宿主接受请求。检查正式安装版、操作系统通知授权与免打扰设置；macOS/Windows 分别验证 |
| 页面关闭或 `DISCONNECTED` | 检查是否释放了真实连接、用户关闭视图或撤销权限；从 Workbench 重新打开，旧客户端不可继续使用 |

## 反馈问题

小程序业务问题在对应小程序仓库反馈；文档错误向[本仓库](https://github.com/MiraWorkspace/Mira-Workbench-miniapps/issues)反馈。可复现的安装器或 SDK 问题分别提交到 [Workbench](https://github.com/MiraWorkspace/Mira-Workbench/issues) 或 [Mira-App-SDK](https://github.com/MiraWorkspace/Mira-App-SDK/issues)。

请附 Mira 与 Workbench 版本、操作系统、安装方式、公开清单地址、实际错误文字和复现步骤。不要粘贴供应商密钥、私人文件或包含凭证的地址。区分“构建通过”“隔离 Electron 检查通过”与“正式安装版实际能力验收”，避免把夹具结果当成真实模型或系统行为。
