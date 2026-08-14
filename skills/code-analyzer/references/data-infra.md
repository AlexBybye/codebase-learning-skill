# Data / Infra 域合同

仅在 `route-plan.json` 将 `data_infra` 标为 `relevant` 时读取本文件。该域 relevant 不表示其所有子能力都存在；逐项记录 `present`、`skipped` 或 `unavailable`。

## 证据类别

只分析仓库实际存在的类别：

- persistence：schema、migration、entity/model、DAO/query、transaction；
- cache：缓存持有者、key、TTL、失效和回源逻辑；
- messaging：producer/consumer、queue/topic、payload、ack/retry；
- build/deploy：CI 流水线、容器、基础设施即代码、环境装配和部署声明；
- observability：日志、metric、trace、告警配置和关联字段。

仅有环境变量名、普通应用编译文件或依赖声明时，把它作为线索，不能据此让本域 relevant；需要数据能力或实际 CI/部署/观测配置与代码才能形成能力结论。

## 划分单元

优先把 data/infra 证据附着到使用它的 web、mobile、backend 或 AI 业务流。只有共享 schema、migration 序列、消息拓扑或部署/观测入口横跨多个业务流时才建立独立单元。

### Persistence

追踪 `caller → repository/DAO/query → entity/table/collection → schema/migration`。记录查询条件、写入字段、transaction 边界和返回映射。只从真实定义记录表名与关系；不要根据 DTO 名猜 schema。

### Cache 与 messaging

追踪 cache 的读、miss、回源、写与失效；追踪 message 的 producer、payload、topic/queue、consumer、ack/retry。若 broker、consumer 或 key 定义位于仓库外，明确标为 `unavailable`。

### Build/deploy 与 observability

记录配置表达的构建阶段、部署对象、环境输入、health check，以及代码写入的 log/metric/trace。把它们描述为“仓库配置支持/声明”，不得改写为“当前线上已部署/已监控”。

## 输出重点

在统一解释结构中，为每个存在的子能力说明入口消费者和配置/源码锚点、数据形态及变换、生命周期或迁移顺序、失败/retry/fallback/一致性边界，以及与相关业务域复用的跨域边。

对完全不存在的子能力标为 `skipped`；对调用了仓库外设施但当前边界看不到实现的环节标为 `unavailable`。不要推断数据库实际数据、队列积压、部署健康度或观测覆盖率。
