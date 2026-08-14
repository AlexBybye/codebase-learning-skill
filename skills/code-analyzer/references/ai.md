# AI 域合同

仅在 `route-plan.json` 将 `ai` 标为 `relevant` 时读取本文件。不得联网调用模型、向量库或外部评测服务。

## 确认证据

要求至少存在一段被入口使用的 AI 实现，例如实际 model/provider 调用、prompt 组装、embedding/retrieval、tool/agent 调度或 evaluation 执行路径。只有 SDK 依赖、环境变量、文档描述、类型定义或未被调用的样例时，不足以把 AI 域判为 relevant。

记录源码发现的实际 provider、模型、索引、工具和协议名称；没有证据时不填通用示例。

## 按流水线划分单元

从真实入口建立少量端到端 flow，并逐项判断以下阶段：

1. **ingestion**：数据源、解析、清洗、文档身份和更新入口；
2. **chunk/embedding**：切分规则、metadata、embedding 调用、batch 与存储；
3. **retrieval/reranking**：query 处理、filter、top-k、检索器、reranker 和阈值；
4. **prompt/model call**：模板来源、消息组装、provider/model、参数和超时；
5. **output/citation**：结构化输出 schema/parser、验证、引用或来源映射；
6. **tools/agents**：工具注册、参数合同、选择/循环/停止条件和权限边界；
7. **evaluation**：数据集入口、metric/judge、阈值和执行触发；
8. **safety/privacy**：输入输出过滤、敏感数据处理、授权与数据发送边界；
9. **cost/latency**：静态 token/batch/cache/concurrency/timeout/retry 设置；
10. **fallback**：provider/model/retrieval/parser 失败后的降级或返回路径。

某阶段没有实现证据时标为 `skipped`；代码引用了仓库外服务或资产而无法查看其实现时标为 `unavailable`。不要为了填满十项扩大到无关目录。

## 追踪链路

按源码实际情况还原，例如：

```text
ingestion trigger
  → parse/chunk
  → embedding
  → index write

user/API input
  → query transform
  → retrieval/filter
  → optional rerank
  → context selection
  → prompt assembly
  → model call
  → parse/validate
  → citation or response
```

引用必须追到 chunk/document metadata 如何映射到最终输出；只看到模型生成文本而没有映射实现时，把 citation 标为 `unavailable`。若入口位于 web/mobile/backend，复用对应域的入口和 handler 证据。

## 关键合同

### Prompt 与结构化输出

记录 prompt 的存放位置、变量、拼装顺序、role、输出约束与 parser/schema。不得输出真实凭据或从名称猜 prompt 内容。将静态 schema 存在与运行时模型遵守 schema 区分开。

### Retrieval 与评测

记录代码中的 filter、top-k、阈值、rerank 顺序、评测数据入口和 metric/judge 配置。不得从这些设置声称召回率、准确率、质量或一般化效果。

### Safety、privacy、cost 与 fallback

只记录显式控制及其调用位置。区分“存在过滤函数”和“已证明安全”，区分“设置 timeout/token limit”和“已证明延迟/成本”。追踪异常捕获到 fallback 输出的真实路径；只有注释或配置而没有调用时标为 `unavailable` 或 `inferred`。

## 输出重点

在统一解释结构中，必须包含阶段状态表及证据、至少一条当前源码可闭合的 AI 主路径、与 UI/API/data 的跨域边、存在的 prompt/model/structured-output/citation 合同，以及 evaluation、safety/privacy、cost/latency、fallback 的存在或缺失边界。

结尾明确说明静态源码无法证明线上质量、性能、成本、安全或部署状态。
