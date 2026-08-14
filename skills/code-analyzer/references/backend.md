# Backend 域合同

仅在 `route-plan.json` 将 `backend` 标为 `relevant` 时读取本文件。

## 确认证据

从注册和触发关系识别服务端能力：

- HTTP/RPC/GraphQL route 与 handler/controller 的注册；
- message/event consumer 与 topic/handler 的绑定；
- worker、scheduled job 或 task queue 的注册；
- CLI command 的入口注册；
- server bootstrap 与上述入口的装配关系。

类名含 `Service`、存在服务端依赖或开放端口配置只能作为线索，不能单独证明请求路径。

## 划分单元

按入站 route、事件、job 或 command 代表的业务流建单元。共享 middleware 和 domain service 由首次使用它的主单元提取，其他单元引用证据台账。

每个单元只追踪实际存在的层：

1. trigger 与注册位置；
2. middleware、鉴权、校验或反序列化；
3. handler 到 domain/service 逻辑；
4. persistence、queue 或 outbound integration；
5. response、ack、retry 或 failure handling。

## 追踪链路

按入口类型选用实际路径：

```text
HTTP/RPC route → middleware → handler → service/domain → persistence/integration → response
event/topic → consumer → service/domain → persistence/outbound event → ack/retry
schedule/queue → worker/job → service/domain → persistence/integration → completion/failure
CLI registration → command handler → service/domain → output/exit state
```

不要求每条链包含每一层。只找到客户端调用却找不到入站注册时，把服务端边界标为 `unavailable`。

## 输出重点

在统一解释结构中，重点说明 trigger 类型及注册，输入解析、校验、鉴权和 middleware 顺序，核心业务分支与状态变化，持久化/cache/queue/外部集成，以及 response/ack、错误映射、retry 或 fallback。

仓库没有 UI 时不要生成 page、card、组件或移动端单元。不要从源码结构推断该入口已在线上暴露或具备生产流量。
