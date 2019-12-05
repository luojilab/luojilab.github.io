title: 如何打造一套Vue组件库
date: 2019-08-26
author: zouyawei
toc: true
thumbnail: https://piccdn.luojilab.com/fe-oss/default/MTU2NzM5MjkwMTYy.jpeg
tag: 
  - 前端
categories: 
  - 前端

---

# 开篇

组件库能帮我们节省开发精力，无需所有东西都从头开始去做，通过一个个小组件拼接起来，就得到了我们想要的最终页面。在日常开发中如果没有特定的一些业务需求，使用组件库进行开发无疑是更便捷高效，而且质量也相对更高的方案。

目前的开源组件库有很多，不管是react还是vue的体系里都有很多非常优秀的组件库，比如我经常使用的就有elementui和iview。当然也还有其他的一些组件库，他们的本质其实都是为了节省重复造基础组件这一轮子的过程。也有的公司可能会对自己公司的产品有特别的需求，不太愿意使用开源的组件库的样式，或者自己有一些公司内部的业务项目需要用到，但开源项目无法满足的组件需要沉淀下来的时候，自建一套组件库就成为了一个作为业务驱动所需要的项目。

<!-- more -->

本文会从 ”准备“ 和 ”实践“ 两个阶段来阐述，一步步完成一个组件库的打造。大致内容如下：

1. **准备**：主要讲了搭建组件库之前我们需要先提及一下一些基础知识，为实践阶段做铺垫。
2. **实践**：有了一些基本概念，咱们就直接通过一个实践案例来动手搭建一套基础的组件库。从做的过程中去感受组件库的设计。

希望通过本文的分享以及包含的一个简单的 **实际操作案例**，能让你从组件库使用者的角色向组件库创造者的角色迈进那么一小步，在日常使用组件库的时候心里有个底，那我的目的也就达到了。

我们的案例地址是：https://arronkler.github.io/lime-ui/ 

对应的 repo也就是：https://github.com/arronKler/lime-ui

# 准备 ：打造组件库之前你应该知道些什么？

这一个章节主要是想先解析清楚一些在组件库的建立中会用到的一些平时在业务概念中很少去关注的概念。我会分为工程和组件两个方面来阐述，把我所知道的一些其中的技巧和坑点都交付出来，以帮助我们在实际去做的过程中可以有所准备。



## 项目：做一个组件库项目有哪些额外需要考虑的事？

做组件库项目和常规业务项目肯定还是有一些事情是我们业务项目不怎么需要，但是类库项目一般都会考虑的事，这一小节就是介绍说明一下，那些我们在做组件库的过程中需要额外考虑的事。



### 组件测试

很多开发者平时业务项目都比较赶，然后就是一般业务项目中都不怎么写测试脚本。但在做一个组件库项目的过程中，最好还是有对应的组件测试的脚本。至少有两点好处：

1. 自动化测试你写的组件的功能特性
2. 改动代码不用担心会影响之前的使用者。（测试脚本会告诉你有没有出现未预料到的影响）

对于类库型项目，我觉得第二点好处还是很重要的，这才能保证你在不断推进项目升级迭代的过程中，确保不会出现影响已经在用你所创造的类库的那些人，毕竟你要是升级一次让他的项目出现大问题，那可真保不准别人饭碗都能丢。（就像之前的antd的圣诞节雪花事件一样）

由于我们是要写vue的组件库，这里推荐的测试工具集是 `vue-test-utils` 这套工具，https://vue-test-utils.vuejs.org/zh/ 。其中提供的各种测试函数和方法都能很好的满足我们的测试需要。具体的安装使用可以参见它的文档。

我们这里主要想提的是 **组件测试到底要测什么？**

我们这里给到一张很直观的图，看到这张图其实你应该也清楚了这个问题的答案

![IMG](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MDIzNDE2.png)

这张图来自视频 https://www.youtube.com/watch?v=OIpfWTThrK8 ，也是vue-test-util推荐的一个非常棒的演讲，想要具体了解可以进去看一下。

所以回过头来，组件测试，实际需要我们不仅仅作为创造者的角度对组件的功能特性进行测试。更要从使用者的角度来看，把组件当做一个“黑盒子”，我们能给到它的是用户的交互行为、props数据等，这个“黑盒子”也会对应的反馈出一定的事件和渲染的视图可以被使用者所捕获和观察。通过对这些位置的检查，我们就能获知一个组件的行为是否如我们所愿的去进行着，确保它的行为一定是一致不出幺蛾子的。

另外还想提的一点偏的话题就是 **契约精神**。作为组件的使用者，我使用你的组件，等于咱们签订一个契约，这个组件的所有行为应该是和你描述的是一致的，不会出现第三种意料之外的可能。毕竟对于企业项目来说，我们不喜欢surprise。antd的彩蛋事件也是给各位都提个醒，咱们搞技术可以这么玩也挺有创意，但是这种公用类库，特别是企业使用的也比较多的，还是把创意收一收，讲究契约，不讲surprise。就算是自家企业内部使用的组件库，除非是业务上的人都是认可的，否则也不要做这种危险试探。

好的组件测试也是能够帮助我们识别出那些我们有意或无意创造的surprise，有意的咱就不说了，就怕是那种无意中出现的surprise那就比较要命了，所以写好组件测试还是挺有必要的。



### 文档生成

一般来说，我们做一个类库项目都会有对应的说明文档的，有的项目一个README.md 的文档就够了，有的可能需要在来几个 Markdown的文档。对于组件库这一类的项目来说，我们可以用文档工具来辅助直接生成文档。这里推荐 vuepress ，可以快速帮我们完成组件库文档的建设。(https://vuepress.vuejs.org/zh/guide/)

vuepress是一个文档生成工具，默认的样式和vue官方文档几乎是一致的，因为创造它的初衷就是想为vue和相关的子项目提供文档支持。它内置了 Markdown的扩展，写文档的时候就是用 markdown来写，最让人省心的是<u>你可以直接在 Markdown 文件中使用Vue组件</u>，意味着我们的组件库中写的一个个组件，可以直接放到文档里去用，展示组件的实际运行效果。 我们的案例网站也就是通过vuepress来写的，生成静态网站后，用 `gh-pages` 直接部署到github上。

vuepress更好的一点在于你可以自定义其webpack配置和主题，意味着你可以让你自己的文档站点在开发阶段有更多的功能特性的支持，同时可以把站点风格改成自己的一套主题风格。这就无需我们重头开始去做一套了，对于咱们想要快速完成组件库文档建设这一需求来说，还是挺有效的。

不过这只是咱们要做的事情的一个辅助性的东西，所以具体的使用咱们在实践阶段再说明，这里就不赘述了。



### 自定义主题

自定义主题的功能对于一个开源类库来说肯定还是挺有好处的，这样使用者就可以自己使用组件库的功能而在界面设计上使用自己的设计风格。其实大部分组件库的功能设计都是挺好挺完善的，所以一般来说中小型公司即使想要实现自己的一套组件风格的东西，直接使用开源类库如 element、iview或者基于react的Antd 所提供的功能和交互逻辑，然后在其上进行主题定制基本就满足需求了（除非你家设计师很有想法。。。）。

自定义主题的功能一般的使用方式是这样的

1. 通过主题生成工具。（制作者需要单独做一个工具）
2. 引入关键主题文件，覆盖主题变量。(这种方式一般都需要适配制作者所使用的css预处理器）

对于第一种方式往往都是组件库的制作者通过把生成组件样式的那一套东西做成一个工具，然后提供给使用者去根据自己的需要来调整，最后生成一套特定的样式文件，引入使用。

第二种方式，**作为使用者来说，你主要做的其实是覆盖了组件库中的一些主题变量**，因为具体的组件的样式文件不是写死的固定样式值，而是使用了定义好的变量，所以你的自定义主题就生效了。但是这也会引入一个小问题就是你必须适配组件库的创造者所使用的样式预处理器，比如你用iview，那你的项目就要能解析Less文件，你用ElementUI，你的项目就必须可以解析SCSS。

其实对于第一种方式也主要是以调整主题变量为主。所以当咱们自己要做一套组件库的时候，不难看出，一个核心点就是需要**把主题变量文件和样式文件拆开来**，后面的就简单了。



### webpack打包

类库项目的构建这里提两点：

1. **暴露入口**
2. **外部化依赖**

先谈第一点 “暴露接口”。业务项目中，我们的整个项目通过webpack或其他打包工具打包成一个或多个bundle文件，这些文件被浏览器载入后就会直接运行。但是一个类库项目往往都不是单独运行的，而是通过暴露一个 “入口”，然我在业务项目中去调用它。 在webpack配置文件里，可以通过定义 `output` 中的 `library` 和 `libraryTarget` 来控制我们要暴露的一个 “入口变量” ，以及我们要构建的目标代码。

这一点可以详细参考webpack官方文档: https://webpack.js.org/configuration/output/#outputlibrarytarget

```javascript
module.exports = {
  // other config
	output: {
    library: "MyLibName",
    libraryTarget: "umd",
    umdNamedDefine: true
  }
}
```

再说一下 “外部化依赖”，我们做一个vue组件库项目的时候，我们的组件都是依赖于vue的，当我们组件库项目中的某个地方引入了vue，那么打包的时候vue的运行时也是会被一块儿打包进入最终的组件库bundle文件的。这样的问题在于，我们的vue组件库是被vue项目使用的，那么项目中已经有运行时了，我们就没必要在组件库中加入运行时，这样会多增加组件库bundle的体积。使用webpack的 `externals`可以将vue依赖 "外部化"。

```javascript
module.exports = {
	// other config
	externals: {
    vue: {
      root: 'Vue',
      commonjs: 'vue',
      commonjs2: 'vue',
      amd: 'vue'
    }
  }
}
```



### 按需加载

组件库的按需加载功能还是很实用的， 这样可以避免我们在使用组件库的过程中把所有的用到和没用到的内容都打包到业务代码中去，导致最后的bundle文件过大影响用户体验。

在业务项目中我们的按需加载都是把需要按需加载的地方单独生成为一个chunk，然后浏览器运行我们的打包代码的时候发现我们需要这一块儿资源了，再发起请求获取到对应的所需代码。

在组件库里边，我们就需要改变一下引入的方式，比如一开始我们引入一个组件库的时候是直接将组件库和样式全部引入的。如下面这样

```javascript
import LimeUI from 'lime-ui' // 引入组件库
import 'lime-ui/styles/index.css' // 引入整个组件库的样式文件

Vue.use(LimeUI)
```

那么，换成手动的按需加载的方式就是

```javascript
import { Button } from 'lime-ui' // 引入button组件
import 'lime-ui/styles/button.css' // 引入button的样式

Vue.component('l-button', Button) // 注册组件
```

这种方式的确是按需引入的，但也一个不舒服的地方就是每次我们引入的时候都需要手动的引入组件和样式。一般来说一个项目里面用到的组件少说也有十多个，这就比较麻烦了。组件库是怎么解决这个问题的呢？

通过babel插件的方式，将引入组件库和组件样式的模式自动化，比如antd、antd-mobile、material-ui都在使用的`babel-plugin-import`、还有ElementUI使用的 `babel-plugin-component`。在业务项目中配置好babel插件之后，它内部就可以给你做一个这样的转换（这里以 babel-plugin-component）

```javascript
// 原始代码
import { Button } from 'components'
 

// 转换代码
var button = require('components/lib/button')
require('components/lib/button/style.css')
```

OK，那既然代码可以做这样的转换的话，其实我们所要做的一点就是在我们打造组件库的时候，把我们的组件库的打包代码放到对应的文件目录结构之下就可以了。使用者可以选择手动载入组件，也可以使用babel插件的方式优化这一步骤。



babel-plugin-component 文档： https://www.npmjs.com/package/babel-plugin-component

babel-pluigin-import 文档: https://www.npmjs.com/package/babel-plugin-import



## 组件：比起日常的组件设计，做组件库你还需要知道些什么？

做组件库中的组件的技巧和在项目中用到的还是有一些区别的，这一小节就是告诉大家，组件库中的组件设计，我们还应该知道哪些必要的知识内容。

### 组件通信：除了上下级之间进行数据通信，还有什么？	

我们常规用到的组件通信的方法就是通过 `props` 和 `$emit` 来进行父组件和子组件之间的数据传递，如下面的示意图中展示的那样：父组件通过 `props` 将数据给子组件、子组件通过 `$emit ` 将数据传递给父组件，顶多通过`eventBus`或`Vuex`来达到任意组件之间数据的相互通信。这些方法在常规的业务开发过程中是比较有效的，但是在组件库的开发过程中就显得有点力不从心了，主要的问题在于： <u>**如何处理跨级组件之间的数据通信呢？**</u>

![IMG](https://blog-1257601889.cos.ap-shanghai.myqcloud.com/vue/attrs/vue.png?ynotemdtimestamp=1551245782807)

如果在日常项目中，我们当然可以使用像 `vuex` 这样的将组件数据直接 ”外包“ 出去的方式来实现数据的跨级访问，但是`vuex` 始终是一个外部依赖项，组件库的设计肯定是不能让这种强依赖存在的。下面我们就来说说两个在组件库项目中我们会用到的数据通信方式。

#### 内置的provide/inject

**<u>provide/inject 是vue自带的可以跨级从子组件中获取父级组件数据的一套方案。</u>** 这一对东西类似于react里面的 `Context` ，都是为了处理跨级组件数据传递的问题。

使用的时候，在子组件中的 inject 处声明需要注入的数据，然后在父级组件中的某个含有对应数据的地方，提供子级组件所需要的数据。不管他们之间跨越了多少个组件，子级组件都能获取到对应的数据。(参考下面的伪代码例子)

```javascript
// 引用关系 CompA --> CompB --> CompC --> ... --> ChildComp

// CompA.vue
export default {
  provide: {
    theme: 'dark'
  }
}

// CompB.vue
// CompC.vue
// ... 

// ChildComp.vue
export default {
  inject: ['theme'],
	mounted() {
    console.log(this.theme) // 打印结果: dark
  }
}
```

不过provide/inject的方式主要是子组件从父级组件中跨级获取到它的状态，却不能完美的解决以下问题：

1. 子级组件跨级传递数据到父级组件
2. 父级组件跨级传递数据到子级组件



#### 派发和广播: 自制dispatch和broadcast功能

**<u>dispatch和broadcast可以用来做父子级组件之间跨级通信</u>**。在vue1.x里面是有dispatch和broadcast功能的，不过在vue2.x中被取消掉了。这里可以参考一下下面链接给出的v1.x中的内容。

> dispatch文档（v1.x）：https://v1.vuejs.org/api/#vm-dispatch
>
> broadcast文档（v1.x）：https://v1.vuejs.org/api/#vm-broadcast

根据文档，我们得知

- dispatch会派发一个事件，这个事件首先在自己这个组件实例上去触发，然后会沿着父级链一级一级的往上冒泡，直到触发了某个父级中声明的对这个事件的监听器后就停止，除非是这个监听器返回了true。当然监听器也是可以通过回调函数获取到事件派发的时候传递的所有参数的。这一点很像我们在DOM中的事件冒泡机制，应该不难理解。

- 而broadcast就是会将事件广播到自己的所有子组件实例上，一层一层的往下走，因为组件树的原因，往下走的过程会遇到 “分叉”，也就可以看成是一条条的多个路径。事件沿着每一个子路径向下冒泡，每个路径上触发了监听器就停止，如果监听器返回的是true那就继续向下再传播。

简单总结一下。<u>**dispatch派发事件往上冒泡，broadcast广播事件往下散播，遇到处理对应事件的监听器就处理，监听器没有返回true就停止**</u>

需要注意的是，这里的派发和广播事件都是 **跨层级的** , 而且可以携带参数，那也就意味着可以**跨层级进行数据通信**。

![IMG](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MDIzMzkx.png)

由于dispatch和broadcast在vue2.x中取消了，所以我们这里可以自己写一个，然后通过mixin的方式混入到需要使用到跨级组件通信的组件中。

方法内容其实很简单，这里就直接列代码

```javascript
// 参考自iview的实现
function broadcast(componentName, eventName, params) {
  this.$children.forEach(child => {
    const name = child.$options.name;

    if (name === componentName) {
      child.$emit.apply(child, [eventName].concat(params));
    } else {
      broadcast.apply(child, [componentName, eventName].concat([params]));
    }
  });
}
export default {
  methods: {
    dispatch(componentName, eventName, params) {
      let parent = this.$parent || this.$root;
      let name = parent.$options.name;

      while (parent && (!name || name !== componentName)) {
        parent = parent.$parent;

        if (parent) {
          name = parent.$options.name;
        }
      }
      if (parent) {
        parent.$emit.apply(parent, [eventName].concat(params));
      }
    },
    broadcast(componentName, eventName, params) {
      broadcast.call(this, componentName, eventName, params);
    }
  }
};

```

其实这里的实现和vue1.x中的实现还是有一定的区别的：

1. **dispatch没有事件冒泡。找到哪个就直接执行**
2. **设定了一个name参数，只针对特定name的组件触发事件**

其实看懂了这里的代码，你就应该可以举一反三想出 **找寻任何一个组件的方法了，不管是向上还是向下找，无非就是循环遍历和迭代处理，直到目标组件出现，然后调用它。** 派发和广播无非就是找到之后利用vue自带的事件机制来发布事件，然后在具体组件中监听该事件并处理。



### 渲染函数：它可以释放javascript的能力

首先我们回顾一下一个组件是如何从写代码到被转换成界面的。我们写vue单文件组件的时候一般会有template、script和style三部分，在打包的时候，vue-loader会将其中的template模板部分先编译成Vue实例中render选项所需要的构建视图的代码。在具体运行的时候，vue运行时会使用` $mount` 进行渲染，渲染好之后将其挂载到你提供的DOM节点下。

整个过程里面我们只日常关注最多的当然就是template的部分，但是template其实只是vue提供的一个语法糖，只是让我们写代码写起来跟写html一样轻松，降低刚入手vue的小伙伴的学习成本。React就没有提供template的语法糖，而是使用的JSX来降低写组件的复杂度。(vue能在react和angular两大框架的压力下异军突起，简洁易懂的模板语法是有一定促进作用的，毕竟看起来更简单)

通过上面我们回顾的内容，其实我们也发现了，**我们写的template，最终都是javascript**。这里template被编译之后，给到了 render这个渲染函数，在执行渲染的时候vue就会执行render中的操作来渲染我们的组件。

所以template是好，但 **如果你想要使用全部的javascript的能力，那就可以使用渲染函数**。

> 渲染函数&JSX (官方文档)：https://cn.vuejs.org/v2/guide/render-function.html 

日常写业务组件，我们用template就挺OK的，不过当遇到一些复杂情况，用 `写组件 --> 引入使用 --> 注册组件 --> 使用组件` 的方式就不好处理了，比如下面两种情况：

1. 通过代码动态渲染组件
2. 将组件渲染到其他位置

第一种情况是通过代码动态渲染组件，比如运营常常使用的活动h5页面，每个活动都不一样，每次要么都重新做一份，要么在原有的基础上修改。但是这种修改的页面结构调整是很大的，每次都会是破坏性的，和重做其实没区别。这样的话，每次活动无论内容如何，前端都要上手去写代码。但其实只需要在管理后台做一个活动编辑器，编辑器的内容直接转化为render函数的代码，然后通过配置下发到某个页面上，承载页拿到数据给到render函数执行渲染。这样就可以动态的根据管理后台配置的方式来渲染组件内容，每次的活动页，运营也可以通过编辑器自行生成。

第二种情况是要将组件渲染到不同位置。我们日常写业务组件基本就是写一个组件，在需要的拿来使用。如果你只是在template中把组件写进去，那你的组件的内容就都会作为当前组件的子组件进行渲染，所生成的DOM结构也是在当前的DOM结构之下的。知道render之后，其实我们可以新建vue实例，动态渲染之后，手动挂载到任意的DOM位置上去。

```javascript
import CompA from './CompA.vue'

let Instance = new Vue({
  render(h) {
    return h(CompA)
  }
})

let component = Instance.$mount() // 执行渲染
document.body.appendChild(component.$el) // 挂载到body元素下

```

我们使用的element里面的 `this.$message` 就用到了动态渲染，然后手动挂载到指定位置。




# 实践：做一遍你就会了

这里先贴上我们的github地址，各位可以在做的过程中对照着看。https://github.com/arronKler/lime-ui

## 建立一个工程化的项目

### 第一步，建立工程化结构

这里就不废话了，直接贴目录结构和解释

```bash
|- assets/   # 存放一些额外的资源文件，图片之类的
|- build/  # webpack打包配置
|- docs/  # 存放文档
	|- .vuepress  # vuepress配置目录
	|- component # 组件相关的文档放这里
	|- README.md # 静态首页
|- lib/  # 打包生成的文件放这里
	|- styles/ # 打包后的样式文件
|- src/ # 在这里写代码
	|- mixins/ # mixin文件
	|- packages/ # 各个组件，每个组件是一个子目录
	|- styles/ # 样式文件
		|- common/ # 公用的样式内容
		|- mixins/ # 复用的mixin
	|- utils  # 工具目录
	|- index.js  # 打包入口，组件的导出
|- test/  # 测试文件夹
	|- specs/  # 存放所有的测试用例
|- .npmignore
|- .gitignore
|- .babelrc
|- README.md
|- package.json
```

这里比较重要的目录就是我们的src目录，下面存放了我们的各个单一的组件和一套样式库，另外还有一些辅助的东西。我们写文档就是在 docs目录下去写。项目目录最外层都是些常规的配置内容，比如 `.npmignore` 和 `.gitignore` 这样的文件我们都是很常见的，所以我就不具体细说这一部分了，要是有一定疑惑可以直接参见github上的源码对照着看。

这里我们把需要使用到的类库文件也先建立好

在 src/mixins 下创建一个 emitter.js，写入如下内容，也就是我们的dispatch和broadcast的方法，之后的组件设计中会用到

```javascript
function broadcast(componentName, eventName, params) {
  this.$children.forEach(child => {
    const name = child.$options.name;

    if (name === componentName) {
      child.$emit.apply(child, [eventName].concat(params));
    } else {
      broadcast.apply(child, [componentName, eventName].concat([params]));
    }
  });
}
export default {
  methods: {
    dispatch(componentName, eventName, params) {
      let parent = this.$parent || this.$root;
      let name = parent.$options.name;

      while (parent && (!name || name !== componentName)) {
        parent = parent.$parent;

        if (parent) {
          name = parent.$options.name;
        }
      }
      if (parent) {
        parent.$emit.apply(parent, [eventName].concat(params));
      }
    },
    broadcast(componentName, eventName, params) {
      broadcast.call(this, componentName, eventName, params);
    }
  }
};
```

然后在 src/utils 下新建一个 assist.js 文件，写下辅助性的函数

```javascript
export function oneOf(value, validList) {
  for (let i = 0; i < validList.length; i++) {
    if (value === validList[i]) {
      return true;
    }
  }
  return false;
}
```

这两个地方都是之后会使用到的，如果你需要其他的辅助内容，也可以在这两个文件所在的目录下去建立。



###  第二步， 完善打包流程

目录建好了，那就该填充血肉了，要打包一个组件库项目，肯定是要先配置好我们的webpack，不然写了源码也没法跑起来。所以我们先定位到 build目录下，在build目录下先建立三个文件

- webpack.base.js 。存放基本的一些rules配置

- webpack.prod.js 。整个组件库的打包配置
- gen-style.js 。单独对样式进行打包

以下是具体的配置内容

```javascript
/* webpack.base.js */
const path = require('path');
const webpack = require('webpack');
const pkg = require('../package.json');
const VueLoaderPlugin = require('vue-loader/lib/plugin')

function resolve(dir) {
  return path.join(__dirname, '..', dir);
}

module.exports = {
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        options: {
          loaders: {
            css: [
              'vue-style-loader',
              {
                loader: 'css-loader',
                options: {
                  sourceMap: true,
                },
              },
            ],
            less: [
              'vue-style-loader',
              {
                loader: 'css-loader',
                options: {
                  sourceMap: true,
                },
              },
              {
                loader: 'less-loader',
                options: {
                  sourceMap: true,
                },
              },
            ],
          },
          postLoaders: {
            html: 'babel-loader?sourceMap'
          },
          sourceMap: true,
        }
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        options: {
          sourceMap: true,
        },
        exclude: /node_modules/,
      },
      {
        test: /\.css$/,
        loaders: [
          {
            loader: 'style-loader',
            options: {
              sourceMap: true,
            },
          },
          {
            loader: 'css-loader',
            options: {
              sourceMap: true,
            },
          }
        ]
      },
      {
        test: /\.less$/,
        loaders: [
          {
            loader: 'style-loader',
            options: {
              sourceMap: true,
            },
          },
          {
            loader: 'css-loader',
            options: {
              sourceMap: true,
            },
          },
          {
            loader: 'less-loader',
            options: {
              sourceMap: true,
            },
          },
        ]
      },
      {
        test: /\.scss$/,
        loaders: [
          {
            loader: 'style-loader',
            options: {
              sourceMap: true,
            },
          },
          {
            loader: 'css-loader',
            options: {
              sourceMap: true,
            },
          },
          {
            loader: 'sass-loader',
            options: {
              sourceMap: true,
            },
          },
        ]
      },
      {
        test: /\.(gif|jpg|png|woff|svg|eot|ttf)\??.*$/,
        loader: 'url-loader?limit=8192'
      }
    ]
  },
  resolve: {
    extensions: ['.js', '.vue'],
    alias: {
      'vue': 'vue/dist/vue.esm.js',
      '@': resolve('src')
    }
  },
  plugins: [
    new webpack.optimize.ModuleConcatenationPlugin(),
    new webpack.DefinePlugin({
      'process.env.VERSION': `'${pkg.version}'`
    }),
    new VueLoaderPlugin()
  ]
};
```

```javascript
/*  webpack.prod.js */
const path = require('path');
const webpack = require('webpack');
const merge = require('webpack-merge');
const webpackBaseConfig = require('./webpack.base.js');

process.env.NODE_ENV = 'production';

module.exports = merge(webpackBaseConfig, {
  devtool: 'source-map',
  mode: "production",
  entry: {
    main: path.resolve(__dirname, '../src/index.js')  // 将src下的index.js 作为入口点
  },
  output: {
    path: path.resolve(__dirname, '../lib'),
    publicPath: '/lib/',
    filename: 'lime-ui.min.js',  // 改成自己的类库名
    library: 'lime-ui', // 类库导出
    libraryTarget: 'umd',
    umdNamedDefine: true
  },
  externals: { // 外部化对vue的依赖
    vue: {
      root: 'Vue',
      commonjs: 'vue',
      commonjs2: 'vue',
      amd: 'vue'
    }
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': '"production"'
    })
  ]
});
```

```javascript
/* gen-style.js */
const gulp = require('gulp');
const cleanCSS = require('gulp-clean-css');
const sass = require('gulp-sass');
const rename = require('gulp-rename');
const autoprefixer = require('gulp-autoprefixer');
const components = require('./components.json')

function buildCss(cb) {
  gulp.src('../src/styles/index.scss')
    .pipe(sass())
    .pipe(autoprefixer())
    .pipe(cleanCSS())
    .pipe(rename('lime-ui.css'))
    .pipe(gulp.dest('../lib/styles'));
  cb()
}

exports.default = gulp.series(buildCss)
```

OK，这里我们的webpack配置基本设置好了，webpack.base.js 中的配置就主要是一些loader和插件的配置，具体的出入口都是在 webpack.prod.js 中配置的。这里webpack.prod.js 合并了 webpack.base.js 中的配置项。关于 output.libary 和 externals ，阅读了之前 “准备” 阶段的内容的应该不会陌生了。

另外还有 gen-style.js 这个文件是单独使用了 gulp 来对样式文件进行打包操作的，我们这里选用的是 scss的语法，如果你想用less或其他的预处理器，也可以自行修改这里的文件和相关依赖。

不过这个配置肯定还没有结束，首先我们需要安装好这里的配置里使用到的各种loader和plugin。为了不漏掉安装项和保持一致性，可以直接复制下面的配置内容放到 package.json 下，通过 `npm install` 来进行安装。需要注意的是，这里的安装完成之后，其实后面的一些内容的依赖也都一并安装好了。

```json
"dependencies": {
  "async-validator": "^3.0.4",
  "core-js": "2.6.9",
  "webpack": "^4.39.2",
  "webpack-cli": "^3.3.7"
},
"devDependencies": {
  "@babel/core": "^7.5.5",
  "@babel/plugin-transform-runtime": "^7.5.5",
  "@babel/preset-env": "^7.5.5",
  "@vue/test-utils": "^1.0.0-beta.29",
  "babel-loader": "^8.0.6",
  "chai": "^4.2.0",
  "cross-env": "^5.2.0",
  "css-loader": "2.1.1",
  "file-loader": "^4.2.0",
  "gh-pages": "^2.1.1",
  "gulp": "^4.0.2",
  "gulp-autoprefixer": "^7.0.0",
  "gulp-clean-css": "^4.2.0",
  "gulp-rename": "^1.4.0",
  "gulp-sass": "^4.0.2",
  "karma": "^4.2.0",
  "karma-chai": "^0.1.0",
  "karma-chrome-launcher": "^3.1.0",
  "karma-coverage": "^2.0.1",
  "karma-mocha": "^1.3.0",
  "karma-sinon-chai": "^2.0.2",
  "karma-sourcemap-loader": "^0.3.7",
  "karma-spec-reporter": "^0.0.32",
  "karma-webpack": "^4.0.2",
  "less": "^3.10.2",
  "less-loader": "^5.0.0",
  "mocha": "^6.2.0",
  "node-sass": "^4.12.0",
  "rimraf": "^3.0.0",
  "sass-loader": "^7.3.1",
  "sinon": "^7.4.1",
  "sinon-chai": "^3.3.0",
  "style-loader": "^1.0.0",
  "url-loader": "^2.1.0",
  "vue-loader": "^15.7.1",
  "vue-style-loader": "^4.1.2",
  "vuepress": "^1.0.3"
},
```

另外，由于我们使用了babel，所以需要在项目的根目录下设置一下 `.babelrc` 文件，内容如下：

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "loose": false,
        "modules": "commonjs",
        "spec": true,
        "useBuiltIns": "usage",
        "corejs": "2.6.9"
      }
    ]
  ],
  "plugins": [
    "@babel/plugin-transform-runtime",
  ]
}
```

当然也不要忘记在package.json文件中写上scripts简化手动输入命令的过程

```json
{
	"scripts": {
    "build:style": "gulp --gulpfile build/gen-style.js",
    "build:prod": "webpack --config build/webpack.prod.js",
  }
}
```



### 第三步，建立文档化工具

如果在上一步中未安装了 vuepress ，可以通过 `npm install vuepress --save-dev` 来安装，

然后在 package.json 中加入脚本，快速启动

```json
{
  "scripts": {
    // ...
    "docs:dev": "vuepress dev docs",
    "docs:build": "vuepress build docs"
  }
}
```

这个时候你可以在你的 docs/README.md 文件里写点内容，然后运行 `npm run docs:dev` 就可以看到本地的文档内容了。需要打包的时候使用 `npm run docs:build ` 就可以了。

如果我们的项目是要放到github上的，那么其实也可以一并将我们的文档生成之后也放到github上去，利用github的pages功能让这个本地的文档在线运行。（github pages托管我们的静态页面和资源）

可以运行 `npm install gh-pages --save-dev` 安装 `gh-pages` 这个可以帮我们一键部署github pages文档的工具。它的工作原理就是将对应的某个文件夹下的资源迁移到我们的当前项目的gh-pages分支上，然后这个分支在push给了github之后，github就会将该分支内的内容服务起来。为了更好的使用它，我们可以在package.json中添加scripts

```json
{
  "scripts": {
    // ...
  	"deploy": "gh-pages -d docs/.vuepress/dist",
    "deploy:build": "npm run docs:build && npm run deploy",
  }
}
```

这样你就可以使用 `npm run deploy` 直接部署你的vuepress生成的静态站点，不过务必在部署之前运行一下文档的构建程序。因此我们也添加了一条 `npm run deploy:build` 命令，使用这条命令就可以直接把文档的构建和部署直接一起解决。是不是很简单呢？

不过为了我们能够直接使用自己写的组件，还需要对vuepress做一点点配置。在 docs/.vuepress目录下新建一个 enhanceApp.js 文件，写入如下内容，将我们的组件库的入口和样式注入进去

```javascript
import LimeUI from '../../src/index.js'
import "../../src/styles/index.scss"

export default ({
  Vue,
  options,
  router
}) => {
  Vue.use(LimeUI)
}
```

这个时候我们之后写的组件就可以直接在文档中使用了。



### 第四步，样式构建

先需要说明的是这里我们所使用的样式预处理器的语法是scss。那么在“完善打包流程”这一小节中已经将用gulp进行打包的代码给出了，不过有必要说明一下，我们又是如何去整合样式内容的。

首先，为了之后便于做按需加载，对于每个组件的样式都是一个单独的scss文件，写样式的时候，为了避免太多的层级嵌套，使用了BEM风格的方式去书写。

我们需要先在 src/styles目录执行如下命令生成一个基本的样式文件

```bash
cd src/styles
mkdir common
mkdir mixins
touch common/var.scss  # 样式变量文件
touch common/mixins.scss
touch index.scss  # 引入所有样式
```

然后将对应的 var.scss 和 mixins.scss 文件填充上一些基础内容

```scss
/* common/var.scss */

$--color-primary: #ff6b00 !default;
$--color-white: #FFFFFF !default;
$--color-info: #409EFF !default;
$--color-success: #67C23A !default;
$--color-warning: #E6A23C !default;
$--color-danger: #F56C6C !default;
```

```scss
/* mixins/mixins.scss */
$namespace: 'lime';  /* 组件库的样式前缀 */

/* BEM
 -------------------------- */
@mixin b($block) {
  $B: $namespace+'-'+$block !global;

  .#{$B} {
    @content;
  }
}
```

在mixins文件中我们声明了一个mixin，用于帮助我们更好的去构建样式文件。



## 组件打造案例

上面的内容设置好了， 咱们就可以开始具体去做一个组件试试了

### 简单的button组件

这是做好之后的大致效果

![2.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MDIzMzc0.png)

OK，那我们建立基本的button组件相关的文件

```bash
cd src/packages
mkdir button && cd button
touch index.js
touch button.vue
```

写入button.vue的内容

```javascript
<template>
  <button class="lime-button" :class="{[`lime-button-${type}`]: true}" type="button">
    <slot></slot>
  </button>
</template>

<script>
import { oneOf } from '../../utils/assist';

export default {
  name: 'Button',
  props: {
    type: {
      validator (value) {
          return oneOf(value, ['default', 'primary', 'info', 'success', 'warning', 'error']);
      },
      type: String,
      default: 'default'
    }
  }
}
</script>
```

这里我们需要在 index.js 中导出这个组件

```javascript
import Button from './button.vue'
export default Button
```

这样单个的一个组件就完成了，之后你可以再多做几个组件试试，不过有一点就是这些组件需要一个统一的打包入口，我们再webpack中已经配置过了，那就是 src/index.js 这个文件，我们需要在这个文件里面将我们刚才写的button组件以及你自己写的其他组件都引入进来，然后统一导出给webpack打包使用，具体代码见下

```javascript
import Button from './packages/button'

const components = {
  lButton: Button,
}

const install = function (Vue, options = {}) {

  Object.keys(components).forEach(key => {
    Vue.component(key, components[key]);
  });
}

export default install
```

可以看到的是index.js中我们最终导出的是一个叫install的函数，这个函数其实就是Vue插件的一种写法，便于我们在实际项目中引入的时候可以使用 `Vue.use` 的方式来自动安装我们的整个组件库。install接受两个参数，一个是Vue，我们把它用来注册一个个的组件。还有一个是options，便于我们可以在注册组件的时候传入一些初始化参数，比如默认的按钮大小、主题等信息，都可以通过参数的方式来设定。

然后我们可以在 src/styles目录下新建一个button.scss 文件，写入我们button对应的样式

```scss
/* button.scss */
@charset "UTF-8";
@import "common/var";
@import "mixins/mixins";

@include b(button) {
  min-width: 60px;
  height: 36px;
  font-size: 14px;
  color: #333;
  background-color: #fff;
  border-width: 1px;
  border-radius: 4px;
  outline: none;
  border: 1px solid transparent;
  padding: 0 10px;

  &:active,
  &:focus {
    outline: none;
  }

  &-default {
    color: #333;
    border-color: #555;

    &:active,
    &:focus,
    &:hover {
      background-color: rgba($--color-primary, 0.3);
    }
  }
  &-primary {
    color: #fff;
    background-color: $--color-primary;

    &:active,
    &:focus,
    &:hover {
      background-color: mix($--color-primary, #ccc);
    }
  }

  &-info {
    color: #fff;
    background-color: $--color-info;

    &:active,
    &:focus,
    &:hover {
      background-color: mix($--color-info, #ccc);
    }
  }
   &-success {
    color: #fff;
    background-color: $--color-success;

    &:active,
    &:focus,
    &:hover {
      background-color: mix($--color-success, #ccc);
    }
  }
}
```

最后我们还需要在 src/styles/index.scss 文件中将button的样式引入进去

```scss
@import "button";
```

为了简单的实验，你可以直接在 docs/README.md 文件下写两个button组件试试看

```javascript
<template>
	<l-button type="primary">Click me</l-button>
</template>
```

如果你想要得到和我在 https://arronkler.github.io/lime-ui/ 上一样的效果，可以参考 https://github.com/arronKler/lime-ui 项目中的 docs 目录下的配置。如果想要更个性化的配置，可以查阅vuepress的官方文档。



### Notice提示组件

这个组件就要用到我们的动态渲染的相关的东西了。具体最后的使用方式是这样的

```javascript
this.$notice({
  title: '提示',
  content: this.content || '内容',
  duration: 3
})
```

效果类似于这样

![4.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MDIzMzAx.png)

OK，我们先来写一下这个组件的一个基本源码

在 src/packages 目录下新建notice文件夹，然后新建一个 notice.vue 文件

```javascript
<template>
  <div class="lime-notice">
    <div class="lime-notice__main" v-for="item in notices" :key="item.id">
      <div class="lime-notice__title">{{item.title}}</div>
      <div class="lime-notice__content">{{item.content}}</div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      notices: []
    }
  },
  methods: {
    add(notice) {
      let id = +new Date()
      notice.id = id
      this.notices.push(notice)

      const duration = notice.duration
      setTimeout(() => {
        this.remove(id)
      }, duration * 1000)
    },
    remove(id) {
      for(let i = 0; i < this.notices.length; i++) {
        if (this.notices[i].id === id) {
          this.notices.splice(i, 1)
          break;
        }
      }
    }
  }
}
</script>

```

代码很简单，其实就是声明了一个容器，然后在其中通过控制 notices 的数据来展示和隐藏，接着我们在同一个目录下新建一个notice.js 文件来做动态渲染

```javascript
import Vue from 'vue'
import Notice from './notice.vue'

Notice.newInstance = (properties) => {
  let props = properties || {}
  const Instance = new Vue({
    render(h) {
      return h(Notice, {
        props
      })
    }
  })

  const component = Instance.$mount()
  document.body.appendChild(component.$el)

  const notice = component.$children[0]

  return {
    add(_notice) {
      notice.add(_notice)
    }, 
    remove(id) {

    }
  }
}

let noticeInstance


export default (_notice) => {
  noticeInstance = noticeInstance || Notice.newInstance()
  noticeInstance.add(_notice)
}
```

这里我们我们通过动态渲染的方式让我们的组件可以直接挂在到body下面，而非归属于根挂载点之下。

然后在 src/styles 目录下新建 notice.scss 文件，写上我们的样式文件

```scss
/* notice.scss */
@charset "UTF-8";
@import "common/var";
@import "mixins/mixins";

@include b(notice) {
  position: fixed;
  right: 20px;
  top: 60px;
  z-index: 1000;

  &__main {
    min-width: 100px;
    padding: 10px 20px;
    box-shadow: 0 0 4px #aaa;
    margin-bottom: 10px;
    border-radius: 4px;
  }

  &__title {
    font-size: 16px;
  }
  &__content {
    font-size: 14px;
    color: #777;
  }
}
```

最后同样的，也需要在 src/index.js 这个入口文件中对 notice做处理。完整代码是这样的。

```javascript
import Button from './packages/button'
import Notice from './packages/notice/notice.js'

const components = {
  lButton: Button
}

const install = function (Vue, options = {}) {

  Object.keys(components).forEach(key => {
    Vue.component(key, components[key]);
  });

  Vue.prototype.$notice = Notice;
}

export default install
```

我们可以看到我们再Vue的原型上挂上了我们的 `$notice` 方法，这个方法调用的时候就会触发我们在 notice.js 文件中动态渲染组件的一套流程。这个时候我们就可以在 docs/README.md 文档中测试着用了。

```javascript
<script>
export default() {
  mounted() {
    this.$notice({
        title: '提示',
        content: this.content,
        duration: 3
    })
  }
}
<script>
```






## 单独打包样式和组件

为了能支持按需加载的功能，我们除了将整个组件库打包之外，还需要对样式和组件单独打包成单个的文件。这里我们需要做两件事儿

1. 打包单独的css文件
2. 打包单独的组件内容

对于第一点，我们需要对 build/gen-style.js 文件做一下改造，加上buildSeperateCss任务，完整代码如下

```javascript
// 其他之前的代码...

function buildSeperateCss(cb) {
  Object.keys(components).forEach(compName => {
    gulp.src(`../src/styles/${compName}.scss`)
      .pipe(sass())
      .pipe(autoprefixer())
      .pipe(cleanCSS())
      .pipe(rename(`${compName}.css`))
      .pipe(gulp.dest('../lib/styles'));
  })

  cb()
}

exports.default = gulp.series(buildCss, buildSeperateCss) // 加上 buildSeperateCss
```

对于第二点，我们可以用一个新的webpack配置来处理，新建一个 build/webpack.component.js 文件，写入

```javascript
const path = require('path');
const webpack = require('webpack');
const merge = require('webpack-merge');
const webpackBaseConfig = require('./webpack.base.js');
const components = require('./components.json')
process.env.NODE_ENV = 'production';

const basePath = path.resolve(__dirname, '../')
let entries = {}
Object.keys(components).forEach(key => {
  entries[key] = path.join(basePath, 'src', components[key])
})

module.exports = merge(webpackBaseConfig, {
  devtool: 'source-map',
  mode: "production",
  entry: entries,
  output: {
    path: path.resolve(__dirname, '../lib'),
    publicPath: '/lib/',
    filename: '[name].js',
    chunkFilename: '[id].js',
    // library: 'lime-ui',
    libraryTarget: 'umd',
    umdNamedDefine: true
  },
  externals: {
    vue: {
      root: 'Vue',
      commonjs: 'vue',
      commonjs2: 'vue',
      amd: 'vue'
    }
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': '"production"'
    })
  ]
});

```

这里我们引用了build文件夹下的一个叫做 component.json 的文件，该文件是我自定义用来标识我们的组件和组件路径的，实际上你也可以通过脚本直接遍历 src/packages目录自动获得这样一些信息。这里只是简单演示， build/component.json 的代码如下

```json
{
  "button": "packages/button/index.js",
  "notice": "packages/notice/notice.js"
}
```

所有的单独打包流程配置好以后，我们就可以在 package.json 文件中再加上 scripts 命令

```json
{
	"scripts": {
    // ...
		"build:components": "webpack --config build/webpack.component.js",
    "dist": "npm run build:style && npm run build:prod && npm run build:components",
	}
}
```

OK，现在只需要运行 `npm run dist` 命令，它就会自动去构建完整的样式内容和各个组件单独的样式内容，然后会打包一个完整的组件包和各个组件的单独的包。

这里需要注意的一点就是你的package.json 文件中的这几个字段需要做一下调整

```json
{
	"name": "lime-ui",
  "version": "1.0.0",
  "main": "lib/lime-ui.min.js",
  //...
}
```

其中name表示别人使用了你的包的时候的包名，main字段很重要，表示别人直接引入你包的时候，入口文件是哪一个。这里因为我们webpack打包后的文件是 lib/lime-ui.min.js 所以我们这样去设置。

一切就绪后，你就可以运行 `npm run dist` 打包你的组件库，然后 `npm publish` 去发布你的组件库了（发布前需要 `npm login` 登陆）



## 使用自己的组件库

### 直接使用

我们可以用vue-cli 或其他工具另外生成一个demo项目，用这个项目去引入我们的组件库。如果你的包还没有发布出去，可以在你的组件库项目目录下 用 `npm link` 或者 `yarn link`的命令创建一个link（推荐使用yarn）

然后在你的demo目录下使用 `npm link package_name` 或者 `yarn link package_name` 这里的package_name就是你的组件库的包名，然后在你的demo项目的入口文件里

```javascript
import Vue from vue
import LimeUI from 'lime-ui'
import 'lime-ui/lib/styles/lime-ui.css'
// 其他代码 ...

Vue.use(LimeUI)
```

这样设置好之后，我们创建的组件就可以在这个项目里使用了



### 按需加载

上面我们谈的是全局载入的一种使用方法，那如何按需加载呢？其实我们之前也说过那么一点

先通过npm安装好 `babel-plugin-component` 包，然后在你的demo项目的 `.babelrc` 文件中写上这部分内容

```json
{
    "plugins": [
        ["component", {
            "libraryName": "lime-ui",
            "libDir": "lib",
            "styleLibrary": {
                "name": "styles",
                "base": false, // no base.css file
                "path": "[module].css"
            }
        }]
    ]
}
```

这里的配置是要符合我们的lime-ui 的一个目录结构的，有了这个配置我们就可以进行按需加载了，你可以像这样做加载一个Button

```javascript
import Vue from 'vue'
import { Button } from 'lime-ui'

Vue.component('a-button', Button)
```

可以看到的是，我们并没有在这个位置加载任何样式，因为 `babel-plugin-component` 已经帮我们做了，不过因为我们只在组件库的入口点里面设置了 install 方法用来注册组件，所以这里我们按需引入的时候，就需要自己手动注册了。



### 主题定制

前面的内容做好之后，主题定制就比较简单了，我们先在DEMO项目的入口文件同级目录下创建一个 global.scss 文件，然后在其中写入类似下面这样的代码。

```scss
$--color-primary: red;
@import "~lime-ui/src/styles/index.scss";
```

然后在入口文件中把引入组件库的方式改变一下

```javascript
import Vue from vue
import LimeUI from 'lime-ui'
import './global.scss'
// 其他代码 ...

Vue.use(LimeUI)
```

我们在入口文件中把对组件库的样式引入，改成引入我们自定义的global.scss文件。

其实这里就是覆盖了我们在组件库项目里 var.scss 里的变量的值，然后其余的组件基础样式还是使用了各自的样式内容，这样就可以达到主题定制了。



# 结语

本文通过对组件库的一些特性的介绍和一个实际的操作案例，阐述了打造一套组件库的一些基础的东西。希望能通过这样的一次分享，让我们不只是去使用组件库，而是能知道组件库的诞生过程和了解组件库的一些内部特性，帮助我们在日常使用的过程中能“心中有数”，当出现问题或组件库需求可能不满足的时候有一个新的思考入手点，那就足够了。



# 引用参考



1. Vue` $dispatch `和` $broadcast `详解: https://juejin.im/post/5c7fd345f265da2da771f4cd
2. Component Tests with Vue.js - Matt O'Connell : https://www.youtube.com/watch?v=OIpfWTThrK8
3. 掘金小册：Vue.js 组件精讲
4. ElementUI ：https://github.com/ElemeFE/element
5. iView ：https://github.com/iview/iview