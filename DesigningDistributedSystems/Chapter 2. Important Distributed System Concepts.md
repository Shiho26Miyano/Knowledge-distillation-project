# Chapter 2 – Important Distributed System Concepts  
# 第 2 章 – 分布式系统关键概念（核心观点清单）

---

## 1) 分布式系统的本质是 API / RPC  
## 1) Distributed systems are fundamentally API/RPC

- **分布式系统里，服务之间通过网络调用完成协作，这类调用就是 RPC。**  
  In distributed systems, services collaborate via network calls—these are RPCs (Remote Procedure Calls).

- **常见实现是 HTTP + JSON（REST），也可能是 gRPC / CORBA 等更结构化协议。**  
  Common implementations are HTTP + JSON (REST), but more structured protocols exist (e.g., gRPC, CORBA).

- **API 是系统的“连接面”，没有清晰 API，就没有可演化的分布式系统。**  
  APIs are the system’s “connection surface.” Without clear APIs, a distributed system can’t evolve cleanly.

---

## 2) 同步 vs 异步 API：用执行时长决定  
## 2) Sync vs Async APIs: decide by operation duration

- **同步 API：请求处理完才返回，最简单，但不适合长耗时操作（容易超时）。**  
  Synchronous APIs return only after processing completes—simple, but poor for long-running tasks (timeouts).

- **异步 API：先接受请求，返回 operation ID，后续用查询接口追踪进度。**  
  Asynchronous APIs accept the request, return an operation ID, and let clients poll status later.

- **经验规则：操作耗时按分钟计 → 应设计为异步。**  
  Rule of thumb: if it takes minutes, model it as async.

---

## 3) API 需要“可用的客户端生态”  
## 3) APIs need a usable client ecosystem

- **让每个开发者手写 HTTP 调用，会导致重复劳动、易错、接口设计混乱。**  
  Having every developer hand-roll HTTP calls causes duplication, bugs, and messy interfaces.

- **大系统通常用 IDL（如 OpenAPI）描述接口，从而自动生成多语言 client library。**  
  Large systems use IDLs (e.g., OpenAPI) to define APIs and auto-generate multi-language clients.

---

## 4) SLO 是 API 的“合同”  
## 4) SLOs are the API’s “contract”

- **API 的核心不是“长什么样”，而是“表现如何”。需要明确：**  
  The core of an API isn’t just its shape—it’s how it behaves. Define:

  - **Latency（延迟） / Latency**  
  - **Reliability（可靠性） / Reliability**  
  - **Throughput（吞吐/RPS） / Throughput (RPS)**

- **有了可量化 SLO，客户端/上下游才能做容量规划、超时设置、重试策略与依赖管理。**  
  With measurable SLOs, clients can do capacity planning, timeouts, retries, and dependency management.

---

## 5) Latency 不只要测“快不快”，还要能归因  
## 5) Latency isn’t just “fast or slow”—it must be attributable

- **延迟要反映真实用户体验（后面会讲 percentile）。**  
  Latency should represent real user experience (percentiles matter).

- **关键是 attribution（归因）：知道慢来自哪里，否则只能“知道慢”，却无法优化。**  
  The key is attribution: knowing *where* latency comes from; otherwise you can’t improve it.

---

## 6) Reliability 的定义很复杂：HTTP 成功不等于业务成功  
## 6) Reliability is tricky: HTTP success ≠ business success

- **HTTP 协议层面“成功”非常常见，但业务层可能是错误码返回。**  
  Protocol-level “success” is common, but business-level failure may appear in response codes.

- **50x 通常是明显服务端错误；20x 通常成功。**  
  50x usually indicates server errors; 20x usually indicates success.

- **30x/40x 有灰区：**  
  30x/40x are gray areas:
  - **404 可能是用户输入错，也可能是服务配置/路由出问题**  
    404 may be client mistake—or service misconfig/routing issues.
  - **403 可能是权限不足，也可能是 auth 服务故障**  
    403 may be lack of permission—or an auth service outage.

- **结论：可靠性指标必须结合业务语义定义，否则会误报/漏报。**  
  Conclusion: reliability metrics must be defined with business semantics, or you’ll get false alarms/missed outages.

---

## 7) 平均值会骗人：要看 P90/P99 的尾延迟  
## 7) Averages lie: focus on tail latency (P90/P99)

- **平均值容易被少量极快/极慢请求扭曲，不能代表用户体验。**  
  Averages are skewed by outliers and don’t represent user experience well.

- **用户体验往往由多个请求组成：请求越多，出现一次“慢请求”的概率越接近 100%。**  
  User journeys involve many requests; as requests increase, the chance of at least one slow request approaches 100%.

- **因此要用 percentiles（如 P90/P99）观察“多数用户的真实体验”，并明确你测的是哪条用户路径/哪类请求。**  
  Use percentiles (P90/P99) to reflect real experience, and be explicit about which user journey/request type you measure.

---

## 8) Idempotency（幂等）是分布式系统“重试”的前提  
## 8) Idempotency enables safe retries

- **幂等：执行一次或多次结果相同（如 X=5）。**  
  Idempotent: same result whether executed once or many times (e.g., `X=5`).

- **非幂等：重复执行会改变结果（如 X=X+1）。**  
  Non-idempotent: repeated execution changes state (e.g., `X=X+1`).

- **分布式环境常见失败（超时、断连）会触发重试，幂等让你可以“重试直到成功”，而不引入重复副作用。**  
  Failures (timeouts, disconnects) trigger retries; idempotency lets you retry until success without duplicate side effects.

---

## 9) 交付语义：现实中通常只有 at-least-once / at-most-once  
## 9) Delivery semantics: usually at-least-once or at-most-once

- **At least once：至少送达一次，但可能重复 → 通常要求幂等或去重。**  
  At-least-once: delivered at least once, may duplicate → require idempotency or deduplication.

- **At most once：最多送达一次，但可能丢 → 通常需要补偿机制/人工重试。**  
  At-most-once: never more than once, may drop → need compensation or manual retry.

- **Exactly once：实践中非常难，通常不作为默认设计目标。**  
  Exactly-once is extremely hard in practice and usually not the default design goal.

---

## 10) Relational Integrity（关系完整性）是性能 vs 复杂度的权衡  
## 10) Relational integrity trades off performance vs simplicity

- **当业务依赖多表/多存储之间“强一致关系”，就需要跨存储协调与事务，系统更复杂、并发更差。**  
  If your business requires strong cross-store/table consistency, you need coordination/transactions—more complexity and less parallelism.

- **SQL 通常提供更强的关系一致性与事务语义；许多 NoSQL 选择弱一致/键值模型，把一致性问题推给应用层。**  
  SQL typically provides stronger relational consistency and transactions; many NoSQL systems relax this and push consistency to application code.

- **结论：更强一致性 → 代码更简单但性能/扩展性更差；更弱一致性 → 性能更好但应用复杂度更高。**  
  Conclusion: stronger consistency simplifies code but hurts performance/scale; weaker consistency improves performance but increases app complexity.

---

## 11) Data Consistency（强一致 vs 最终一致）影响巨大且难回头  
## 11) Data consistency (strong vs eventual) is a hard-to-reverse choice

- **强一致：写入必须在所有副本/地域确认后才算成功 → 写吞吐和延迟会显著变差。**  
  Strong consistency: writes complete only after all replicas/regions confirm → worse throughput and latency.

- **最终一致：写入先在部分地点成功即可返回，其余异步复制 → 性能更好，但会出现“读不到刚写的数据”（read-your-own-writes 问题）。**  
  Eventual consistency: return after partial acknowledgment; replicate async → better performance, but may not “read your own writes.”

- **一旦选择一致性模型，会渗透到系统设计和代码里，后期很难更改。**  
  Once chosen, consistency permeates system and code—hard to change later.

---

## 12) CAP：优化一个维度，必然牺牲另一个维度  
## 12) CAP: optimizing one property sacrifices another

- **CAP 指出在网络分区条件下，不可能同时保证一致性、可用性、分区容忍都完美。**  
  Under network partitions, you can’t perfectly guarantee consistency, availability, and partition tolerance all at once.

- **它不是“只能二选一”的教条，而是提醒：你在追求某个性质时，必然在牺牲其他性质，要根据业务做可接受的权衡。**  
  Not a rigid “pick two” slogan—rather a reminder that optimizing one property trades off others, so choose based on business needs.

---

## 13) Orchestrator（如 Kubernetes）负责“运行态的对齐”  
## 13) Orchestrators (e.g., Kubernetes) keep runtime aligned with desired state

- **现代分布式系统依赖编排系统来维持“期望状态 = 实际状态”（例如：要 3 个副本就保持 3 个）。**  
  Modern systems rely on orchestrators to keep actual state aligned with desired state (e.g., keep 3 replicas running).

- **编排系统不实现业务逻辑，但负责部署、重启、扩缩、调度等运维动作。**  
  Orchestrators don’t implement business logic; they handle deployment, restarts, scaling, and scheduling.

---

## 14) 健康检查分两类：Liveness vs Readiness  
## 14) Health checks: liveness vs readiness

- **Liveness：还活着吗？不活 → 重启。**  
  Liveness: is it alive? If not → restart.

- **Readiness：准备好接流量吗？没准备好 → 不加入负载均衡。**  
  Readiness: ready to serve traffic? If not → don’t send traffic / don’t add to load balancer.

- **典型场景：应用启动后需要初始化（连 DB、加载文件）。此时“活着但未就绪”，readiness 能避免把流量打到半成品实例上。**  
  Typical case: app starts and initializes (DB connections, loading files). It’s alive but not ready—readiness prevents routing traffic to a half-initialized instance.
