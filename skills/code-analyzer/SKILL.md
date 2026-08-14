---
name: code-analyzer
description: Learn an unfamiliar repository from current source and configuration evidence and write a bounded repository learning map under doc/analysis. Use only for systematic repository-level understanding of web, mobile, backend, data/infra, AI, or mixed codebases, including cross-domain request and data flows and resumable analysis. Do not use for a focused file explanation, patch or code review, bug diagnosis or fix, architecture-diagram-only request, security or license audit, or general source-code change.
---

# Code Analyzer

从证据发现开始，先决定分析域，再只深入相关源码。把移动端作为五个平等域之一，不预设仓库技术栈。

## 硬边界

1. 将用户给出的路径解析为绝对 `target_root`。用户明确说“当前仓库”时，使用当前工作目录；路径仍不明确时再询问。
2. 只读取目标仓库。只在解析并校验后的 `<target_root>/doc/analysis/` 内写文件，拒绝路径穿越或解析到该目录外的链接；不得修改、格式化或生成业务源码。
3. 默认不运行目标应用、测试、构建或迁移，不安装依赖，不读取凭据，不联网或调用外部服务。只使用文件列举、搜索、读取和分析产物写入能力。
4. 不从静态结构声称线上部署状态、运行性能、模型质量、安全性或生产就绪。把这些结论标为 `unavailable`，除非用户另行提供相应运行证据和授权。
5. 仅在源码或配置实际出现时记录项目专有的品牌、客户端、协议、设计系统、模型或基础设施名称；绝不把示例名称填入结果。
6. 把用户指定的输出路径和精确文件结构视为输出合同。若用户要求某 JSON “严格为”给定对象，逐键逐值写入，禁止追加状态、证据、时间或说明字段。

## 统一证据合同

对路由理由、单元结论、调用链边和缺失环节等每条关键事实，同时写出 `source` 与 `evidence`：

- `source: code`：可由当前业务源码中的具体符号直接证明。
- `source: config`：可由当前仓库管理的清单、schema、migration、构建或部署配置中的具体键直接证明。
- `source: inferred`：由至少两个独立源码或配置锚点组合推断；同时说明推断步骤，不能冒充直接事实。
- `source: unavailable`：当前边界内缺少证明；在 `evidence` 写明已经检查的范围和缺少的连接点。

将 `evidence` 写成相对 `target_root` 的 ``path/to/file#symbol``、``path/to/file#key`` 或 ``path/to/file:Lx-Ly``。优先引用符号；禁止用绝对路径、仅文件名、旧分析文档或仓库说明文字替代实现证据。续跑时可以把既有分析产物当缓存，但最终关键结论仍须保留其源码或配置锚点。

## 统一解释合同

取证后、写入学习地图前做一次忠实解释，不二次分析：

1. 每个单元按“作用 → 入口 → 流转 → 结果与边界 → 证据”组织；先用一句话说明作用，再展开真实符号和调用链。首次术语保留原名并简释。
2. 标识符、路径、配置键、代码、精确 JSON、状态/锚点，以及事实、数字、条件、范围、不确定性和 `source/evidence` 不变；不得把 `inferred` 或 `unavailable` 写成事实。
3. 类比须标注且不能充当证据。写入前对照证据台账和域合同复核；表达自然与技术精度冲突时，以精度为准。

## 执行流程

### 1. 固定输出合同并读取续跑状态

先记录用户要求的学习地图路径、`route-plan.json` 路径及任何精确 JSON。未指定地图文件名时使用 `doc/analysis/repository-learning-map.md`；未指定路由文件名时使用 `doc/analysis/route-plan.json`。

若 `doc/analysis/pipeline_state.json` 或目标学习地图已存在：

1. 把已有 `status: completed` 单元视为不可重跑的恢复对象；续跑时不得为它读取业务源码或重新执行域分析。
2. 若 completed 状态已记录 artifact、section anchors 和 evidence anchors，只在 `doc/analysis/` 内据此定位并复用唯一章节。
3. 若旧状态缺少这些字段，只在 `doc/analysis/` 及其 sibling 分析产物中，用稳定 unit ID、章节标题和指向源码/配置的 evidence 锚点做组合匹配。唯一命中时复用其中的事实内容，一次性补齐统一 `source/evidence` 和 unit 边界标记，再参与整体重写；不得为规范化回读业务源码。
4. 旧 completed 单元零命中或多命中时，将该单元标为 `failed`，原因为 `state_inconsistent`；不复制可疑内容、不回读源码，也不重跑或改写其他 completed 单元。
5. 仅把 `pending`、`in_progress` 或原本未完成的 `failed` 单元放入分析队列；复用成功恢复的证据锚点和跨域边。
6. 不因旧状态缺少可选字段而重置整个分析。若用户给出精确最小状态对象，最终保持该对象的键集合，不擅自扩展；恢复映射仅在本次执行中维护。

采用稳定单元 ID：`<domain>::<relative-entry-path>#<symbol-or-flow>`；同一事实只归一个主单元，其他单元引用它。除用户要求精确最小 JSON 外，全新任务的每个 unit state 都明确记录以下恢复信息，并在设为 completed 前填好锚点：

```json
{
  "id": "<stable-unit-id>",
  "status": "pending",
  "artifact": "<path-relative-to-doc/analysis>",
  "section_anchors": {
    "start": "<!-- code-analyzer:unit=<id>:start -->",
    "end": "<!-- code-analyzer:unit=<id>:end -->"
  },
  "evidence_anchors": []
}
```

单元状态只在 `pending → in_progress → completed|failed` 之间变化。每完成一个单元即持久化其状态；一个单元失败时保留其他已完成单元，并继续处理不依赖它的单元。

### 2. 只做浅层证据发现

在读取业务实现正文之前完成一次仓库盘点：

1. 列出相对文件路径和顶层模块，排除版本控制目录、`doc/analysis/`、第三方依赖、vendor、构建产物、coverage、缓存与明显生成文件。
2. 读取少量根级或模块级清单、构建配置、入口注册和 schema/migration 目录索引。依赖声明只能作为发现线索，不能单独证明某域真实存在。
3. 通过文件名、目录和符号搜索定位每个域的少量正证据；此阶段不要逐个读取所有源文件，也不要把“大仓策略”变成最终读取剩余全部文件。
4. 记录搜索范围和候选入口。后续域分析复用这份清单，不再次全仓列举或全仓搜索同一问题。

使用以下证据含义判断域，不按扩展名或框架名猜测：

| 域 | 判为 `relevant` 的正证据 | 判为 `skipped` 的条件 |
| --- | --- | --- |
| `web` | 浏览器 UI 入口、路由或可渲染组件与其实际注册关系 | 定向检查后没有浏览器界面实现 |
| `mobile` | 移动应用入口或清单、页面/组件与导航或状态实现 | 没有移动应用目标及其 UI 实现 |
| `backend` | 已注册的 HTTP/RPC handler、事件消费者、worker、定时任务或 CLI 入口 | 没有服务端或后台触发入口 |
| `data_infra` | schema/migration、持久化查询、cache/queue，或实际 CI/部署/基础设施/observability 配置与代码 | 上述能力均无仓库证据；普通应用编译或依赖清单本身不算该域 |
| `ai` | 实际模型调用或 prompt、embedding/retrieval、tool/agent、evaluation 等流水线实现 | 只有名称、文档或未被调用的依赖，未找到实现路径 |

### 3. 先写路由计划，再加载域合同

在任何深度分析前写出 `route-plan.json`。固定使用五个域，值只能是 `relevant` 或 `skipped`：

```json
{
  "domains": {
    "web": "relevant",
    "mobile": "skipped",
    "backend": "relevant",
    "data_infra": "relevant",
    "ai": "skipped"
  }
}
```

上例只说明 schema，不代表默认判断。始终用当前仓库证据替换每个值。为兼容精确 JSON 合同，把路由理由和发现证据写进学习地图，不追加到 `route-plan.json`。

只读取 `relevant` 域对应的参考合同：

- `web`：读取 [references/web.md](references/web.md)。
- `mobile`：读取 [references/mobile.md](references/mobile.md)。
- `backend`：读取 [references/backend.md](references/backend.md)。
- `data_infra`：读取 [references/data-infra.md](references/data-infra.md)。
- `ai`：读取 [references/ai.md](references/ai.md)。

不要读取、调用或执行 `skipped` 域的专项分析。只在路由摘要中记录其跳过证据，不生成空 page、card、API 或 AI 模板。

### 4. 建立有界单元并复用读取结果

依据相关域的真实入口划分少量业务流单元，不按每个文件机械建单元。为每个单元记录：稳定 ID、入口锚点、需回答的问题、限定路径、直接依赖和状态。

执行每个单元时：

1. 只读取该入口、直接调用链和为解析下一跳必需的定义；先查本次证据台账，再决定是否读文件。
2. 让每个源文件由一个主单元负责提取事实；其他单元引用同一证据台账。尽量每次运行只读一次正文，只有锚点不完整或调用边冲突时才定向复读。
3. 禁止每个域重新扫描整个仓库。禁止为了填满模板追踪无关工具、测试、生成代码或第三方实现。
4. 对跨域边读取相邻域已登记的入口和必要实现；无法在当前仓库闭合时，在该边标为 `source: unavailable`，不要扩大为无界扫描。
5. 在写单元章节前把状态设为 `in_progress`；证据和章节校验通过后再设为 `completed`。记录失败原因并保持其他单元可恢复。

混合仓库至少输出一条证据相连的端到端路径，例如 UI → API → persistence，或 UI/API → retrieval → model → structured output/citation。每一跳单独标注 `source` 和 `evidence`；缺失的中间连接必须显式显示，不得用常见架构补齐。

### 5. 经统一解释层幂等生成学习地图

按稳定单元 ID 和相对路径排序，重建目标学习地图，不在文件尾部盲目追加。

- 对带完整恢复字段的 `completed` 单元，按 artifact 和 anchors 复用且只放入一次，不读取业务源码。
- 对重试单元，替换其原有章节；可使用唯一边界标记 `<!-- code-analyzer:unit=<id>:start -->` 与对应 `end` 标记定位，禁止产生第二个同 ID 区块。
- 对旧状态的唯一映射章节，只复用已有事实并规范一次；零或多命中的 `state_inconsistent` 单元不写入学习地图。收集所有可恢复章节和新完成章节后，整体重写目标文件一次。
- 不写当前时间、随机 ID 或遍历顺序等不稳定内容。相同输入与状态再次执行应得到相同文件内容。
- 只覆盖本任务约定的分析产物；不得清理或重写 `doc/analysis/` 中无关文件。

先按统一解释合同组织各单元，再生成学习地图。学习地图至少包含：

1. **范围与静态边界**：目标范围、排除项、不能由源码证明的内容。
2. **路由摘要**：五域的 `relevant/skipped`、理由、`source` 和 `evidence`。
3. **相关域单元**：只输出相关域合同要求的真实单元和链路。
4. **跨域路径**：混合仓库输出至少一条；单域仓库说明没有可构造的跨域边。
5. **缺失与跳过**：区分“仓库未实现所以 skipped”与“边界外实现所以 unavailable”。

### 6. 收尾校验

完成前逐项检查：

- 所有写入路径都位于 `<target_root>/doc/analysis/`，且业务文件没有被改动。
- `route-plan.json` 已在深度分析前生成，只含合同允许的键和值；精确 JSON 与用户给定对象相等。
- 没有为 `skipped` 域运行专项合同或生成伪造单元。
- 每条关键结论和调用边同时含有效 `source` 与相对路径 `evidence`；`inferred` 至少有两个锚点。
- 所有专有名称都能回链到源码或配置，不存在占位名称。
- 每个稳定单元 ID 只出现一次，完成单元未被重复执行或重复追加。
- 仅当所有相关单元均已完成时才把顶层状态写为 `completed`；按用户精确要求写最终状态，不擅自增加字段。

向用户报告实际产物路径、相关/跳过域、静态分析限制和失败单元。不要把静态结构校验描述成宿主自动路由、并行执行或真实运行效果的证明。
