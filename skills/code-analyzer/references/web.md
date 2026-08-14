# Web 域合同

仅在 `route-plan.json` 将 `web` 标为 `relevant` 时读取本文件。

## 确认证据

从实际注册关系确认浏览器界面，而不是仅凭 JavaScript/TypeScript 扩展名或前端依赖：

- 浏览器入口挂载、路由表、页面目录或服务端渲染页面入口；
- 能追到入口的可渲染组件、模板或样式；
- 页面消费的状态容器、loader/action 或请求边界。

若只存在共享类型、构建工具或服务端 JavaScript，不得因此建立 Web 单元。

## 划分单元

按用户可识别的页面或交互流划分，不要把每个叶子组件变成单元。每个单元从已注册入口开始，限制在以下必要路径：

1. 路由或页面入口；
2. 直接组成页面的关键组件；
3. 驱动显示和交互的状态；
4. 数据加载或请求边界；
5. 导航、加载、空态和错误处理。

将纯展示叶子组件合并进所属页面。只在一个共享组件拥有独立业务状态或被多个入口以不同流程使用时单列。

## 追踪链路

优先还原以下证据链：

```text
route/page entry
  → rendered component
  → user action or lifecycle load
  → state transition
  → request/server action
  → response mapping
  → render or navigation result
```

客户端请求只能追到源码可见的 URL、请求函数或 server action；若服务端实现不在仓库内，将下一跳标为 `unavailable`。若相邻 `backend` 域 relevant，则复用其 handler 证据，不重新扫描服务端目录。

## 输出重点

在统一解释结构中，重点说明入口 URL/route 与注册、页面和关键组件层级、状态的写入与 UI 消费、用户操作或加载到导航/失败/空态的过程，以及到 backend 或外部服务的已证明边。

不猜测像素或 design token，不生成移动端生命周期、卡片模板或未在源码出现的设计系统名称。
