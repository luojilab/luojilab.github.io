title: 【八里庄技术沙龙-12 期】如何从零实现一个高性能的API网关
date: 2019-08-14
author: yanbo
toc: true
thumbnail: https://piccdn.luojilab.com/fe-oss/default/MTU2NzE1Mjk5Mjg1.jpeg
tag: 
  - 八里庄技术沙龙 
categories: 
  - 八里庄技术沙龙

---

## API网关是什么？ 
要回答这个问题我们需要先了解下我们得到的架构变迁。
![单体应用架构](https://piccdn.luojilab.com/fe/blog/20190918/upload_7f178cd4397fa1e5d7c8a7077e5dd880.png)

我们公司最早的时候都是PHP实现的单体应用，比如生活作风的H5商城，得到的V3。这张图就是我们得到的早期架构，当时所有的业务逻辑实现全部在V3当中，然后DCAPI封装了与数据库的交互。这就是一个典型的单体应用架构。


<!-- more -->

![微服务无服务架构g](https://piccdn.luojilab.com/fe/blog/20190918/upload_c45a152965f8ec10d45b2f67fad7220d.png)

然后到17年的时候，随着公司人员越来越多以及微服务的兴起，我们也开始进行服务化的改造。但是在进行服务化的时候首先面临的一个问题就是：**当我们把一个单体应用拆成众多微服务之后，每一个服务如何与客户端进行通信？**原来客户端只需要和V3进行对接，现在难道要让客户端分别与这么多服务进行对接么？ 这显然是不可行的。

所以，实际上这时候我们与所有进行微服务落地工作的团队一样面临微服务的一些痛点。那么解决这些痛点的方式，一般业界通用的是引入一个叫做API网关的组件。也就是这样的微服务架构。

![微服务架构](https://piccdn.luojilab.com/fe/blog/20190918/upload_b997a5f989bc093c0751ac3d37a38083.png)

现在我们就可以来回答网关是什么？
>API网关一般作为系统与外界联通的入口，在微服务架构中，所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。

### 网关的核心功能
- 路由转发
- 路由重写
- 访问控制
- 流量控制
- 负载均衡
- 健康检查
- 服务发现
- 熔断降级

### 网关的设计目标
- **具备Cloud-Native特性**
- **部署支持无限水平扩展**
- **高性能，高可用**
- **开箱即用，功能易扩展**
- **支持多机房部署**

### 网关的抽象模型
1. 应用：一组路由的集合，抽象为应用，一般为一个业务或一条业务线
2. 路由：归属于应用，定义了可以通过网关访问的路由规则及限流策略
3. 服务：对应一个真实的后端服务，可以绑定在多个应用的不同路由上
4. 节点：每个节点即一个后端服务节点，支持监控检查及负载均衡

这四个模型的关系如下：
![内部模型关系](https://piccdn.luojilab.com/fe/blog/20190918/upload_1873872e716aca434fb201ff7183a82a.png)

### 网关的高可用方案

![得到网关高可用方案](https://piccdn.luojilab.com/fe/blog/20190918/upload_513e3adb0aa4a3362b1e651bc71aa15d.png)

我们采用etcd来存储配置，通过ETCD的高可用来保证网关集群的高可用和水平无限扩展。

### 网关的易扩展设计
![image.png](https://piccdn.luojilab.com/fe/blog/20190918/upload_7d2ee18094781cca3e54b43c991ad24b.png)

我们还是希望整个系统能够具备一定的扩展能力，为了避免调度系统过于复杂，所以我们想采用类似Gin框架中间件的形式来实现易扩展。这样每个请求进来之后会穿过所有的中间件，中间件也就可以在这个过程中对其进行操作控制。
![image.png](https://piccdn.luojilab.com/fe/blog/20190918/upload_3ac9277f62bcaaa58a64383ca8c3e97f.png)

Gin框架中间件的巧妙之处主要在于通过Context的Next方法进行自调用实现了一个拦截器。这样，不论在中间件中是否调用了Next都不会影响中间件的执行顺序。如果看代码不好理解，可以看下面这两张图。

![中间件内部不调用Next](https://piccdn.luojilab.com/fe/blog/20190918/upload_478c92e973f783ae8704f8ab3a05183b.png)

![中间件内部调用Next](https://piccdn.luojilab.com/fe/blog/20190918/upload_9b0b2ae60182518bf450b6d0f80d6b33.png)

### 网关的内部组件
![内部组件调用关系](https://piccdn.luojilab.com/fe/blog/20190918/upload_995dd1ac9e9b5c3da7026cd8848e76a8.png)

最终，在易扩展、高可用的基础上，我们在系统内部实现了上图这样的内部结构。
1. **APIServer负责提供配置变更的接口**
2. **当它收到配置变更请求时将配置数据写入ETCD**
3. **由Watcher组件对ETCD进行watch并把最新的数据映射到Model上，同时编译中间件的HandlerChain**
4. **代理服务器负责接收需要转发的请求，开始执行HandlerChain**


## Golang高性能系统实战
** 工具：**
- Jmeter：用于发起压测流量
- runtime/pprof：采集程序（非 Server）的运行数据进行分析
- net/http/pprof：采集 HTTP Server 的运行时数据进行分析
- go tool pprof bin/server http://10.2.0.2:8088/debug/pprof/profile
- go-torch -u http://10.2.0.2:8088/debug/pprof/profile -f slice.svg

** 环境：**
- 压测机：8C16G
- API网关：4C8G
- 系统：Centos7

### 反向代理
![Go反向代理服务器](https://piccdn.luojilab.com/fe/blog/20190918/upload_0afe549f4844c4704d7951492b90289e.png)
反向代理是网关的基础功能，所以我们首先就对Golang实现的反向代理做了压测，如上图采用Golang官方的net.http包实现。经过多次压测验证，在4C8G的服务器上只能压到19000+，将近两万QPS。这结果令我们很愕然，因为大家都知道Golang的并发处理能力是不弱的，所以我们又单独对Golang实现的HTTPServer进行了压测，实现代码如下图。

![GoHTTP服务器](https://piccdn.luojilab.com/fe/blog/20190918/upload_85bb90f0a02994f08c77eb3007f00921.png)

最终压出了88000+的QPS，这结果才令人满意。那么我们就分析，Golang的http.Client性能可能远低于Server。我们都知道Golang社区除了官方的net.http包还有一个fasthttp，那么能不能使用fasthttp来替代反向代理中的Client部分呢。见过反复验证，也是可以的。最终我们压出了这两张火焰图。

![nethttp反向代理：19343QPS](https://piccdn.luojilab.com/fe/blog/20190918/upload_0cda12b0b3b1fd0a5254999bcda1e83e.png)

![fasthttp反向代理：45654QPS](https://piccdn.luojilab.com/fe/blog/20190918/upload_6442a094e4f719b502bf6801d1432f09.png)

### 正则匹配
![路由重写正则测试](https://piccdn.luojilab.com/fe/blog/20190918/upload_dcc63629c6c545d8fd4848f5dc15ede3.png)

![路由重写正则测试结果](https://piccdn.luojilab.com/fe/blog/20190918/upload_bfa7789824669bf871ebccae2416cd96.png)

**Tips: 在热点代码中避免使用正则，如果无法避免，那么一定要进行预编译。**

### 字符串拼接
![多字符串拼接测试](https://piccdn.luojilab.com/fe/blog/20190918/upload_4205efe3bc8bd61b8a7ee1265add951b.png)

![多字符串拼接测试结果](https://piccdn.luojilab.com/fe/blog/20190918/upload_99a70ebcb2591dbea9483de304af868e.png)

![单字符拼接测试](https://piccdn.luojilab.com/fe/blog/20190918/upload_c43fc59b5ab8ac26c534aa1bb712248f.png)

![单字符拼接测试结果](https://piccdn.luojilab.com/fe/blog/20190918/upload_9b46064a7b1474f084617e04e1d7f5b4.png)

**Tips1：单次调用时，操作符+ > strings.Join >= bytes.Buffer > fmt.Sprintf**

**Tips2：多次调用时，bytes.Buffer >= strings.Join > 操作符+ > fmt.Sprintf**

### 频繁创建对象 Vs 复用对象
![频繁创建对象](https://piccdn.luojilab.com/fe/blog/20190918/upload_f7890f002b341d5dc9cec25c880baca9.png)

![使用sync.Pool复用对象后](https://piccdn.luojilab.com/fe/blog/20190918/upload_089cb3989e0df932465c69b2f0f4de67.png)

可以看到明显的提升，所以在热点代码中如果有创建对象的操作，要尽量进行复用。

### Slice查询 Vs Map查询

![Slice和Map查询对比测试](https://piccdn.luojilab.com/fe/blog/20190918/upload_70d8f685f8c6286a961eb21e23fe8290.png)

 ![Slice和Map查询对比测试结果](https://piccdn.luojilab.com/fe/blog/20190918/upload_51c78c45274095c6fb8e2fe982a7c124.png)


Map查询的时间复杂度为O1，而Slice查询的时间复杂度为On，我们直观上理解Map肯定是比Slice快的。但是实际的情况并不是那么绝对，可以看到上面的例子中，当成员数量为10时我们遍历整个slice的速度都比map的一次查询快。经过研究map底层代码，我们发现这是因为map底层还有hash的操作。

所以，最终经过我们测试，在不考虑key大小的情况下，成员数量小于25时slice的性能要好于map。


### 高性能总结
- **避免反射和锁的使用**
- **避免创建过多的对象（避免GC）**
- **尽量复用已经创建的对象（sync.Pool）**
- **避免进行[]byte和string的转换**
- **设置GOGC用内存换CPU时间**
- **对于精度不高的时间自己实现时钟**
- **数据量较少时用slice替代map**

