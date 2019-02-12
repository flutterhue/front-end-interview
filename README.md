## 前端面试题收录
* 浏览器渲染过程，回流重绘
  - https://juejin.im/post/5b2e352ef265da59af409832
  - 渲染过程
    1. url 解析: url -> ip(by dns) -> 建立TCP连接 -> 发送http request -> 收到http request
    2. html 解析: html构建DOM Tree, CSS构建CSSOM(Rule Tree), 然后合并为rendering tree
    3. layout: 基于rendering tree, 浏览器开始计算各个节点内容在屏幕上的位置
    4. paint: 按照上一步计算的结果在浏览器上进行绘制
    5. script 解析
  - 几个概念
    + layout(布局)或者reflow(回流), 某个部分发生了变化影响了布局，渲染树需要重新计算，计算出每一个渲染对象的位置和尺寸，将其安置在浏览器窗口的正确位置，而有些时候我们会在文档布局完成后对DOM进行修改，这时候可能需要重新进行布局，也可称其为回流，本质上还是一个布局的过程，每一个渲染对象都有一个布局或者回流方法，实现其布局或回流。
    + paint(绘制)或repaint(重绘)浏览器UI组件将遍历渲染树并调用渲染对象的绘制（paint）方法，将内容展现在屏幕上，也有可能在之后对DOM进行修改，需要重新绘制渲染对象，也就是重绘，绘制和重绘的关系可以参考布局和回流的关系. 改变了某个元素的背景颜色，文字颜色等，不影响元素周围或内部布局的属性，将只会引起浏览器的repaint，根据元素的新属性重新绘制，使元素呈现新的外观。重绘不会带来重新布局，并不一定伴随重排；Reflow要比Repaint更花费时间，也就更影响性能。所以在写代码的时候，要尽量避免过多的Reflow。
    + DOM的增加删除移动, 页面内容的变更, 浏览器窗口的大小变化或者滚动等会影响节点位置的操作都将导致reflow.
    + 只发送重绘通常来说是一下几个元素样式发送了变化`color`, `background-color`, `visibility`
  - 如果避免(过多不必要的回流和重绘)
    + 将动画效果应用到position属性为absolute或fixed的元素上（脱离文档流）
    + 避免频繁操作样式，最好一次性重写style属性，或者将样式列表定义为class并一次性更改class属性
    + 避免频繁操作DOM，创建一个documentFragment，在它上面应用所有DOM操作，最后再把它添加到文档中, 也可以先为元素设置display: none，操作结束后再把它显示出来。因为在display属性为none的元素上进行的DOM操作不会引发回流和重绘
    + 避免频繁读取会引发回流/重绘的属性，如果确实需要多次使用，就用一个变量缓存起来
     ```
        现代浏览器会对频繁的回流或重绘操作进行优化：
        浏览器会维护一个队列，把所有引起回流和重绘的操作放入队列中，如果队列中的任务数量或者时间间隔达到一个阈值的，浏览器就会将队列清空，进行一次批处理，这样可以把多次回流和重绘变成一次。
        当你访问以下属性或方法时，浏览器会立刻清空队列：

        clientWidth、clientHeight、clientTop、clientLeft
        offsetWidth、offsetHeight、offsetTop、offsetLeft
        scrollWidth、scrollHeight、scrollTop、scrollLeft
        width、height
        getComputedStyle()
        getBoundingClientRect()

        因为队列中可能会有影响到这些属性或方法返回值的操作，即使你希望获取的信息与队列中操作引发的改变无关，浏览器也会强行清空队列，确保你拿到的值是最精确的。
     ```
* 浏览器中DOMContentLoaded, load等等事件的触发顺序
  + https://github.com/fi3ework/BLOG/issues/3
  + `DOMContentLoaded` —— 浏览器已经完全加载了 HTML，DOM 树已经构建完毕，但是像是  `<img>` 和样式表等外部资源可能并没有下载完毕, 此时JS可以访问所有 DOM 节点，初始化界面
  + `load` —— 浏览器已经加载了所有的资源（图像，样式表等), 此时可以获得图片大小
  + `beforeunload` 在用户即将离开页面时触发，它返回一个字符串，浏览器会向用户展示并询问这个字符串以确定是否离开
  + `unload` 在用户已经离开时触发，我们在这个阶段仅可以做一些没有延迟的操作，由于种种限制，很少被使用
  + `defer`会在`DOMContentLoaded`触发之前就执行, 但是`async`不确定, 只是在下载完成之后立刻执行。
  + 考虑到上述情况, 有时候我们还需要确定页面的状态：
  ```
  <script>
  function log(text) { /* output the time and message */ }
  log('initial readyState:' + document.readyState);

  document.addEventListener('readystatechange', () => log('readyState:' + document.readyState));
  document.addEventListener('DOMContentLoaded', () => log('DOMContentLoaded'));

  window.onload = () => log('window onload');
</script>

<iframe src="iframe.html" onload="log('iframe onload')"></iframe>

<img src="http://en.js.cx/clipart/train.gif" id="img">
<script>
  img.onload = () => log('img onload');
</script>
* `document.readyState` 在 `DOMContentLoaded` 前一刻变为 `interactive`，这两个事件可以认为是同时发生。
* `document.readyState` 在所有资源加载完毕后（包括 `iframe` 和 `img`）变成 `complete`，我们可以看到`complete`、 `img.onload` 和 `window.onload` 几乎同时发生，区别就是 `window.onload` 在所有其他的 `load` 事件之后执行
  ```
* 你能描述渐进增强 (progressive enhancement) 和优雅降级 (graceful degradation) 之间的不同吗?
  ```
  渐进增强（Progressive Enhancement）：一开始就针对低版本浏览器进行构建页面，完成基本的功能，然后再针对高级浏览器进行效果、交互、追加功能达到更好的体验。

  优雅降级（Graceful Degradation）：一开始就构建站点的完整功能，然后针对浏览器测试和修复。比如一开始使用 CSS3 的特性构建了一个应用，然后逐步针对各大浏览器进行 hack 使其可以在低版本浏览器上正常浏览。
  ```
* 你如何对网站的文件和资源进行优化？/ 请说出三种减少页面加载时间的方法。(加载时间指感知的时间或者实际加载时间)
  + CDN (Content Distribution Network)
  + 尽可能减少http请求次数，将css, js, 图片各自合并 
  + 添加Expire/Cache-Control头
  + 启用Gzip压缩文件
  + 最小化css, js，减小文件体积
* 浏览器同一时间可以从一个域名下载多少资源？
  + 即使最新的也在8以内, 目的主要是安全以及性能因素
  + 并发过多请求容易超出服务器阈值而被BAN.
  + 有利于浏览器复用现有连接 (keep alive技术)
  + 详见 https://www.zhihu.com/question/20474326
* 什么是viewport, 有什么作用??
  + https://www.cnblogs.com/2050/p/3877280.html
  + 首先设备中的1px和css中的1px不一定相等, 前者是物理像素后者是独立像素, 在现代高分屏幕(1080x1920)上通常一个独立像素由多个物理像素表示
  + window.devicePixelRatio 表示1个独立像素所占用的物理像素数量
  + 浏览器默认有一个 `layout viewport`, 通常来说大小为`980px`(css对应的px宽度)
  + `visual viewport` 是浏览器的可视宽度, 在移动设备上通常小于`layout viewport`, 这也是为什么有些页面会在手机上出现横向滚动条
  + `ideal viewport`是移动端理想的viewport, 例如iphoneX该值为`350px`, 它的宽度约等于屏幕的实际宽度
  + 使用以下标签`<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">` 可以使得`layout viewport`的大小等于`ideal viewport`, 一般出现在专门对移动端做过适配的页面.
* `window.innerWidth`和 `document.documentElement.clientWidth`的区别
  + ??? https://zhuanlan.zhihu.com/p/37031348
  ```
  	
 	// 设备屏幕的宽高，可用宽高都包含在该对象中
	document.screen / screen
	
	// 如果缩放原地爆炸 以下结论不一定成立（浏览器有差异）
	
	// 浏览器视窗宽高 不包括滚动条
	document.documentElement.clientWidth
	document.documentElement.clientHeight
	
	// 浏览器视窗宽高 包括滚动条
	window.innerWidth
	window.innerHeight
  ```
* 请解释 CSS 动画和 JavaScript 动画的优缺点。
  + CSS定制自由度差，但比较方便。JS自由度大，但需要代码开发。
  + CSS动画更流畅，JS易导致页面掉帧。
  + CSS兼容性差
* `requestAnimationFrame`是什么, 有什么作用
  + 用来代替`setTimeout`实现动画, 于requestAnimationFrame的功效只是一次性的，所以若想达到动画效果，则必须连续不断的调用requestAnimationFrame，就像我们使用setTimeout来实现动画所做的那样, 但是它的性能相对来说更好, 原因如下
    + 会把每一帧中的所有DOM操作集中起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率，一般来说，这个频率为每秒60帧。
    + 在隐藏或不可见的元素中，requestAnimationFrame将不会进行重绘或回流，这当然就意味着更少的的cpu，gpu和内存使用量
* `package.json`中库的版本号`^`和`~`的区别
  ```json
  "dependencies": {
    "bluebird": "^3.3.4",
    "body-parser": "~1.15.2"
  }
  ```
  - 1.15.2对应就是MAJOR,MINOR,PATCH：1是marjor version；15是minor version；2是patch version。
  - MAJOR：这个版本号变化了表示有了一个不可以和上个版本兼容的大更改。
  - MINOR：这个版本号变化了表示有了增加了新的功能，并且可以向后兼容。
  - PATCH：这个版本号变化了表示修复了bug，并且可以向后兼容。
  - 波浪符号（~）：他会更新到当前minor version（也就是中间的那位数字）中最新的版本。放到我们的例子中就是：body-parser:~1.15.2，这个库会去匹配更新到1.15.x的最新版本，如果出了一个新的版本为1.16.0，则不会自动升级。波浪符号是曾经npm安装时候的默认符号，现在已经变为了插入符号。
  - 插入符号（^）：这个符号就显得非常的灵活了，他将会把当前库的版本更新到当前major version（也就是第一位数字）中最新的版本。放到我们的例子中就是：bluebird:^3.3.4，这个库会去匹配3.x.x中最新的版本，但是他不会自动更新到4.0.0。
* 什么是跨域资源共享 (CORS)？它用于解决什么问题？
  + Same-origin(同源) : 资源路径的协议、域名以及端口号与当前域一致
  + `<script>``<img>``<iframe>``<link>``<video>``<audio>``等带有`src`属性的标签默认支持跨域
  + 不同源的document或者js(例如iframe中的js)想要读取或者操作当前document将受到限制
  + 禁止Ajax发起跨域请求， 实际上请求会发起， 只不过返回响应会被浏览器拦截。
  + Ajax跨域请求(注意是请求不是响应， 响应会被拦截)不能携带本网站Cookie
  + 跨域方式
  ```js
    // 1. JSONP
    利用`<script><img><iframe>`标签默认跨域的特征, 所以该方式只支持get方法
    与服务器约定好， 让服务器返回一个`script`并在其中回调页面中的函数， 页面所需的数据作为调用参数
    
    // html
    // <script type="text/javascript" src="http://a.com/index.js">
    //  function showTable(tableData){
    //   需要ajax获取服务器端表单数据
    // }
    // </script>
    // <script type="text/javascript" src="http://b.com/remote.js"></script> 
    // 当然也可以用 document.createElement('script') 动态生产script
    
    
    // 服务器端返回的remote.js脚本
    showTable ({ name: 'Fred', age: 22 })
    
    // 2.  Cross-Origin Resource Sharing(CORS)
    // 在跨域服务器中的响应头部中加入`Access-Control-Allow-Origin`表示服务器允许哪些域可以访问该资源
    // Access-Control-Allow-Origin: <origin> | *
    // 包括该字段之外， 还有`Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, `Access-Control-Max-Age` 等等字段配合达到更强大的效果
    
    // 3. 父子域之间还可以考虑 `document.domain`,  `location.hash`等方式
    
    // 4. 还可以靠同源服务器代理（转发）请求。
  ```
* `doctype`(文档类型) 的作用是什么 / 浏览器标准模式 (standards mode) 、几乎标准模式（almost standards mode）和怪异模式 (quirks mode) 之间的区别是什么
  - DOCTYPE 主要用来引用该标记文档的 Document Type Definition (DTD), 以此告诉浏览器用什么样的解析规则来解析该文档.
  - 因为HTML4.01基于SGML(Standard Generalized Markup Language), 所以DOCTYPE需要对DTD进行引用
    + 4.01 严格模式 `<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">`
    + 4.01 过渡模式 `<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"  "http://www.w3.org/TR/html4/loose.dtd">`       + 4.01 frameset模式 `<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Frameset//EN"  "http://www.w3.org/TR/html4/frameset.dtd">`
  - HTML5并不基于SGML，所以不需要引用DTD，所以直接写为  `<!DOCTYPE html>`
  - 以上都属于浏览器的`standards mode`， 但是如果没有DOCTYPE或者DOCTYPE语法错误会让浏览器进入`quirks mode`
  - 在“标准模式”(standards mode) 页面按照 HTML 与 CSS 的定义渲染，而在“怪异模式(quirks mode) 模式”中则尝试模拟更旧的浏览器的行为。 一些浏览器（例如，那些基于 Mozilla 的 Gecko 渲染引擎的，或者 Internet Explorer 8 在 strict mode 下）也使用一种尝试于这两者之间妥协的“近乎标准”(almost standards) 模式，实施了一种表单元格尺寸的怪异行为，除此之外符合标准定义
  ```
    1. 盒模型不同
      `standard mode`是 `content-box` 
      `quirks mode`是 `border-box`
    2. 行内元素的宽高
      `standard mode`不能设置
      `quirks mode`是可以设置 
  ```
* 如果页面使用 'application/xhtml+xml' 会有什么问题吗？
  - 这是服务器http返回头部中的? xhtml 语法要求严格，必须有head、body 每个dom 必须要闭合。空标签也必须闭合。例如`<img />, <br/>, <input />`等。另外要在属性值上使用双引号。一旦遇到错误，立刻停止解析，并显示错误信息。如果页面使用'application/xhtml+xml',一些老的浏览器会不兼容。
* 使用 `data-` 属性的好处是什么？
  - 自定义的data Chrome中可以用dataset来取得, 比如 data-house-id="13"  xx.dataset.houseId = 13
* 请描述 `cookies`、`sessionStorage` 和 `localStorage` 的区别。
  - 都是同源的, 同时都保存在客户端
  - 可储存大小
    + cookies 大小不超过4k
    + sessionStorage, localStorage 大小限制可以达到5M甚至更多
  - 过期时间长短
    + cookie 在设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭
    + sessionStorage 数据在当前浏览器窗口关闭后自动删除, 在同一session间共享（https://github.com/lmk123/blog/issues/66）
    + localStorage 存储持久数据，浏览器关闭后数据不丢失除非主动删除数据
  - 是否传递
    + 同时每次http请求都会自动带上cookie
    + sessionStorage, localStorage 则只在本地保存
* 请解释 `<script>`、`<script async>` 和 `<script defer>` 的区别。
  - https://segmentfault.com/q/1010000000640869
  - `<script>` 同步加载, 加载完成后立即执行
  - `<script async>` 异步加载, 加载立即执行, 不考虑多个script的先后顺序
  - `<script defer>` 异步加载, 执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成, 但是执行会按照声明script的顺序.
* 为什么通常推荐将 CSS `<link>` 放置在 `<head></head>` 之间，而将 JS `<script>` 放置在 `</body>` 之前？你知道有哪些例外吗 / 什么是 FOUC (无样式内容闪烁)？你如何来避免 FOUC？
  - https://www.zhihu.com/question/309982596
  - css加载放HEAD防止FOUC(flash of unstyled content), 但是如果css过大可能导致白屏时间过长可以考虑放一部分非首屏可见元素的css在末尾.
  - js加载放末尾防止白屏时间过长. 但一些统计类js, 或是要网页面中添加内容的js可以考虑放在开头.
* 什么是渐进式渲染 (progressive rendering)？
  - https://stackoverflow.com/questions/33651166/what-is-progressive-rendering
  - 目的:尽快让用户看到内容
  - 手段: 图片懒加载, 可视化内容优先(Server Side Rendering, SSR 只返回首屏可视化部分的html，已由服务器端渲染好)
* 什么`mobile first`, 为什么需要`mobile first`
    ```
        // 区别在于`mobile-first`首先考虑移动端布局然后在这个基础上在添加以及改写样式来适配桌面端
        
        /* mobile-first */ 
        // This applies from 0px to 600px
        body {
          background: red;
        }

        // This applies from 600px onwards
        @media (min-width: 600px) {
          body {
            background: green;
          }
        }
        
        /* desktop-first */ 
        // This applies from 600px onwards
        body {
          background: green;
        }

        // This applies from 0px to 600px
        @media (max-width: 600px) {
          body {
            background: red;
          }
        }
    ```
    - 这样做的原因是: 通常移动端的布局会更加简单, 移动端优先的方式有利于更好地复用以及简化代码
* CSS 中类 (classes) 和 ID 的区别。
  - id用来标记一个 类用来标记很多个  
  - id优先级更高
* 请问 "resetting" 和 "normalizing" CSS 之间的区别？你会如何选择，为什么？
  - https://www.jianshu.com/p/a7b9e2d20b73
  - 目的都是
  - CSS重置更激进, 对浏览器的默认样式进行了一些重置, 一般是通过`* 通配符选择器`来达到目的, 最终使得CSS样式有一个统一的基准
  - CSS一般化修复了浏览器的自身bug并保持浏览器的一致性, 宗旨是保护有用的浏览器默认样式而不是完全去掉它们
  - `reset`的缺点在于浏览器调试工具中大段大段的继承链, 样式调试变得复杂
  - `normalized`保护了有价值的默认值(这样不需要为所有的排版元素都添加样式), 同时修复了浏览器的bug (这往往超出了Reset所能做到的范畴).
* 请解释浮动 (Floats) 及其工作原理。
  - 强制block, 同时跳出文本流浮动元素从网页的正常流动中移出，但保留了部分的流动性，会影响其他元素的定位（比如文字会围绕着浮动元素）。这一点与绝对定位不同，绝对定位的元素完全从文档流脱离
  - 如果浮动元素的父元素只包含浮动元素，那么该父元素的高度会坍塌为0，我们可以通过清除（clear）从浮动元素后到父元素关闭前之间的浮动来修复这个问题(clear: both | left | right)
  - 把浮动元素的父元素属性设置为overflow: auto或overflow: hidden,会使其内部子元素形成BFC，并且父元素会扩张自己，使其能够包围它的子元素
* 描述`z-index`和叠加上下文是如何形成的。
  - https://juejin.im/post/5b876f86518825431079ddd6
  - https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/
* 请描述 BFC(Block Formatting Context) 及其如何工作。
  - http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html
  - 消边距折叠 https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing
* 列举不同的清除浮动的技巧，并指出它们各自适用的使用场景。
  - 末尾添加一个空div作为兄弟：`<div style="clear: both;"/>`
  - 给父元素添加一个伪类
  ```css
    .clearfix::after {
	    content: '',
	    display: block;
	    clear: both;
    }
  ```
  - 父元素添加overflow: hidden | auto 或其他能够构成BFC的属性
* 请解释 CSS sprites，以及你要如何在页面或网站中实现它。
  - 为了节省http请求数量, 提高网页性能, 将多张素材图片合并到一张大图中
  - 可维护差, 每次改动都需要图片
* 你用过栅格系统 (grid system) 吗？如果使用过，你最喜欢哪种？
  - css3 grid https://www.zhangxinxu.com/wordpress/2018/11/display-grid-css-css3/
  - 自制 grid https://www.zcfy.cc/article/how-to-build-a-responsive-grid-system-zell-liew
* 请写一个简单的幻灯效果页面。
  - TODO 
* 什么是`rem`和`em`有什么区别
  - https://yanhaijing.com/css/2017/09/29/principle-of-rem-layout/
  - em作为font-size的单位时，其代表父元素的字体大小，em作为其他属性单位时，代表自身字体大小
  - rem作用于非根元素时，相对于根元素字体大小；rem作用于根元素字体大小时，相对于其出初始字体大小
  - rem可以用来做弹性布局, 通过设置根元素的字体大小来作为其他元素大小的单位.
  - 但更好的方案是 vw —— 视口宽度的 1/100；vh —— 视口高度的 1/10
* 水平居中的方式
  - 对于行内元素, 可以使用为其容器添加 `text-align: center`
  - 对于有宽度的块级元素, 可以为其自身添加 `margin: 0 auto`
  - 为其自身添加 `position: relative; left: 50%; transform: translate(-50%, 0)` 也可以
  - 为其容器添加 `display: flex; justify-content: center;` 也可以
* 垂直居中的方式
  ```css
  	/* 对于单行文本 */
	.wrapper {
		height: 100px;
		line-height: 100px;

	}
	
	/* 对于块级元素 */
  	.wrapper {
		display: relative
	}
	.wrapper > child {
		position: absolute;
		top: 0;
		bottom: 0;
		margin: auto;
	}
	
	/* 对于块级元素 */
	.wrapper {
		display: relative
	}
	.wrapper > child {
		position: absolute;
		top: 50%;
		transform: translate(0, -50%);
	}
  ```
* 有哪些行内替换元素, 他们有什么特点, 和行内元素有什么区别
  - 例如`img`, `input`根据标签的属性显示内容, 可以设置四个方向上的`padding, margin`以及`width, height`
  - 行内元素例如`span`, `a`竖直方向上的`padding-top、padding-bottom、margin-top、margin-bottom`无效果, 水平方向有效果
  - 实际上行内元素的`padding-top、padding-bottom`从视觉上确实是撑开了(例如添加背景), 但实际上并不对周围元素产生影响
* `vertical-align`有什么作用
  - http://www.cnblogs.com/hykun/p/3937852.html
  - https://zhuanlan.zhihu.com/p/28626505
* 你用过媒体查询，或针对移动端的布局/CSS 吗？
  ```js
      if (window.matchMedia("(min-width: 400px)").matches) {
        /* The viewport is at least 400 pixels wide */
      } else {
        /* The viewport is less than 400 pixels wide */
      }
  ```
  ```css
      @media (min-width: 900px) {
          article {
            padding: 1rem 3rem;
          }
      }
  ```
* 如何优化网页的打印样式？
  - https://blog.csdn.net/xujie_0311/article/details/42271273
  ```css
      // 以下两种方式都可以
      <link rel="stylesheet" type="text/css" media="screen" href="screenstyles.css" />
	  <style type="text/css">
		@import url("screenstyles.css") screen;
		 @media print {
		     /* 打印时使用的样式放置在此 *
		}
	  </style>
  ```
* 在书写高效 CSS 时会有哪些问题需要考虑？

* 请解释浏览器是如何判断元素是否匹配某个 CSS 选择器？
* 请描述伪元素 (pseudo-elements) 及其用途。
* 请解释你对盒模型的理解，以及如何在 CSS 中告诉浏览器使用不同的盒模型来渲染你的布局。
* 请解释 ```* { box-sizing: border-box; }``` 的作用, 并且说明使用它有什么好处？
* 请罗列出你所知道的 display 属性的全部值
* 请解释 inline 和 inline-block 的区别？
* 如何去除inline-block元素间的间隙
  - 产生间隙的原因是标签间的空格
  - 去除相邻的标签之间的空格
  ```html
     <div>
        <a href="">
        链接1</a><a href="">
        链接2</a><a href="">
        链接3</a><a href="">
        链接4</a>
    </div>
  ```
  - 设置`font-size`或者`letter-spacing`或者`word-spacing`
  - 使用margin负值
* 请解释 relative、fixed、absolute 和 static 元素的区别
* CSS 中字母 'C' 的意思是叠层 (Cascading)。请问在确定样式的过程中优先级是如何决定的 (请举例)？如何有效使用此系统？
* 你在开发或生产环境中使用过哪些 CSS 框架？你觉得应该如何改善他们？
* 请问你有尝试过 CSS Flexbox 或者 Grid 标准规格吗？
* 为什么响应式设计 (responsive design) 和自适应设计 (adaptive design) 不同？
* 你有兼容 retina 屏幕的经历吗？如果有，在什么地方使用了何种技术？
* 请问为何要使用 `translate()` 而非 *absolute positioning*，或反之的理由？为什么？
* JavaScript中对象的属性定义与赋值的区别
    - http://www.cnblogs.com/ziyunfei/archive/2012/10/31/2738728.html
    ```js
        const person = {}
        person.age = 12 // 赋值
        Object.defineProperty(person, "name", propDesc) // 定义

        // 背景知识
        // js有三种属性
        // 1. named data properties: 有确定的值, 最常见
        // 2. named accessor properties: 通过getter和setter读取以及赋值, 实际值得存放位置不确定, 甚至可以是个临时计算量
        // 3. internal properties: 内部属性, 比如每个对象都有一个内部属性[[Prototype]]
        // 下文只讨论前两种
        
        // 每个属性有四个特征（括号里是使用定义创建属性时, 如果忽略某个特征, 该特征的默认值）
        // named data properties: 
        //    1. `value`(undefined): 值
        //    2. `writable`(false): 决定值是否可改变
        
        // named accessor properties: 
        //    1. `get`(undefined): 取值时将调用
        //    2. `set`(undefined): 赋值时讲调用
        
        // shared properties:
        //    1. `enumerable`(false): 是否可枚举,如果为false,这个属性在某些操作下是不可见的,比如for...in和Object.keys()
        //    2. `configurable`(false): 是否可配置, 如果一个属性是不可配置的, 则该属性的所有特性(除了[[Value]])都不可改变.
        
                
        // 有啥区别？？
        
        //  `Object.defineProperty(person, "name", propDesc)` 
        //  若`name`属性不存在, 则创建该属性, 具体特征由参数`propDesc`对象指明, 如果没有指明的直接按照上述括号中的默认值
        //  若已存在且属性可配置, 则修改`propDesc`对象中指明的属性特征, 其他特征保持不变
        //  但若属性是不可配置的, 则只能修改`value`, 如果`writable`是false, 则连value都不可修改
        
        //  person.age = 12 // 赋值
        //  一般来讲, 若`age`不存在, 该操作等效于`Object.defineProperty(person, "age", { value: 12, writable: true, enumerable: true, configurable: true })`
        //  若存在, 则直接修改值
        //  但有以下情况赋值不生效:
        //    1. 如果在原型链上存在一个名为P的只读属性(只读的数据属性或者没有setter的访问器属性
        //    2. 如果在原型链上存在一个名为P的且拥有setter的访问器属性:则调用这个setter.
        //    3. `age`已经存在且`age`属性存在`set`方法, 则具体行为依`set`而定
        
    ```
* JavaScript 内部属性
    - 由JavaScript引擎内部使用的属性,不能通过JavaScript代码直接访问到
    - 例如 `[[Prototype]]`能用`Object.getPrototypeOf()`和`Object.setPrototypeOf()`来读取和修改
* 手写函数防抖和函数节流
	```js
		// 防抖和节流多用于页面滚动事件等短时间内触发频率较高的事件, 关键是限制函数调用次数来提高性能
		// 不同点在于 防抖只在用户操作结束之后触发一次 例如input输入验证
		// 而节流在用户操作过程中仍不断触发, 但以较低的频率 例如js动画
		
		// 防抖
		function debounce (handle, delay) {
			let timeout 
			return function () {
				clearTimeout(timeout)
				timeout = setTimeout (handle, delay)
    		}
		}

		// 节流
		function throttle (handle, threshold) {
			let timeout
			let previousDate
			return function () {
				clearTimeout(timeout)
				let currentDate = new Date()

				if (currentDate - previousDate >= threshold) {
					handle()
					previousDate = 0
				} else {
					timeout = setTimeout (handle, threshold)
				}
				if (!previousDate) previousDate = currentDate
		    }
		}

        // 尝试去掉注释 滑动页面 观察效果
        // window.onscroll = debounce (e => { console.log(1) }, 1000)
        // window.onscroll = throttle (e => { console.log(1) }, 1000)
	```
* 请解释事件代理 (event delegation)。
  - 事件委托是将事件监听器添加到父元素，而不是每个子元素单独设置事件监听器。当触发子元素时，事件会冒泡到父元素，监听器就会触发
  - 这样一来只需要为父元素编写事件处理函数, 有利于统一管理, 同时同类元素无需分别添加事件处理
* JavaScript的sort方法内部使用的什么排序？
  - Chrome 查过10用快排否则用插入排序， Firefox 是归并
* 请解释 JavaScript 中 `this` 是如何工作的。
  - new Person(), Person函数内部this 指向新对象
  - apply, call, bind强行绑定
  - fuck.something(), somthing函数内部this指向fuck
  - 不符合上述, this指向全局, 若为strict模式, 则指向undefined
  - 箭头函数中this永远指向创建该函数时上下文中的this, 且无法修改
* 请解释原型继承 (prototypal inheritance) 的原理。
  - 每个对象都有原型, 对象访问属性时, 若未找到, 会沿着原型链一直往上找
  - 根据这个原理, 我们可以让被继承对象成为继承对象的原型即可实现继承的效果
* 你怎么看 AMD vs. CommonJS？
  - 
* 请解释为什么接下来这段代码不是 IIFE (立即调用的函数表达式)：`function foo(){ }();`.
  * 要做哪些改动使它变成 IIFE?
* 描述以下变量的区别：`null`，`undefined` 或 `undeclared`？
  * 该如何检测它们？
* 什么是闭包 (closure)，如何使用它，为什么要使用它？
* 请举出一个匿名函数的典型用例？
* 你是如何组织自己的代码？是使用模块模式，还是使用经典继承的方法？
* 请指出 JavaScript 宿主对象 (host objects) 和原生对象 (native objects) 的区别？
* 请指出以下代码的区别：`function Person(){}`、`var person = Person()`、`var person = new Person()`？
* `.call` 和 `.apply` 的区别是什么？
* 请解释 `Function.prototype.bind`？
* 在什么时候你会使用 `document.write()`？
* 请指出浏览器特性检测，特性推断和浏览器 UA 字符串嗅探的区别？
* 请尽可能详尽的解释 Ajax 的工作原理。
* 使用 Ajax 都有哪些优劣？
* 请解释 JSONP 的工作原理，以及它为什么不是真正的 Ajax。
* 你使用过 JavaScript 模板系统吗？
  * 如有使用过，请谈谈你都使用过哪些库？
* 请解释变量声明提升 (hoisting)。
* 请描述事件冒泡机制 (event bubbling)。
* "attribute" 和 "property" 的区别是什么？
* 为什么扩展 JavaScript 内置对象不是好的做法？
* 请指出 document load 和 document DOMContentLoaded 两个事件的区别。
* `==` 和 `===` 有什么不同？
* 请解释 JavaScript 的同源策略 (same-origin policy)。
* 如何实现下列代码：
```javascript
[1,2,3,4,5].duplicator(); // [1,2,3,4,5,1,2,3,4,5]
```
* 什么是三元表达式 (Ternary expression)？“三元 (Ternary)” 表示什么意思？
* 什么是 `"use strict";` ? 使用它的好处和坏处分别是什么？
* 请实现一个遍历至 `100` 的 for loop 循环，在能被 `3` 整除时输出 **"fizz"**，在能被 `5` 整除时输出 **"buzz"**，在能同时被 `3` 和 `5` 整除时输出 **"fizzbuzz"**。
* 为何通常会认为保留网站现有的全局作用域 (global scope) 不去改变它，是较好的选择？
* 为何你会使用 `load` 之类的事件 (event)？此事件有缺点吗？你是否知道其他替代品，以及为何使用它们？
* 请解释什么是单页应用 (single page app), 以及如何使其对搜索引擎友好 (SEO-friendly)。
* 你使用过 Promises 及其 polyfills 吗? 请写出 Promise 的基本用法（ES6）。
* 使用 Promises 而非回调 (callbacks) 优缺点是什么？
* 使用一种可以编译成 JavaScript 的语言来写 JavaScript 代码有哪些优缺点？
* 你使用哪些工具和技术来调试 JavaScript 代码？
* 你会使用怎样的语言结构来遍历对象属性 (object properties) 和数组内容？
* 请解释可变 (mutable) 和不变 (immutable) 对象的区别。
  * 请举出 JavaScript 中一个不变性对象 (immutable object) 的例子？
  * 不变性 (immutability) 有哪些优缺点？
  * 如何用你自己的代码来实现不变性 (immutability)？
* 请解释同步 (synchronous) 和异步 (asynchronous) 函数的区别。
* 什么是事件循环 (event loop)？
  * 请问调用栈 (call stack) 和任务队列 (task queue) 的区别是什么？
* 解释 `function foo() {}` 与 `var foo = function() {}` 用法的区别

#### <a name='testing-questions'>测试相关问题：</a>

* 对代码进行测试的有什么优缺点？
* 你会用什么工具测试你的代码功能？
* 单元测试与功能/集成测试的区别是什么？
* 代码风格 linting 工具的作用是什么？

#### <a name='performance-questions'>效能相关问题：</a>

* 你会用什么工具来查找代码中的性能问题？
* 你会用什么方式来增强网站的页面滚动效能？
* 请解释 layout、painting 和 compositing 的区别。

#### <a name='network-questions'>网络相关问题：</a>

* 为什么传统上利用多个域名来提供网站资源会更有效？
* 请尽可能完整得描述从输入 URL 到整个网页加载完毕及显示在屏幕上的整个流程。
* Long-Polling、Websockets 和 Server-Sent Event 之间有什么区别？
* 请描述以下 request 和 response headers：
  * Diff. between Expires, Date, Age and If-Modified-...
  * Do Not Track
  * Cache-Control
  * Transfer-Encoding
  * ETag
  * X-Frame-Options
* 什么是 HTTP method？请罗列出你所知道的所有 HTTP method，并给出解释。
* 请解释 HTTP status 301 与 302 的区别？


*问题：`foo`的值是什么？*
```javascript
var foo = 10 + '20';

// '1020'
```

*问题：如何实现以下函数？*
```javascript
add(2, 5); // 7
add(2)(5); // 7

function add (a, b) {
	if (b === undefined) {
		return b => a + b
  }
	return a + b
}
```

*问题：下面的语句的返回值是什么？*
```javascript
"i'm a lasagna hog".split("").reverse().join("");

// "goh angasal a m'i"
```

*问题：`window.foo`的值是什么？*
```javascript
( window.foo || ( window.foo = "bar" ) );

// "bar"
```

*问题：下面两个 alert 的结果是什么？*
```javascript
var foo = "Hello";
(function() {
  var bar = " World";
  alert(foo + bar);
})();
alert(foo + bar);
```

*问题：`foo.x`的值是什么？*
```javascript
var foo = { n: 1 };
var bar = foo;
foo.x = foo = { n: 2 };

// foo 
// { n: 2 }
//
// bar
// { n: 1, x: { n: 2 } }
```

## 来源
https://github.com/h5bp/Front-end-Developer-Interview-Questions

https://zhuanlan.zhihu.com/p/54397576
