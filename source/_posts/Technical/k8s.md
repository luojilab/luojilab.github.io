title: 【八里庄技术沙龙-14 期】Kubernetes在得到App的落地实践
date: 2019-10-21
author: sunqingyun
toc: true
thumbnail: https://piccdn.luojilab.com/fe-oss/default/MTU2NzE1Mjk5Mjg1.jpeg
tag: 
  - 八里庄技术沙龙 
  - Kubernetes
  - Docker
categories: 
  - 八里庄技术沙龙

---

## 引言

罗辑思维是一家创业公司，主要产品有：得到App。主要有两类业务，线上: 订阅课程、商城、听书、讲座、电子书，线下：跨年演讲，得到大学，线下大课等。目前有高质量用户3300万，后端服务以容器方式运行，正在基于Kubernetes进行混合云建设，目前线上主要的主机资源是使用的阿里云。

由于技术选型比较“激进”，并且践行微服务架构设计，目前的语言栈有：按照占比排名，Golang、Node.js、Python、Java、PHP、C++，之前使用云主机（ECS）带来的运行环境管理复杂、发布过程不统一等问题。所以，将应用容器化以及微服务治理，一直是较为迫切的需求。

<!-- more -->

从2013年底Docker开源，到现在已经发展了超过5年的时间，大家已经听说容器技术的优势和收益并逐渐接受准备拥抱之。但想要落地容器以及容器管理系统Kubernetes这种新一代基础设施，远没有想象中容易，在新技术落地的过程中，阻碍往往不是来自于技术本身，而在于观念的更新、生态的丰富以及易用性。

首先介绍一下容器技术。容器是Linux Kernel的功能模块封装，主要基于有十年年以上历史的namespace(since1992)和cgroups(since2007)，容器镜像是一类CopyOnWrite的Overlay文件系统，跟虚拟机的主要区别是，容器间共享宿主机系统内核，所以启动速度较虚拟机更快，可以达到秒级。但简单来说，每个容器是一个进程以及它所拥有的资源和边界。Docker是目前容器技术的事实标准，也可能是未来应用的交付标准。

![1.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_109865868b5c33bc6e1dd39f613c4ee0.png)

然后介绍一下Kubernetes（k8s）。Kubernetes是跨主机、跨集群、跨IDC的容器管理系统，基于Google 生产负载上的 15 年管理经验（Borg），最初由 Google 的工程师设计和开发并开源，且融合了来自社区的经验与实践，已经成为企业级容器管理的事实标准。简单来说，Kubernetes是管理容器（进程）的云操作系统。

![2.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_a3c86eb5251c5fe1c8c9b019b4dbe47d.png)

## 架构演进
回顾容器技术在得到App落地的过程， 主要有四个阶段：公有云虚拟机阶段，容器化Docker阶段，Kubernetes阶段，混合云阶段。

![3.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_746373dcbc5009f540d93c4d406c10ac.png)

### 虚拟机阶段
此阶段的进步，是由发布系统代替了散落在不同代码仓库中的发布脚本，从shell脚本时代进入工具时代，使得发布不再需要运维人员参与，并且引入了发布审核机制，大大提高了发布效率。但是，新项目上线时，须经历购买服务器、系统初始化和安装运行环境、项目发版配置，调试部署过程脚本几个步骤，较为繁琐。并且需要增加实例时需要手工操作，仅适合管理少量使用ECS的服务发布。最让人头疼的是由于环境不一致引发的各种线上事故。

![4.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_1071ae4d0a345d7d24d5fd6a293ed5e4.png)

于是，我们决定用通过使用容器技术来解决这些问题。首先，制定出一系列规范：统一运行时版本、域名规范、端口规范、目录规范、日志规范等；然后基于规范开发了两层基础镜像：操作系统层、各语言运行时层；并且整理出Dockerfile模板、entrypoint.sh模板。当应用代码需要容器化时，只需要将Dockerfile和entrypoint.sh两个模板文件添加到项目代码仓库中即可，无需要任何修改，降低了改造的工作量。

### 容器阶段
通过应用的容器化部署，简化了环境管理，屏蔽了部署细节，统一了交付方式，基础设施只关注资源和容器状态，极大减少了运维工作量。同时，因为有容器的快速启动能力加持，扩容速度达到了秒级，使得应用实例的管理更加敏捷。

![5.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_4cf6edea1a9256b2a9da00e103d92bf7.png)

其底层是阿里云容器服务全家桶，上层根据发布流程和规范开发了相关功能。使用的组件有Gitlab（代码管理），Jenkins（调用其API进行docker build，并回调Dozer），Swarm API（发布、调整容器数量等核心功能），云监控（容器监控）、云日志服务（日志收集 存储 分析）、云SLB（多容器实例的汇聚和负载均衡）、云OpenAPI（添加集群Worker节点）。

该方案的优点是：简单，快速，并且有良好的商业支持，可以将精力投在内部的落地上。缺点也是明显的，即限制较多，强依赖公有云，有时候业务的需求由于公有云暂未开放相关功能，不得不进行取舍。同时，由于历史原因我们使用了Overlay网络，当时这种网络方案在高负载集群中非常不稳定，并且Swarm集群缺乏广泛的生产环境考验并不稳定，以及社区不活跃，最终我们决定更新VPC网络和容器基础设施。


### Kubernetes阶段
我们首先花费巨大的时间和精力将公有云的经典网络（Public）中的所有资源迁移到了VPC网络（Private），在此过程中，由于容器化的应用有“一次构建，随处部署”的优势，这使得我们节省了很多时间。然后基于Kubernetes进行了新的容器基础设施的构建，同时建设了自己的监控和日志收集体系。

![6.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_ded836d07936c88e3dedbd37a615a5d9.png)

通过Kubernetes来进行资源管理和交付，通过管理API来进行应用上线和发布。Kubernetes同时提供了弹性伸缩和故障自动迁移，可以应对简单的流量突增或服务器节点故障等问题。还可跟私有云或公有云的基础设施进行联动，对存储、计算资源或负载均衡设备进行自动化管理。

通过一年的努力，所有的业务流量迁移到了围绕容器和Kubernetes构建的基础设施之上。新的容器网络方案，从性能和稳定性上较之前有了本质提升。同时，Kubernetes的良好生态和优秀设计，底层服务器节点进行了标准化管理，极大简化了运维成本。它的容器配额更加高效，把所有的应用容器进行了资源保障和限制，最终提升了资源利用率。

基础设施的更新，为业务架构迭代提供了支撑，应用的开始大范围的微服务化更新，带来了成倍的管理工作，同时微服务间调用链路变长，问题排查难度增加。此时，微服务的管理成为新的挑战，于是我们开始了工具平台研发和服务治理工作。


### 微服务治理
对于服务治理，我们是在2018年初启动，这方面我们的思路是：用规范和约定来将编程框架和基础设施打通、服务以编程框架的形式连接基础设施。

开发框架：将与各个组件对接的代码和公共代码抽离到框架中，集成了服务注册和配置中心以及tracing功能。同时框架中提供Liveness和Readness探针、Graceful Shutdown等，统一日志格式等。通过使用这套框架，可以提升开发效率，统一微服务面向管理的接口。

配置中心：统一管理配置，将配置和发布包解藕，减少业务开发者维护配置的工作。

服务注册与发现：得到服务注册与服务发现的中间件，以AP为设计目标，支持多种健康检测和负载均衡方式，服务在启动时自动将自己注册到服务发现服务上。并且会实时（定期）从服务发现服务同步各个应用服务的地址列表到本地进行缓存，以起到加速的效果，同时监听远端中心事件，进行数据同步。

追踪系统：分布式链路追踪系统，由开发框架中统一封装，每个服务内嵌标准化接口，将分布式请求还原成调用链路，可以集中展示各个服务节点的请求状态，以及花费时间，进行链路追踪、问题分析。

API网关：API网关是系统与外界联通的入口，支持反向代理、重定向、限流等功能，基于服务发现服务发现中心的数据，进行后端实例的注册，API网关一般作为系统与外界联通的入口，在微服务架构中，所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能，反向代理、重定向、限流等功能。

![7.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_b3402b76a641ea720e0566823e7338fd.png)

服务启动时请求配置中心，获取运行环境配置，将自身加入注册中心、获取所依赖服务信息，启动完成后，上报健康状态，并提供API服务。微服务间通过服务发现互相感知，服务通过框架接口来维护自身上线、离线。API网关连接到服务注册中心，动态感知服务变化，并自动更新API路由，外部流量通过API网关将流量转发到对应的业务模块。

**Kubernetes在底层基础设施跟上层微服务治理组建中间，起到了承接作用。**


## 方案细节
所有Kubernetes方案中，网络方案是最重要的部分之一，由于我们的基础设施分别在公有云和私有云，虽然网络组件不同，但使用整体相似的网络模型。

### 网络（公有云）
公有云使用VPC网络，Kubernetes的网络组件，使用Flannel + alivpc Backend，每个Worker节点中的容器作为一个子网，掩码为24，并且使用NAT（地址转换），通过Flannel的alivpc插件调用vRouterAPI，将该此条路由信息写入VPC网络的虚拟路由器（vRouter）的路由表中，以此实现Pod IP与ECS IP互通，该方案设计简洁，比较稳定。

![8.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_d9d2ef476afade3c26474ba3d6031885.png)

### 网络（私有云）
私有云中的网络方案，虽然看起来跟公有云结构很相似，但是基于Calico，主要是基于BGP路由协议进行路由分发，以达到互联互通的效果。节点内使用BIRD软路由，将每个节点上的Pod所在子网，掩码为24，发送给物理设备RouteReflector进行路由学习和发布。相较公有云的网络，BGP协议更加高效和可靠，同时网络设备可对网络流量进行路径优化，间接提升了网络性能。

![9.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_23cd6c9294e33ff37c7d5b4f6af9d626.png)

### 构建
构建服务的核心是Jenkins，管理系统通过调用Jenkins的HTTP API进行任务管理，Jenkins接收到请求后将构建任务加入队列，排队构建。构建时，从Git仓库拉取代码，执行Docker build，产出Docker image，成功后push到registry存储。

CI系统的流程是：开发人员在本地进行功能开发，本地测试，当通过单元测试后，进行代码提交。Git服务接收到开发人员的提交后，通知CI系统，触发CI流程。CI系统使用Jenkins进行构建，将代码编译成制品，并产出Docker镜像，然后将镜像push到镜像仓库存储，然后回调Kubernetes系统API，更新测试和预发布环境。

![10.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_2b997ec11e9f583e672688263954ecc0.png)

### 日志方案
服务数量和应用规模变大时，需要将分布在各处的日志进行收集，集中存储，以提升日志分析效率。我们日志收集分为两类：应用日志和APM日志。

![11.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_af6f720cef244c262aad8977500f0ef9.png)

由于自建的日志系统需要较多服务器资源，后续要花费很多精力去优化，考虑到投入产出比，使用阿里云日志服务(SLS)比较有优势，目前我们容器内产生的日志都是使用ilogtail收集发往SLS，使用阿里云监控的日志关键字监控功能监控错误日志中的特定关键字，进行告警。

另外，filebeat目前只负责收集APM产生的trace日志，发往kafka，由日志处理程序来进行消费，处理后序列化到ElasticSearch，由APM系统进行使用。

### 日志收集
filebeat和ilogtail以DaemonSet方式部署，每个Worker节点上部署一个Agent。Pod使用EmptyDir易失性存储方式，通过HostPath挂载形式收集。应用将日志写入规范目录后，日志收集Agent会监听到特定事件，将新增日志取出，发往存储服务。

![12.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_052b4de24adfa7a626516d3efa82dd91.png)

### 监控
我们的Prometheus是单独的服务器，以运行在集群外部的方式，通过APIServer获取资源信息，然后对自动发现的各个endpoint进行metrics pull。导出metrics，使用了cAdvisor、node-exporter、kube-state-metrics等组件来导出不同维度的度数。prometheus拉取到数据后，根据预设阈值进行评估，触发阈值后发往AlertManager，在AlertManager中根据不同的级别对告警进行路由、沉默和收敛。通知通道有：邮件、企业微信、短信。

![13.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_61f382bf288779f9c9746baa80060555.png)

当数据量大、查询较慢时，可使用Prometheus alert中的record语句，进行数据预处理，即将查询产生的结果存入新的metrics，使用新metric绘制图表和报警的rules检查，速度会有较大提升。


### 监控（Pod）
监控数据展示使用Grafana，数据来自Prometheus，Pod级别。主要展示CPU、内存使用率，TCP连接数。

![14.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_4ee02b92968e55d95f088e28cc83cede.png)


### 监控（服务）
服务维度的监控项有：主要展示CPU、内存使用率，TCP连接数，文件描述符，nf_conntrack，IO等。

![15.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_0efb742d91a74163e87e058209bb0e67.png)


### 监控（大盘）
监控大盘，管理人员使用监控大盘，关注各集群控制平面的各系统组件监控状况，资源的分配情况，依据资源水位进行节点的增减。

![16.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_e3510e8f68040abbb29654cdb445a3ba.png)


### 踩到的坑
1. 初期由于我们的服务器还运行在阿里云经典网络（IaaS的早期多租户网络），在Swarm集群中我们使用了overlay网络，每个容器创建或删除时，由于需要集群内部广播该容器IP等信息，随着容器数量的增加，会有同步失败情况，造成服务容器间不通问题。
2. 得到App的微服务，大部分是golang语言开发，由于我们的服务多为HTTP短连接形式，并且如果请求的是域名的话，golang会直接发起dns查询，当查询量过大时会遇到“lookup failed”相关报错，需要在容器内部运行nscd服务。
3. 内核中的tcp参数，尤其是netfilter相关，对于容器网络稳定性影响较大，图中为目前我们在生产环境的Worker节点使用的内核参数。

![17.png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191022/upload_f1d5b1274057d6c0b52d8f92e30f488c.png)

## 总结
目前得到App后端服务中，80%以上项目、90%以上业务流量运行在Kubernetes管理的容器基础设施之上，日均发布近百次。通过容器的落地，简化了环境管理，统一了发布流程，屏蔽发布细节，基础设施只关注服务运行状态，开放了运维能力，打通了开发和运维间屏障。得益于Kubernetes的优秀设计，现在运维人员不需要关注节点用途，运行环境配置等功能，每个节点都只是资源池的一部分，只需关注集群资源水位，管理工作只剩下增减节点。通过资源配额，对计算资源进行再分配，从而保证和限制了应用的资源需求，进而提升了资源的利用率。

未来我们将会Kubernetes和容器技术的特性进行混合云建设和落地，实现跨云基础环境下的流量调度、资源分配、伸缩等。同时精细化发布过程、设计多种可预期场景的弹性伸缩控制器降、强化资源交付效率，为研发人员赋能。

> **愿大家都能够落地感受容器化交付方式的便利，拥抱Kubernetes“云操作系统”，希望我们的经验对大家有所帮助。**
