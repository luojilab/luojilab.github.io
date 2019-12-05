title: 【八里庄技术沙龙-15 期】得到安卓客户端的工程架构实践
date: 2019-11-07
author: liushuo
toc: true
thumbnail: https://piccdn.luojilab.com/fe-oss/default/MTU3MzQ0NjI1NDI4.png
tag: 
  - 八里庄技术沙龙 
  - Android
categories: 
  - 八里庄技术沙龙

---

大家好，我是刘硕，来自得到安卓客户端。主要负责业务架构方向的工程效能提升相关工作。我们希望，通过对工程架构的改造升级，践行工程化方面的一些通用实践。使安卓团队在研发效率和研发体验上得到整体提升，提高app稳定性。

最近两年，我们在工程架构方面有了一些成果，主要围绕着工程架构，开发架构相关方面做了很多工作，大概分为两部分内容：组件化和mvvm开发架构。

<!-- more -->

#### 组件化拆分
组件化的概念其实理解上很简单，所谓组件化，就是把一个功能完整的app拆分成多个子模块，每个子模块可以独立编译和运行，也可以将这些子模块任意组合成一个新的app，子模块之间不互相依赖，但可以相互交互。

##### 单体工程架构
一般app的开发早期，团队的重心并不在开发架构的选型上。主要也是因为早期的项目比较小，大家更关注
多快好省的完成任务，对于如何复用，如何解耦没有过度考虑。如下图是得到早期的工程架构。

随着版本迭代，app的功能越来越多，项目结构逐渐也演变成了一个庞大的单体工程，内部依赖错综复杂。

当然，这会带来很多很多问题。

第一、逻辑复杂，不易理解。想要熟悉掌握所有功能需要耗费大量的时间和精力，不仅如此，对于新人熟悉业务来说，也会给他们带来巨大的挑战。

第二、不同的业务功能耦合严重。导致面对一个修改，我们无法界定它的影响范围，牵一发，动全身。

第三、构建时间越来越长，降低了研发效率。

![Untitled Diagram (23).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_3087dd52ede277f8bd91fe9366be7b48.png) 

##### 单体工程组件化拆分
为了解决单体工程存在的这些问题，我们开始了工程的组件化改造。优先梳理各个业务模块，将不同业务模块
的代码和资源放到不同的业务子工程中，这些作为组件化工程中的业务组件。对于不同业务组件之间公用的代码和资源，下沉到基础子工程中，作为业务子工程的依赖。独立壳工程app，它的主要任务是负责组件的集成打包，所以尽量不要包含业务逻辑。组件化拆分如下图：

![Untitled Diagram (26).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_e98c87a323e87aa1e305202c87a3fc6d.png)

##### 组件间通信
工程组件化拆分结束后，我们遇到的另一个问题是如何进行组件间的方法调用，因为业务组件之间是完全解耦的，所以不能简单的通过引用的方式进行调用。

![Untitled Diagram (21).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_d8506a2f2d326d503d3d70de223f499d.png)

我们为业务组件之间通信提供了两种方式。页面路由和服务调用。从实现上来说，这两种方式都是遵循协议下沉的原则，将服务协议下沉到通信组件。

app启动后，组件工程分别将自己提供的服务和页面路由注册到通信组件。如果某个组件
希望调用其他组件提供的服务或路由到其他组件，就可以通过查询通信组件中的注册信息，完成组件间通信的
任务。

##### 组件单独调试运行
如果我们集成所有组件构建项目，时间会很长，平均大概10分钟左右，为了解决这个问题。我们提出了单组件调试
运行的方案。

![Untitled Diagram (27).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_f765bb0788e2e7f8b36078850af3b63a.png)

通过自定义组件构建脚本，每个业务组件都可以作为壳工程独立运行。并且可以集成其他依赖的组件并进行组件间通信。通过这种方式，编译效率得到了很大的提升，我们运行单组件工程，构建时间只需要40秒左右。

##### 组件化2.0
组件化用了大概一年时间，我们遇到了新的问题。大家有时希望全组件集成运行app，正如之前的解决方案，我们并没有针对这种情况的优化手段，所以就导致了每天还是浪费了很多时间在编译构建上。

其实，gradle在构建项目时，确实是支持增量编译的，但有时改动一个文件，会导致项目构建时间超过10分钟。我们需要尽快解决这个问题，不然每天团队所有人在构建上浪费的总时间，我粗略的算了下  7个人*5次*10分钟，超过5个小时，还是比较吓人的。

##### 全组件集成构建流程分析
下图简要的描述了我们集成构建app的时候需要执行的核心环节。通过分析，我们发现构建时的一些问题。每次编译项目,所有组件工程都会重新执行同样的编译构建流程，所以，如果组件集成可以直接使用aar的方式，那么这些执行的重复构建
就可以节省下来。编译时间上会有很大的提升

![Untitled Diagram (33).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_c19f37fdc74d41101463a4578f204ae5.png)

##### 组件打包aar
所以，顺着前面的思路，我们构建了一套完整的组件打包体系。

![Untitled Diagram (38).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_16304fe99e16eee44a9ea846855bb2f7.png)

ci 负责实时监控git仓库代码变动,当开发人员提交了代码，CI 自动开始执行组件打包脚本。打包脚本会分析出所有包含代码变动的组件，并计算出组件对应的版本和maven仓库信息。使用这些信息，执行每个组件的gradle打包任务，并将打包成功后的产出物 aar 上传到组件仓库。

##### 组件化2.0集成构建
组件有了aar的管理方式，我们的全组件集成构建逻辑就可以进一步得到升级。

![Untitled Diagram (36).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_d616f84d06a018d87976dd1aacba49f5.png)

执行全组件集成构建时，首先解析工程下的组件化配置文件。该配置文件中明确标明了某个组件的依赖方式是aar还是源码，及aar的依赖版本和仓库信息。然后构建脚本就会根据这些信息，灵活的配置组件依赖并集成构建app。

使用aar的集成方式，避免了组件的再次编译，全量编译时间从之前的10分钟降低为现在的2分半左右

#### MVVM 开发架构
这部分，主要包含一些我们在选型开发架构上的心得和实践。如下图描述，工程组件化架构搭建完成后，我们的编译效率和项目管理方式得到了很大的改进。

但是，组件内的业务开发还在采用比较粗放的模式。Controller作为业务功能组织的核心，完成了大概80%的工作量，内部耦合十分严重，极大的限制了代码的复用能力，导致研发效率底下。

![Untitled Diagram (49).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_500734ffdd71d031dfa89c4025f85953.png)

##### MVC VS MVP

![Untitled Diagram (83).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_83b0b621659d7633d42742eb8176d09f.png)

为了解决目前以Controller为核心的开发模式带来的代码复用问题,我们对比了常见的三种MVX 架构。

其中，MVC 是开发gui应用程序的经典架构。但是，由于Controller直接持有了View的引用并使用这些引用组织展现逻辑，导致展现逻辑不能很好的被复用。

MVP的出现很好的 解决了MVC中展现逻辑不易复用的问题。MVP中展现交互逻辑完全由Presenter负责，并通过View接口与View通信。

但是MVP也存在一些问题。展现逻辑的复用粒度由View接口的力度决定，而且，当展现逻辑非常复杂，
就会造成Presenter与View联系过于紧密，限制了复用能力。

##### MVP VS MVVM

![Untitled Diagram (54).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_bc0b779af1c6bc2aee13b3bfbac19920.png)

相对于MVP，MVVM中展现逻辑的复用更为彻底。
MVVM 中创新的提出了抽象View的概念 ViewModel，ViewModel封装了View的一切状态和行为，但与具体的显示框架，布局规则没有任何关系。

这就使得ViewModel可以满足几乎任何场景下的被复用需求。基于传统的MVVM概念和google 推出的AAC 架构组件，我们开发了一套更符合自己实际情况的MVVM方案.

下面开始详细的介绍我们的MVVM 实践。

##### MVVM 中的依赖原则

MVVM 遵循单向依赖原则，依赖关系从上向下 依次为 View 依赖 ViewModel,ViewModel依赖 Model，不允许跨层依赖。这样的好处是可以使调用依赖关系更加清晰。

沿着依赖方向的通信方式以直接方法调用为主。由于不能违背依赖原则，从下向上的通信主要借助观察者模式实现，上层注册观察者，下层需要通信时，触发观察者回调。

![Untitled Diagram (57).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_908f888db681263a44229d3f899f1fe1.png)

##### MVVM 中的类层次
如下图，MVVM 中View ViewModel Model 都有自己的类层次结构。

其中，View 需要 承载 布局渲染等逻辑，所以Activity， Fragment，ViewHolder 及其子类属于View的角色范畴。

ViewModel作为View的抽象表示，分别针对页面和列表item提供了不同的子类实现。

Model中BaseModel类主要封装了网络库相关的方法调用，具体子类可以根据不同场景，实现不同的需求。

![Untitled Diagram (81).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_5e0f7c3e0e36df4358644bbf3e9977cd.png)

##### MVVM 在首页的实践

![Untitled Diagram (82).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_f61fa909899fbfa3a6f4efdafb5c8d3f.png)

首页算是得到app中比较特殊的页面。最外层结构是一个列表，列表中每个item 独立请求需要显示的业务数据。

我们在使用mvvm架构整个页面的过程中，确实遇到了一些问题。
这些问题，大概包含了三个方面的内容。

问题1：如何复用逻辑
面向对象开发中，复用的主要手段包括组合还有继承

那么，mvvm中，展现逻辑和数据逻辑的复用，也不外乎这两种手段。

例如，得到app 首页中 推荐课程，推荐听书都包含 负反馈和底部推荐标签功能，我们将这两个功能抽象到TagsItemVM 中，课程，听书VM分别继承TagsItemVM，这样就可以非常容易的实现这两个展现交互逻辑的复用。

![Untitled Diagram (86).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_4aea96b8fbf0c310c141478c81bbf034.png)

问题2：ViewModel如何感知View的生命周期变化

![Untitled Diagram (95).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_fce79cfa2bb995e19bc87177755d7170.png)

在mvvm中，View直接持有ViewModel的引用，所以，当View的生命周期发生变化，ViewModel对应的生命周期函数会立即被调用。通过这种方式，我们确保ViewModel与View的
生命周期能够保持同步

但由于页面Activity，Fragment和列表ItemViewHolder具有不同的生命周期形式，所以他们对应的ViewModel会有不同的生命周期回调。

ViewModel内部使用一个对象维护自身的生命周期状态，当ViewModel与View绑定后，ViewModel的生命周期 活跃，当ViewModel与View解除绑定后，ViewModel的生命周期不活跃。

此外，ViewModel生命周期的活跃状态受其parent ViewModel的生命周期影响，当parent ViewModel 不活跃，当前ViewModel的生命周期同样已经不活跃。

通过感知ViewModel生命周期的活跃状态,在生命周期不活跃时，执行某些资源的清理操作，可以有效防止内存泄露。

![Untitled Diagram (75).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_8a5dad97b4785a5b386659f6c81ed694.png)code block

问题3：ViewModel之间如何互相通信

![Untitled Diagram (61).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_ed373b1658fa4ff4ba9c914acfa5c5fc.png)

ViewModel之间通信主要依赖于LifecycleBus，这是一种特殊的事件总线。

ViewModel不活跃时，由于会断开与总线的链接，所以不会收到总线上的事件。

这样的设计主要考虑到 viewmodel 已经不在与View有绑定关系，ViewModel继续关注View中的事件通知是没有意义的，还可能带来其他未知的问题。

这中方案还带来了另外的好处，使event的派发效率更好，因为事件只会派发到活跃的ViewModel

##### 消除模板代码，简化开发

![Untitled Diagram (78).png](https://luoji-img.oss-cn-beijing.aliyuncs.com/fe/blog/20191110/upload_735be772fe5abd4942e14d8d813f5c9e.png)

如上图，传统的MVVM实现中，如果我们希望实现一个列表效果，至少需要新创建四个文件，view adapter，item view hodler，item view model，layout file。但是，view adapter和view holder中主要是一些模板代码，几乎没有有效的业务逻辑。

所以，我们为了解决这个问题，在MVVM framework中提供三个基础设施类，通用的view adapter，通用的view holder，bindItemVH注解。

这样，我们再实现同样的列表效果，只需要创建两个文件 item view model和layout file。

然后使用注解关联这两个文件。运行时，通用的view adapter 根据注解指定的关联关系，就可以将相关的ui渲染到屏幕上。

##### 我们的收获
目前我们已经上线首页，已购的mvvm改造，消息中心，问答，搜索，课程的mvvm方案也已经完成。

通过mvvm开发架构的升级，我们的程序结构更加清晰，代码可读性更高，通过运行时注解的支持，彻底消除了不必要的模板代码,使我们的开发更加顺畅。

#### 未来规划
未来，我们想尝试的方向有组件平台化，插件化，跨平台，希望通过这些手段，进一步提升团队协作效能，提高app研发效率，和用户体验

以上就是我们最近两年在工程架构上的努力，谢谢大家
