
  LangEngine作为阿里集团内部发起的纯Java版本的AI应用开发框架，经过充分实践，已经广泛应用于包括淘宝、天猫、阿里云、爱橙科技、菜鸟、蚂蚁、飞猪、1688、LAZADA等在内的多个业务场景。此外，LangEngine还支撑了阿里国际AI应用搭建平台的自研与上线，对集团内部的AI平台基础设施产生了深远影响。


 


**目前，该框架在阿里集团内部钉钉群中拥有超过1200名开发者，社区贡献群体中有超过40名核心贡献者。**


**现在阿里LangEngine正式对外开源。开源链接：**[https://github.com/AIDC\-AI/ali\-langengine](https://github.com)


 


随着阿里集团AI业务的快速发展，AI应用网关的高可用性和稳定性变得尤为重要。在本文中，我们将介绍在构建高可用网关过程中，LangEngine应用框架的一些架构设计理念和经验总结。我们的分享旨在为大家提供实用的指导，帮助在构建高效稳定的AI应用时避免不必要的弯路。


 


**LangEngine的网关架构执行**


**设计架构**


![](https://images.cnblogs.com/cnblogs_com/liping13599168/2435080/o_241207121240_640.webp)


**通讯协议：**LangEngine框架支持接收HTTP的非流式和流式请求，也支持了集团HSF的非流式和流式内部请求，可以提供高效的服务通信能力。


**运行时协议：**包括普通API协议和消息协议，用于标准数据传输和事件通知。LangEngine支持工作流协议、异步化轮询以及MetaQ消息协议，以实现复杂任务的编排和调度。


**集成AI框架：**LangEngine框架提供全生命周期的AI应用执行链路的代码实现。内置权限访问和流控插件扩展，以确保安全性和资源的有效利用。


**AI组件网关****：**应用运行时通过统一内部各项服务，网关负责将请求路由到后台的HTTP、HSF、算法工作台以及消息服务。


**元数据缓存：**采用多级数据缓存策略，降低数据库访问频率，以高效更新和获取AI应用的元数据信息。未来也会集成到LangEngine开源框架中。


**应用运维：**常规的SLS Log打点，通过Blink聚合计算回流Hologres生成实时报表，采用UniqueId以及RpcId树状机制提供统一日志SDK进行全链路的日志回流。


通过这些能力结合，系统能够在复杂的应用场景下保持高性能和高可靠性。


 


**流式与非流式输出**


和传统的API网关一样，采用了HTTP异步Callback方式，可以在不阻塞主线程的情况下进行异步请求处理，这对于提高应用响应速度和用户体验是非常重要。LangEngine\-AgentFramework框架内置的工作流引擎，可用于编排微服务架构中的多项服务，以极高的性能和低存储成本启动/发送流程实例，也可用于传统的流程审批场景，在进入某个组件节点执行调用时候，通过异步化回调来做节点的线程上下文切换，来提升线程的资源池利用率。


**非流式输出**


![](https://img2024.cnblogs.com/blog/18497/202412/18497-20241207202004959-1800558670.png)


**流式输出**


流式输出是一种数据处理和传输方式，可以逐步传送数据，而不是一次性将所有数据发送完。这种方式在多种场景中都有显著的优点：


* **降低内存使用：**流式输出允许数据逐段传输和处理，不需要将整个数据集存储在内存中。对于处理大文件或数据流，这可以显著降低内存占用，提高应用的可伸缩性。
* **减少延迟：**通过流式输出，数据可以在生成后立即开始传输，而不必等待整个数据生成完成。这对于实时应用尤其重要，可以大大降低端到端的延迟。
* **处理大数据集：**流式输出允许处理和传输大小超过可用内存的数据集，因为它不需要在内存中保留整个数据集。
* **提高响应速度：**在 Web 应用中，流式输出可以让用户更快地看到部分数据。例如，视频流可以让用户在数据还未完全加载时就开始观看。
* **减少服务器负载：**通过流式传输，服务器可以将计算和传输的工作量分配到更长的时间段内，而不是瞬间处理大量数据请求，从而更好地管理资源和负载。


大模型的推理输出是个很耗时的过程，运行时链路中也需要支持了流式输出的功能。因此，LangEngine AgentFlow Engine目前支持HTTP Stream和HSF Stream两种方式。


![](https://img2024.cnblogs.com/blog/18497/202412/18497-20241207224235221-292002967.png)


 


**元数据多级缓存**


随着平台用户的增多，平台的请求量和数据访问频率也在迅速提升，这对平台、尤其是核心调用链路的响应时间和吞吐量提出了更高的要求。引入多级缓存架构是提升平台性能的常用手段，为了减少重复开发、提升系统可复用性，设计实现agentpaas通用缓存框架，供应用、工具、RAG执行模块使用。


**问题\&挑战：**


1\. 应用执行链路中涉及的信息类型多种多样，有apikey这样的String，也有agent运行实例类，如何使得缓存配置尽可能轻量又保证缓存的通用性。


2\. 对于数据插入、更新、删除时的数据一致性保证。


**解决方案：**


构建 本地缓存 \-\> 分布式缓存 \-\> 数据库 的分级存储架构


![](https://img2024.cnblogs.com/blog/18497/202412/18497-20241207224326612-450325968.png)


通过实施多级缓存策略，可以有效提高API网关在高并发业务场景下的性能和可靠性，从而保障系统的快速响应和稳定运行。


 


**LangEngine的核心处理单元**


**设****计架构**


![](https://img2024.cnblogs.com/blog/18497/202412/18497-20241207224454612-1933688528.png)


LangEngine\-Core：LangEngine核心模块，包括六大模块：Retrieval、Model I/O、Memory、Chains、Agents、Callbacks，以及LangRunnable的动态链编排引擎。开源代码：[https://github.com/AIDC\-AI/ali\-langengine/tree/main/alibaba\-langengine\-core](https://github.com)


LangEngine\-Community：LangEngine社区生态模块，目前主要会针对六大模块的可扩展功能进行共建。开源代码：[https://github.com/AIDC\-AI/ali\-langengine/tree/main/ali\-langengine\-community](https://github.com):[wgetcloud加速器官网下载](https://longdu.org)


 


**LangEngine\-Core工作原理**


![](https://img2024.cnblogs.com/blog/18497/202412/18497-20241207224621411-2140130065.png)


整体上，LangEngine分为六大模块：Retrieval、Model I/O、Memory、Chains、Agents、Callbacks。详细介绍可以看：[https://mp.weixin.qq.com/s/MdixFZx6MklBXC0\-2dX2wA](https://github.com)


特点：阿里体系下基于LLM的AI应用开发框架；引入Java特色的工程模块化思路，可支持日志记录、业务监控、链式编排，实现了类流程持久化；面向阿里系Java工程开发同学，易学易用。支持社区生态共建


LangEngine框架目前在内部钉钉群里有1200\+的使用方和开发者，内部社区贡献群有40\+核心贡献者。


 


**LangRunnable架构**


**![](https://img2024.cnblogs.com/blog/18497/202412/18497-20241207224706445-1979648638.png)**


可以轻松地从基本组件构建复杂的链条。


统一的接口：每个 LangRunnable 对象都实现 Runnable 接口，该接口定义一组通用的调用方法（invoke、batch、stream、invokeAsync 等）。这使得 LangRunnable 对象链也可以自动支持这些调用。也就是说，每个 LangRunnable 对象链本身就是一个 LangRunnable 对象。


组合原语：LangRunnable 提供了许多原语，可以轻松组合链、并行化组件、添加后备、动态配置链内部等。


 


**LangEngine\-Community社区共建**


重新针包扩展模块分离（按需引用pom包），聚焦在各个模块能力扩展社区生态共建。


![](https://img2024.cnblogs.com/blog/18497/202412/18497-20241207224802053-1180374842.png)


 


**LangEngine实现AIGC和LLM的高可用**


**异步化设计**


**应用同步\+单节点异步**


该设计方案仍然是通过应用同步请求的方式来使用，对于部分组件的能力本身如果仅支持异步化请求，可以对该节点单独设置可执行异步化操作。这样的优点是，用户接入通过同步HTTP方式即可最终得到结果，不用感知里面的执行逻辑。而对于网关来说，针对于部分节点进行异步化任务提交，通过CompletableFuture进行任务的状态轮询，当任务的状态为finished或者failed时候，再继续下一节点的执行。


虽然这种同步方式比较轻量也能达到部分节点异步化的目的，但仅能支持异步化比较快速并不产生大量堆积的情况。往往上，我们的业务中会执行大量的任务跑批任务，可能存在消息队列的任务堆积，而这个堆积往往会持续比较久而慢慢去消费掉，所以当超过了最大超时时间，同步请求就会把超时信息直接异常返回。为了防止任务堆积，导致的请求超时，于是就有了应用全异步的方案。


**应用全异步**


异步化任务提交后，将当前的流程实例中断并返回结果，完成本轮调用。并将任务ID以及当前的ProcessInstanceId的持久化到Taskinstance表中，状态置为暂停。


运行时的消息监听程序将接收任务系统的对于组件的任务处理结果，任务存在finished和failed两个状态。如果任务是finished状态，将进行流程实例的恢复，执行signalProcessInstance，同时将taskInstance表中的状态置为运行，避免任务抢占，并将变量上下文从数据库捞起合并任务携带的变量结果，恢复流程实例进入到下一个节点。


在流程节点到下一个流程节点不在是在同一个主线程之中了，这样避免了主线程阻塞，并且通过持久化方式进行延迟序列化加载与恢复。


自从应用异步化上线以后，整体系统性能平稳，并且能够支撑比较大的任务堆积流量。


 


**应用Serverless**


AI应用目前整个调用链路目前都是通过中心化部署的AI应用网关上去执行的，由于AIGC与大模型相关服务耗时比较长是个常态，存在应用之间资源性能上的相互影响。借助于Serverless技术，实现应用容器化隔离。应用分层之后serverless应用面向一线开发者、基座应用面向SRE的角色，让应用不再关心基础组件「业务二方包/三方包/中间件/JDK/安全」的升级。


 


**应用运维集成框架**


利用LangEngine日志采集功能，通过建立全面而细致的日志监控系统，确保能够实时捕获和分析应用运行中的各类事件和异常。这有助于迅速识别和解决潜在问题，从而减少停机时间和对用户的影响。通过自研日志SDK和SLS的ilogtail采集多种类型的应用日志，并使用Sunfire和BLINK工具进行计算，随后通过BLINK SQL将数据存入HOLO数据库，以支持快速检索，同时提供监控、个性化告警、数据分析等平台功能。


 


**总结**


以阿里LangEngine的AI应用开发框架为核心基础，开发者可以快速高效地构建高可用的AI应用网关。通过结合完善的应用运维体系，LangEngine为AI应用的整体稳定性提供了坚实的保障，使开发者能够专注于业务逻辑和创新，而不必担心底层架构和稳定性问题，从而更好地应对不断变化的市场需求。阿里LangEngine在未来将重点探索以下几个方向，并热忱欢迎开源社区的贡献者共同创造，GitHub地址为：[https://github.com/AIDC\-AI/ali\-langengine](https://github.com)


**1\. AgentFramework即将开源**：开发者可以基于AgentFramework框架快速构建AI的工作流应用和智能体应用。


**2\. 流式与智能体异步化支持**：当前，异步化处理已在工作流应用中发挥作用。未来计划将这一异步化方式扩展到流式处理和智能体应用中的CoT（Chain of Thought）规划。这将提升系统的灵活性和响应能力，使其能够更高效地处理复杂任务。


**3\. LangEngine\-Multi\-Agent框架：**探索多智能体执行引擎，结合multi\-agent范式和agentic workflow理念。此项发展将推动系统从传统的Flow架构向智能Flow架构转变。


**4\. LangEngine\-Platform：**提供对外可视化的AI应用搭建的开源工具，并支持一键部署RestAPI统一网关。


通过这些探索与创新，阿里LangEngine将继续推动AI应用的智能化和高效化发展。  


