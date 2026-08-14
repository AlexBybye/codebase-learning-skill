# Mobile 域合同

仅在 `route-plan.json` 将 `mobile` 标为 `relevant` 时读取本文件。保留页面/组件、状态、导航和客户端出站调用四项核心能力。

## 确认证据

从项目清单、构建目标和 UI 注册关系识别实际移动技术，不预设 Android、iOS 或跨平台方案：

- Android 可由应用 Manifest/launcher、Activity/Fragment/Compose 入口和导航资源互证；
- iOS 可由 app target、`@main`/AppDelegate/SceneDelegate、UIViewController/SwiftUI 入口互证；
- 跨平台客户端需由其移动构建目标与实际页面入口互证；共享语言或依赖本身不够。

只有移动共享库而没有应用入口时，记录为候选证据，不创建虚构页面。

## 划分单元

按用户页面或完整交互流建单元。每个单元限定读取：

1. 页面/组件入口及布局或声明式视图；
2. 页面持有或观察的状态容器；
3. 真实导航注册、跳转和参数；
4. 触发的数据访问或客户端出站调用；
5. 直接参与该流的 mapper/repository/use-case（若存在）。

Adapter、Cell、ViewHolder 或复用组件通常归入所属页面；仅在拥有独立业务流程且证据充分时单列。不要强制仓库必须有 ViewModel、UseCase 或 Repository 层。

## 追踪链路

从真实生命周期或用户事件开始，按仓库实际层级追踪：

```text
app/navigation entry
  → page or component
  → lifecycle/user event
  → state write
  → optional domain/data layers
  → outbound client or local store
  → state consumption and UI response
```

层不存在时直接连接实际调用方和被调方；实现位于仓库外时标为 `unavailable`，不能套用固定分层。若请求进入同仓库 backend，引用 backend 单元登记的 handler。

## 输出重点

在统一解释结构中，重点说明页面/组件入口与布局层级，状态定义、写入条件和 UI 消费，进出导航及参数，用户操作或生命周期加载链，以及客户端请求、本地存储、缓存和错误/空态。

颜色、尺寸、图标或 design token 只记录源码可证明的内容；不从命名猜测 UI，不补造夜间样式，不把项目专有网络客户端或设计系统当默认值。
