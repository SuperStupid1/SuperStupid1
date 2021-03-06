# 什么是APM系统？

​		**APM**（**Application Performance Management** ）

​		**应用性能管理(APM)**，对企业系统即时监控以实现对应用程序性能管理和故障管理的系统化的解决方案。

​		应用性能管理是一个比较新的网络管理方向，主要指对企业的关键业务应用进行监测、优化，提高企业应用的可靠性和质量，保证用户得到良好的服务，降低IT总拥有成本(TCO)。一个企业的关键业务应用的性能强大，可以提高竞争力，并取得商业成功，因此，加强应用性能管理(APM)可以产生巨大商业利益。

​		**数据维度** 从数据类型划分，大体可分为

- ​			**日志（`logs`）**：自动埋点/手动埋点

- ​			**指标监控(`metrics`)**：服务、端点、实例的各项指标

- ​			**调用链(`tracing`)**: 同一TraceId的调用序列



​		**功能维度** 从业务角度划分，可分为：

- ​			**基础监控** ：应用服务的基本性能，物理机/虚拟机的指标

- ​			**中间件监控**：kafka Db Redis Zk 等依赖项的性能

- ​			**业务监控**：根据业务需求定制监控内容（业务接口，性能分析）

# APM能干什么？

​		自SpringCloud问世以来，微服务以席卷之势风靡全球，企业架构都在从传统SOA向微服务转型。然而微服务这把双刃剑在带来各种优势的同时，也给运维、性能监控、错误的排查带来的极大的困难。

　　在大型项目中，服务架构会包含数十乃至上百个服务节点。往往一次请求会设计到多个微服务，想要排查一次请求链路中经过了哪些节点，每个节点的执行情况如何，就成为了待解决的问题。于是分布式系统的APM管理系统应运而生。

​		目前大部分的APM系统都是基于Google的Dapper原理实现，我们简单来看看Dapper中的概念和实现原理。

​		先来看一次请求调用示例：

　　		1、服务集群中包括：前端(A)，两个中间层(B和C)，以及两个后端(D和E)

　　		2、当用户发起一个请求时，首先到达前端A服务，然后A分别对B服务和C服务进行RPC调用;

　　		3、B服务处理完给A做出响应，但是C服务还需要和后端的D服务和E服务交互之后再返还给A服务，最后由A服务来响应用户的请求;

​		那么在分布式环境下我们的链路追踪以及监控就更加不方便了，这个时候就需要我们的APM系统了。

# APM核心概念？

​		那么APM系统大概有哪些核心概念以及组件呢？

​		分别是如下:

​					**Agent(数据采集)**

​					**Collector(数据加工)**

​					**Storeage(数据存储)**

​					**UI(数据展示)**



​		**Agent(数据采集): **	如何在**广度**和**效率**上进行数据采集

​		**Collector(数据加工): 	**数据统一格式的整理、调用链集合

​		**Storeage(数据存储):**	将计算出的指标和聚合链路信息实时保存起来

​		**UI(数据展示):**	高颜值、多功能显示

# 主流的APM区别？

​		目前主流的APM系统有如下几种:

​						**CAT**

​						**SkyWalking**

​						**Zipkin**

​						**Pinpoint**

​						





​						

​						