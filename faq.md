# 常见问题（FAQ）

---

## 开发环境

### Q: 编译时报 CS0246 找不到类型或命名空间 UniQQ.SDK？

A: 检查 `.csproj` 中 `UniQQ.SDK` 的引用路径是否正确，确保 `HintPath` 指向实际的 `UniQQ.SDK.dll` 文件。

### Q: 编译时报 CS0012 类型定义缺失？

A: 确保插件 `plugin.json` 中的 `sdkVersion` 与当前使用的 `UniQQ.SDK.dll` 版本一致。

### Q: 支持 .NET Framework 吗？

A: 不支持。UniQQ 插件基于 **.NET 10.0**，项目必须设置 `<TargetFramework>net10.0</TargetFramework>`。

---

## 插件加载

### Q: 插件安装后不显示？

A: 检查压缩包命名是否为 `{id}.uniqq`，其中 `{id}` 必须与 `plugin.json` 中的 `id` 字段完全一致。框架会校验 `manifest.Id + ".uniqq" == 文件名`。

### Q: 插件加载失败提示「压缩包名称错误」？

A: 将 `plugin.json` 的 `id` 作为压缩包文件名，例如 `id` 为 `MyPlugin` 则压缩包命名为 `MyPlugin.uniqq`。

### Q: 插件启用后立刻被禁用？

A: 检查 `OnEnable` 中是否抛出了未捕获的异常。异常会导致启用失败。建议在 `OnEnable` 中包裹 try-catch 并输出日志。

### Q: 修改代码后如何生效？

A: 在 UniQQ 插件管理器中右键插件选择「重载」，或调用 `await Context.ReloadThisPlugin()`。重载会先卸载再重新加载插件。

### Q: 插件出现BUG？

A: 谁写的找谁去反馈。

---

## 事件系统

### Q: 为什么重新启用插件后事件被触发两次？

A: `OnDisable` 中没有取消事件订阅。重新启用时 `OnEnable` 再次订阅，导致同一事件有两个 handler。务必在 `OnDisable` 中调用 `Context.Events.Off<T>()`。

### Q: 一个事件类型能绑定多个 handler 吗？

A: 不能。`On<T>` 会自动覆盖该插件之前对同类型事件的订阅。如需多个处理逻辑，在一个 handler 中调用多个方法。

### Q: 事件 handler 中可以使用同步阻塞吗？

A: 不建议。事件分发是串行的，阻塞会导致后续事件延迟。始终使用 `async Task` 并 `await` 异步操作。

---

## API 调用

### Q: 为什么 SetGroupMemberMuteAsync 返回 false？

A: 常见原因：① 机器人不是管理员/群主；② `memberUin` 是群主（无法禁言群主）；③ `durationSeconds` 超出范围（通常最大 30 天）。

### Q: 为什么 GetGroupMemberListAsync 返回 null？

A: 可能是网络超时或机器人未加入该群。建议检查返回值是否为 null 再做处理。

### Q: Context 在 Load() 中为什么不能用？

A: `Load()` 阶段插件上下文尚未注入。所有需要 `Context` 的逻辑（订阅事件、读取配置）必须放在 `OnEnable()` 及之后。

### Q: 如何获取消息中的图片/语音/视频内容？

A: 使用 `GetImageAsync` / `GetRecordAsync` / `GetVideoAsync` 传入消息段中的 `fileId`，返回详情对象（含下载 URL）。

---

## 数据持久化

### Q: 插件的数据应该存在哪里？

A: 使用 `Context.DataPath`（路径为 `{UniQQ}/Data/plugins/{id}/`）。该目录由框架管理，卸载插件时可一并清理。

### Q: 配置文件用什么格式？

A: 推荐 JSON（`System.Text.Json` 或 `Newtonsoft.Json`）。详见 [数据持久化](/examples/storage)。

---

## 发布

### Q: 插件包必须包含哪些文件？

A: 最少需要 `plugin.json` 和编译输出的 DLL。如有依赖 DLL 也需一并打包。

### Q: 为什么框架不识别我的压缩包？

A: 压缩完成后必须将后缀改为 `.uniqq`。框架通过 `.uniqq` 后缀识别插件包。

### Q: 依赖的第三方 DLL 怎么打包？

A: 将依赖 DLL 与插件主 DLL 放在同一压缩包根目录即可。框架使用 `PluginLoadContext` 按名称解析依赖。

