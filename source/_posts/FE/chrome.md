title: Chrome扩展开发科普
date: 2019-08-20
author: sunguoqiang
toc: true
thumbnail: https://piccdn.luojilab.com/fe-oss/default/MTU2ODYzMjYxNjU1.png
tag: 
  - 前端
categories: 
  - 前端

---

## chrome 扩展是什么
chrome 扩展是用传统的 HTML、CSS、JS、图片等静态资源开发并最终打包成后缀为 `.crx` 的一个压缩包。所以，它和我们平常开发的页面没有多大的区别，所以如果你想引入前端开发所用的各种框架，组件库，构建工具也都是可以的。主要区别只有 2 个:

1. 扩展的页面、js 和普通的页面运行位置不同
2. 扩展可以调用 chrome 提供的更多的 API 来增强我们扩展的能力

<!-- more -->

## chrome 扩展的安装方式
扩展的安装方式有 3 种：
1. 通过 chrome 扩展商店，下载安装
2. 在其他网站下载打包好的 `.crx` 压缩包，把压缩包直接拖拽到 chrome 的扩展管理页面
3. 如果是自己开发的扩展，可以在扩展管理页面，打开开发者模式，手动加载已解压的扩展程序，进行本地调试

![install.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MzU1NTg3.png)

## chrome 扩展的展现形式
这里只简单地介绍几种经常见到的，还有更多的展现形式，大家感兴趣可以去官方文档详细了解
1. 点击地址栏右侧 icon 会有页面弹出，这个大多数扩展都会有，主要是扩展的设置或者功能的入口
2. 页面修饰内容：通过添加 DOM 对页面赋予新功能，比如 `Octotree` (对 gitub 项目页面做导航)
3. 页面右键菜单：定制在页面内右键弹出的菜单，很多划线翻译的扩展都利用了这个功能
4. 覆盖 chrome 默认页面： chrome 有的页面支持开发者自定义，比如 Momentum 就覆盖了默认的 New Tab 页面
5. devtool 工具：这个是开发者经常用到的 比如 `vue-devtool` 等框架提供的调试工具

## 开发介绍
具体扩展各个组成部分的学习，我们以一个很简单的例子为基础进行介绍，这个扩展是一个为页面添加回到顶部功能的扩展。

### 配置文件
每一个扩展都必须要有的一个名字为 manifest.json 的配置文件，这个文件声明了此扩展用到了哪些功能，及各个功能需要用到的静态资源

![manifest1.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MzU1NTY5.png)

![manifest2.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MzU1NTQy.png)

这个 backToTop 扩展的配置已经在途中说明，已经有了基本的说明，下面对一些配置项做下额外的说明,全部配置可以在[官网](https://developer.chrome.com/extensions/manifest)查看:
1. browser_action 指定了 popup 页面相关的 icon、html、tooltip 文字等配置，相似的还有一个 page_action，它的配置参数和 browser_aciton 是相同的 但是它可以通过 `chrome.pageAction` API 来动态的设置扩展在某些页面的行为
2. icons 配置不用每个尺寸都给出，chrome 会自己选出效果最合适的 icon
3. permissions 声明扩展需要用到的 chrome 特性

## 常用 API
1. chrome.runtime
2. chrome.tabs 对标签页进行操作、与对应标签页内容通讯
3. chrome.storage 扩展的存储，类似 localstorage
4. chrome.contextMenus
5. chrome.extension

## 核心 JS
这部分我们说下扩展开发核心的几种 JS

### popup
popup 页面生命周期是点击弹出时，初始化，关闭时，页面也跟着销毁, 并且这个页面没有任何跨域的限制。它在我们扩展里的作用是配置页面里 backToTop icon 的样式，并存入storage。

![popup.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MzU1NTI5.png)

``` javascript
chrome.storage.local.get(['right', 'bottom', 'icon'], function(result) {
  if(result) {
    // 初始化
    right.value = result.right || '';
    bottom.value = result.bottom || '';
    if(result.icon) {
      img.src = result.icon;
      img.dataset.uploaded = true;
    }
  }
})
```

页面初始化时，从 storage 取出存储的值，初始化页面。
保存参数时，再将相关参数存入 storage

``` javascript
chrome.storage.local.set({right: rightVal, bottom: bottomVal});
  if(img.dataset.uploaded) {
    chrome.storage.local.set({icon: img.src});
  }
```
注: storage API 有 2 种 `storage.sync` 和 `storage.local` 他们的区别

1. sync 会将存储的数据定时发到 chrome 的服务器，进行数据的同步，local 就只是将数据存储在本地
2. 尺寸的不同：local 和 `localstorage` 是一样的 5M. 而 sync 存储的大小只有 100K 而且对于单个 key 的值大小，以及写入的频率也有限制，毕竟要同步服务端，所以如果开发的扩展只是用于个人使用的效率提升，不打算发布，可以直接用 local 就好了

### content-script
`content-script` 是我们用来定制化页面，实现页面内扩展逻辑的地方。它的特点是：
1. 因为在页面内，当然可以访问 DOM 但是和页面的 js 是完全独立的，不能互相访问
2. 无法对页面内的 DOM 事件绑定 扩展里的回调，这种情况可以通过 `content-script` 创建一个 script 标签插入到 DOM 里 这个新的 script 里的函数是可以绑定的
3. 因为在页面里运行，所以是会收到跨域限制滴
4. 运行时机是随着页面的加载而运行，页面关闭也就卸载了，所以说 `content-script` 会在每一个 tab 页面里都有一份代码在运行
5. 因为在页面初始化才会运行，所以在初始加载插件时，需要刷新页面，`content-script ` 才会开始运行。
6. 注入页面的 css 优先级非常高，一定要注意好类名 ID 名的设置

那我们的 backToTop 里 `content-script` 都干了什么呢
首先，初始化我们页面里的 icon 并根据页面 `scrollTop` 判断当前是否需要展示 icon

``` javascript
const el = document.createElement('div');
el.show = true; // 控制icon是否显示
el.classList.add('ce-btt-container');


el.style.opacity = target.scrollTop > visibleHeight ? 1 : 0;

const img = document.createElement('img');
img.classList.add('ce-btt-icon');

el.appendChild(img);


chrome.storage.local.get(['right', 'bottom', 'icon'], function(result) {
  el.style.right = result && result.right ? result.right + 'px' : right;
  el.style.bottom = result && result.bottom ?  result.bottom + 'px' : bottom;
  img.src = result && result.icon ? result.icon : chrome.runtime.getURL('icons/backToTop.png');
  document.body.appendChild(el);
});
```

第二步， 要想实现返回顶部，当然要给我们的 icon 绑定点击事件，以及监听 `scroll` 事件判断什么时候该隐藏展示

``` javascript
el.addEventListener('click', function(e) {
  let step = 20;
  let timer = setInterval(() => {
    if(target.scrollTop <= 0) {
      clearInterval(timer);
    } else {
      step += 20;
      target.scrollTop -= step;
    }
  }, 20);
});

const handleScroll = function() {
  if(!el.show) return false;  // icon不显示时，不处理
  if(target.scrollTop > visibleHeight) {
    el.style.opacity = 1;
  } else {
    el.style.opacity = 0;
  }
};

container.addEventListener('scroll', throttle(handleScroll, 300));
```

第三步， 如果 popup 页面有配置的变更， `content-script` 都需要立刻进行更新

```javascript
chrome.storage.onChanged.addListener(function(changes, namespace) {
  if(changes.bottom) {
    el.style.bottom = `${changes.bottom.newValue}px`;
  }
  if (changes.right) {
    el.style.right = `${changes.right.newValue}px`;
  }
  if (changes.icon) {
    img.src = changes.icon.newValue;
  }
});
```
注: 这里需要注意的是，因为我们要把扩展里的 icon 插入到页面，所以需要在 `manifest.json` 里配置 `web_accessible_resources` 赋予页面可以访问我们指定的扩展静态资源的权限。这是因为页面里 icon 的 src 属性是这样的

![resource.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MzU1NDg4.png)

### background
`background` 可以理解为是扩展在后台一直运行的一个 JS(实际并不是)， 它在整个浏览器里只会有一个 js 在运行。在 background 的配置里，有一个 `persistent` 的配置, 当它为 true 时，background 才会一直运行，false 时，浏览器会检测长时间不活动时，自动卸载调，只有监听的事件发生时，才会重新执行，官方的说明是
> The only occasion to keep a background script persistently active is if the extension uses chrome.webRequest API to block or modify network requests. The webRequest API is incompatible with non-persistent background pages.

所以绝大多数时候，我们把它设为 false 就可以了。另外 `background` 也是可以跨域的，
所以我们可以总结出，除了页面内的 js chrome 对其他的 js 都没有跨域的限制。

好，我们看看我们扩展里 `background` 干了啥

首先，初始时，肯定要监听浏览器的初始化事件，才可以绑定扩展关注的事件。

``` javascript
chrome.runtime.onInstalled.addEventListener(function() {
  // init extention
})
```

第二步，因为有的页面已经提供了返回顶部的功能，所以这个时候我们需要提供可以把我们的 icon 永久隐藏的功能，我们在 `background` 初始化的逻辑中，创建一个鼠标右键的菜单项，这个菜单项可以实现 切换我们的 icon 显示状态的功能

``` javascript
chrome.contextMenus.create({
    title: 'toggle',
    id: 'toggle'
  });

chrome.contextMenus.onClicked.addListener(function() {
  chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
    sendMsg(tabs.length ? tabs[0].id : null);
  });
});

function sendMsg(tabId) {
  chrome.tabs.sendMessage(tabId, 'toggle', function(res) {
    console.log(res);
  });
}
```
发送消息这里我们用了 `chrome.tabs` API 因为我们每个 tab 都会有一个 `content-script` 所以需要筛选出当前所在的标签页，然后发送消息。

最后，我们的 `content-script` 需要监听发送消息的事件，并切换 icon 的状态
```javascript
chrome.runtime.onMessage.addListener(function(req, sender, respond) {
  if(req === 'toggle') {
    if (target.scrollTop > visibleHeight) {
      el.style.opacity = el.style.opacity === '1' ? 0 : 1;
    }
    el.show = !el.show;
    respond('toggle success');
  }
});
```
至此，整个扩展的功能就基本介绍完了，过程中用到的 API 这里并不作详细的介绍，详细使用还是需要大家去 [官网](https://developer.chrome.com/apps/api_index) 查看
### 调试
最后说一下如何调试，调试个人认为还是比较麻烦的， 代码变更并没有热更新，所以需要我们手动去扩展管理页重新加载。而且几种不同的 JS 调试的位置也不同，设计通讯时经常需要在几个不同的地方来回切换

`popup` 调试：在弹出的窗口里，右键审查元素就可以弹出调试窗口，调试方式和普通的页面调试没有区别。

`background` 调试: 在管理页点击背景页，就可以弹出调试窗口了

![bg-dev.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MzU0OTQx.png)

`content-script` 调试: 因为是在页面运行，所以调试的地方和页面 js 是在一个窗口里

代码位置：在 sources tab 下选中 Content scripts 就可以看到页面加载的全部的扩展 `contnt-script` 了。

![cs-code.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MzU1NjE5.png)

console 输出: 在 Console tab 下拉框里选中要调试的扩展，就可以看到对应扩展的 console 输出。

![cs-console.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MzU1NjAy.png)

## 打包发布
在扩展管理页，可以打包扩展程序,打包后就可以生成 `.crx` 文件

![bundle.png](https://piccdn.luojilab.com/fe-oss/default/MTU2ODI4MzU0OTE4.png)

打包之后，就可以发布了，不过发布需要先花 5$ 注册开发者，我就没有继续下去了。

## 总结
在开发过程中，可以发现写一个扩展其实并不难，用到的技术都是前端每天都在用的东西。只要我们多加留心，就会发现使用 chrome 过程中有许多可以提效，优化体验的地方，这时候我们就可以试着用扩展的方式解决。总体来说，chrome 扩展是一种技术成本很低，就可以干些有趣的事情的技术

参考资料：

[官方文档](https://developer.chrome.com/extensions)

[Chrom插件开发全攻略](https://www.cnblogs.com/liuxianan/p/chrome-plugin-develop.html)