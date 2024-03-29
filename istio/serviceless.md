# Serviceless

## 什么是Serviceless
- Serverless 直译过来就是无服务器，无服务器并不代表 Serverless 真的不需要服务器，只不过服务器的管理，以及资源的分配部分对用户不可见，而是由平台开发商维护。
- Serverless 不是具体的一个编程框架、类库或者工具，它是一种软件系统架构思想和方法。它的核心思想是：用户无需关注支撑应用服务运行的底层资源，比如 CPU、内存和数据库等，只需要关注自己的业务开发就行了。

## Serverless核心特点
- 无穷弹性计算能力：根据请求，自动水平扩容实例，拥有近乎无限的扩容能力。
- “零”运维：不需要申请和运维服务器。
- 极致的伸缩能力：能够根据 CPU、内存、请求量等指标敏感地弹性伸缩，并支持缩容到 0。
- 按量计费：真正按使用量去计费。

## Serverless 有 3 种技术形态
- Serverless 容器，Serverless 容器可以承载业务的核心架构
- 云函数，云函数则可以很好地适配触发器场景
- BaaS（Backend as a Service)，BaaS 则可以满足我们对各种其他 Serverless 组件的需求，例如 Serverless 数据库、Serverless 存储等。
<br>

腾讯云例子如下：
- EKS：弹性容器服务（Elastic Kubernetes Service）是腾讯云容器服务推出的无需用户购买节点即可部署工作负载的服务模式。EKS 完全兼容原生 Kubernetes，支持使用原生方式购买及管理资源，按照容器真实使用的资源量计费。
- SCF：云函数（Serverless Cloud Function）是腾讯云为企业和开发者们提供的无服务器执行环境，帮助你在无需购买和管理服务器的情况下运行代码。你只需使用平台支持的语言编写核心代码，并设置代码运行的条件，就能在腾讯云基础设施上弹性、安全地运行代码。
- TDSQL-C：云原生数据库（Cloud Native Database TDSQL-C）是腾讯云自研的新一代高性能高可用的企业级分布式云数据库，具有高吞吐量、高可靠性等优点。