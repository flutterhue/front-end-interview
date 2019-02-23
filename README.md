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
  ```html
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
    // 输出如下
    * `[1] initial readyState:loading`
    * `[2] readyState:interactive`
    * `[2] DOMContentLoaded`
    * `[3] iframe onload`
    * `[4] readyState:complete`
    * `[4] img onload`
    * `[4] window onload`
  `document.readyState` 在 `DOMContentLoaded` 前一刻变为 `interactive`，这两个事件可以认为是同时发生。
  `document.readyState` 在所有资源加载完毕后（包括 `iframe` 和 `img`）变成 `complete`，我们可以看到`complete`、 `img.onload` 和         `window.onload` 几乎同时发生，区别就是 `window.onload` 在所有其他的 `load` 事件之后执行
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
  
* 为什么传统上利用多个域名来提供网站资源会更有效？
  - 另一方面某些资源服务器可以避免不必要的Cookie的传递
  - 浏览器对单域名的并行数量有限


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
  + `<script>``<img>``<iframe>``<link>``<video>``<audio>`等带有`src`属性的标签默认支持跨域
  + 不同源的document或者js(例如iframe中的js)想要读取或者操作当前document将受到限制
  + 禁止Ajax发起跨域请求， 实际上请求会发起， 只不过返回响应会被浏览器拦截。
  + Ajax跨域请求不能携带本网站Cookie
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
  - 条例清晰, 利于维护
  
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
  
  
* 为什么通常推荐将 CSS `<link>` 放置在 `<head></head>` 之间，而将 JS `<script>` 放置在 `</body>` 之前？你知道有哪些例外吗 / 什么是 FOUC (Flash of Unstyled Content)？你如何来避免 FOUC？
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
  - 选择器越通用效率越低, 四种选择器的解析速度由快到慢依次是：ID、class、tag和universal(*)
  - 避免过度约束, 例如ul#main-navigation { } ID已经是唯一的，不需要Tag来标识，这样做会让选择器变慢
  - 避免后代选择符, 下面这个选择器是很低效的： html body ul li a { }
  - 使用紧凑的语法, 避免不必要的重复
  

* 请解释浏览器是如何判断元素是否匹配某个 CSS 选择器？
    ```
    CSS选择器的解析是从右向左解析的。若从左向右的匹配，发现不符合规则，需要进行回溯，会损失很多性能。若从右向左匹配，先找到所有的最右节点，对于每一个节点，向上寻找其父节点直到找到根元素或满足条件的匹配规则，则结束这个分支的遍历。两种匹配规则的性能差别很大，是因为从右向左的匹配在第一步就筛选掉了大量的不符合条件的最右节点（叶子节点），而从左向右的匹配规则的性能都浪费在了失败的查找上
    ```
    
    
* 请描述伪元素 (pseudo-elements) 及其用途。
  - 单冒号是伪类 例如`:hover, :visited, :checked`, 表示的是元素的状态或者结构特点, 例如鼠标在其上, 浏览过, 选择了, 第3个div等等
  - 双冒号是微元素 例如`::before, ::after`, 表示的是一种虚拟的元素，CSS把它当成普通HTML元素, 之所以叫伪元素，就因为它们在文档树或DOM中并不实际存在
  - 伪元素可用来制作图标, 消除浮动等
  
  
* 请解释你对盒模型的理解，以及如何在 CSS 中告诉浏览器使用不同的盒模型来渲染你的布局。
  - 浏览器默认盒模型是 content-box 也就是说 盒宽 = 左右margin + 左右border + 左右padding + width (宽度即内容宽度) 同理高
  - 设置```* { box-sizing: border-box; }```之后 盒宽 = 左右margin + width (width = 左右border + 左右padding + 内容宽)
  
  
* 请罗列出你所知道的 display 属性的全部值
  - block 
  ```
    block元素会独占一行，多个block元素会各自新起一行。默认情况下，block元素宽度自动填满其父元素宽度。
    block元素可以设置width,height属性。块级元素即使设置了宽度,仍然是独占一行。
    block元素可以设置margin和padding属性。
  ```
  - inline
  ```
    inline元素不会独占一行，多个相邻的行内元素会排列在同一行里，直到一行排列不下，才会新换一行，其宽度随元素的内容而变化。
    inline元素设置width,height属性无效。
    inline元素的margin和padding属性，水平方向的padding-left, padding-right, margin-left, margin-right都产生边距效果；但竖直方向的padding-top, padding-bottom, margin-top, margin-bottom不会产生边距效果, 但背景有效果. 也就是说竖直方向上位置不影响其他元素, 但是视觉效果上会影响, 例如
    https://jsbin.com/kitoxigaca/1/edit?html,css,output
  ```
  - inline-block
  ```
    就是将对象呈现为inline对象，多个inline对象会被排列在同一行内. 但同时我们可以想设置block一样设置其宽高, margin, padding，且像block一样会影响其他元素
    https://jsbin.com/kazezujofe/edit?html,css,output
  ```
  - flex
  ```
  几个常用的
  1. 给容器添加的
    display: flex
    flex-direction: column, row （默认row） 也就是主轴方向
    justify-content: center 主轴居中
    align-items: center 副轴
    
  2. 给孩子的
    flex: flex-grow, flex-shrink 和 flex-basis
    flex-grow: 0 不放大, 其他数字的话根据其他孩子的该值按比例放大
    flex-shrink: 0 不缩小, 其他数字的话根据其他孩子的该值按比例缩小
    flex-basis: 占主轴的宽度, 或者auto表示占宽度按照节点本来大小算
    flex: auto (1 1 auto) | none (0 0 auto)
  ```
  - grid
  ```
  
  ```
  - none
  ```
    不显示, 也不占空间
  ```
  - 忽略 table
  

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
  - 默认是static
  - relative 是相对于static位置调整 left, right, top, bottom 但不影响其他元素布局
  - absolute 相对非static的父类或祖先位置进行调整, 而且已经脱离标准文档流
  - fixed 是相对浏览器, 而且已经脱离标准文档流


* CSS 中字母 'C' 的意思是叠层 (Cascading)。请问在确定样式的过程中优先级是如何决定的 (请举例)？如何有效使用此系统？
  - 首先把所有样式分成三类, `!important`, 行内(即标签内部的), 内联(style标签中)和外联(另外的css文件中)
  - 在每一类中, 从0开始，id选择器+100，一个属性选择器、class或者伪类+10，一个元素选择器，或者伪元素+1，通配符+0
  - 优先取权重高的, 权重相同, 取后定义的, 这就是层叠的来源.
  
  
* 为什么响应式设计 (responsive design) 和自适应设计 (adaptive design) 不同？
  - 前者是一个网页, 根据设备改变而发生布局的变化, 本质上这种检测是客户端的css以及js来完成的
  - 检测不同的设备, 并分别对应不同的网页, 这种检测是服务器端根据请求的不同而返回对应不同的网页来实现的.
  
  
* 请问为何要使用 `translate()` 而非 *absolute positioning*，或反之的理由？为什么？
  - 使用 transform 时，可以让 GPU 参与运算，动画的 FPS 更高。
  - translate不会引起浏览器的重绘和重排
  - 使用 position 时，最小的动画变化的单位是 1px，而使用 transform 参与时，可以做到更小（动画效果更加平滑）
  
* 讲express框架的设计思想
  - TODO
* 讲express的中间件系统是如何设计的
  - TODO
  
* 讲nodejs的eventEmitter的实现
  ```js
	  class EventEmitter {
	  constructor () {
	    this.eventListenersMap = {}
	    this.on = this.addListener
	    this.off = this.removeListener
	  }

	  addListener = (event, listener) => {
	    if (this.eventListenersMap[event]) {
		  this.eventListenersMap[event].push(listener)
	    } else {
	      this.eventListenersMap[event] = [listener]
	    }
	   return listener
	  }

	  removeListener = (event, listener) => {
	    this.eventListenersMap[event] && 
		(this.eventListenersMap[event] = 
		this.eventListenersMap[event].filter(each => each != listener))
	  }

	 emit = (event, ...args) => {
	   this.eventListenersMap[event] && 
	   this.eventListenersMap[event].forEach(
	     listener => listener.apply(event, args)
	   )
	 }
	}
  ```
  
  
* 聊一聊JS中的`event loop`
  - 首先要明确JavaScript本身是单线程的, 但它却是异步的, JavaScript实现异步的机制就是`event loop`
  ```
      ref: https://juejin.im/post/5c148ec8e51d4576e83fd836
	1.「宏任务」、「微任务」都是队列，一段代码执行时，会先执行宏任务中的同步代码。
      + 宏任务包括`整体代码script`, `setTimeout`, `setInterval`, IO, UI渲染
      + 微任务包括`Promise`, Object.observe、MutationObserver
	2. 进行第一轮事件循环的时候会把全部的js脚本当成一个宏任务来运行。
	3. 如果执行中遇到setTimeout之类宏任务，那么就把这个setTimeout内部的函数推入「宏任务的队列」中，下一轮宏任务执行时调用。
	4. 如果执行中遇到 promise.then() 之类的微任务，就会推入到「当前宏任务的微任务队列」中，在本轮宏任务的同步代码都执行完成后，依次执行所有的微任务。
	5. 第一轮事件循环中当执行完全部的同步脚本以及微任务队列中的事件，这一轮事件循环就结束了，开始第二轮事件循环。
	6. 第二轮事件循环同理先执行同步脚本，遇到其他宏任务代码块继续追加到「宏任务的队列」中，遇到微任务，就会推入到「当前宏任务的微任务队列」中，在本轮宏任务的同步代码执行都完成后，依次执行当前所有的微任务。
	开始第三轮，循环往复...
    
    可验证
    
    setTimeout(() => console.log('timeout'), 0);
    Promise.resolve(true).then(() => {console.log('promise')})
    console.log('directly log')

    // Chrome 72
    // console output
    VM745:3 directly log
    VM745:2 promise
    undefined
    VM745:1 timeout
  ```
  
  
* 浏览器的事件循环和nodejs事件循环的区别
  ```
  ref: https://segmentfault.com/a/1190000013660033
  以上聊的实际上是浏览器中的事件循环, nodejs时间循环主要由libuv实现
  其中 Macrotask 主要分化为一下六步骤
    1. timers：执行满足条件的setTimeout、setInterval回调。
    2. I/O callbacks：是否有已完成的I/O操作的回调函数，来自上一轮的poll残留。
    3. idle，prepare：可忽略
    4. poll：等待还没完成的I/O事件，会因timers和超时时间等结束等待。
    5. check：执行setImmediate的回调。
    6. close callbacks：关闭所有的closing handles，一些onclose事件。
  此外还有 MicroTask 和 process.nextTick, 对应 MicroTask Queue 和 NextTick Queue, 和上面合并之后最终过程为:
  
    清空当前循环内的Timers Queue，清空NextTick Queue，清空Microtask Queue。
    清空当前循环内的I/O Queue，清空NextTick Queue，清空Microtask Queue。
    清空当前循环内的Check Queu，清空NextTick Queue，清空Microtask Queue。
    清空当前循环内的Close Queu，清空NextTick Queue，清空Microtask Queue。
    进入下轮循环。
    
    可以看出，nextTick优先级比promise等microtask高。setTimeout和setInterval优先级比setImmediate高。 这些在浏览器中都不存在
  ```
  
  
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
    
    
* 使用es5实现es6的class
  ```js
    // 工具函数 
    // 代替es6中的Object.create
    // 返回一个空对象其原型为传入的prototype
    function createPolyfill (prototype) {
      function F () {}
      F.prototype = prototype
      return new F()
    }

    // 1.1 父类构造函数的定义
    function Animal (age) {
      if (!(this instanceof Animal)) throw new Error('Add new !')
      this.age = age
    }

    // 1.2 父类方法的定义
    Animal.prototype.logAge = function () {
      console.log(this.age)
    }

    // 2.1 子类构造函数的定义
    function Cat (name, age) {

      if (!(this instanceof Cat)) throw new Error('Add new !')
      Animal.call(this, age)
      this.name = name
    }

    // 2.2 继承声明 (必须在子类方法定义前)
    Cat.prototype = createPolyfill(Animal.prototype)
    Cat.prototype.constructor = Cat

    // 2.3 子类方法的定义
    Cat.prototype.logName = function () {
      console.log(this.name)
    }
    
    // 以上代码等效于以下ES6代码, 可自行验证
    class Animal {
       constructor (age) {
         this.age = age 
       }

       logAge () { console.log(this.age) }
    }
    
    class Cat extends Animal {
       constructor (name, age) {
         super(age)
         this.name = name 
       }

       logName () { console.log(this.name) }
    }
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
  
  
* 你怎么看 AMD vs. vs. CMD vs. CommonJS？
  - 都是JavaScript模块化的解决方案
  - CommonJS主要是Node在遵循的规范, 作用于后端
  - CommonJS在前端不适用的主要原因是前端模块需要从网络上下载, 必须采用异步的方式引入.
  - 其他两个是前端依赖解决方案, AMD是依赖前置, CMD是依赖就近. 
  - CMD好像是国人玉伯开发的
  
  
* 请解释为什么接下来这段代码不是 IIFE (立即调用的函数表达式)：`function foo(){ }();`.
  ```
    function foo(){ }();  // 前半部分被解释器解析为一个函数, 后半部分是括号, 直接报错. 解决方案:
    1. (function foo(){ })()
    2. (function foo(){ }())
  ```
  
  
* 描述以下变量的区别：`null`，`undefined` 或 `undeclared`？
  - `null` 表示空对象, 但是使用`typeof null` 会返回 "object", 判断null直接用 === 即可
  - `undefined` 表示 var, let, const 声明了但没有赋值的变量, 连同 `undeclared` 变量一起, typeof 都会返回 'undefined'
  - 由于`undeclared`的变量直接引用会报错, 所以不能直接判断
  ```js
  // 以下方式可区分后两者
  try{
	a // 要检测的变量名
	console.log('已定义')
  } catch (e){
	console.log('未定义')
  }
  ```


* `typeof`, `instanceof` 的作用和原理
  ```js
  // JS基本类型有七种 Number, String, Boolean, Null, Undefined, Symbol, Object
  
  
  // ------------- typeof ----------------
  
    console.log(typeof null)  // object
    console.log(typeof undefined)  // undefined
    console.log(typeof 12.23)  // number
    console.log(typeof true)  // boolean
    console.log(typeof 'str')  // string
    console.log(typeof Symbol(3))  // symbol
    console.log(typeof (() => {}))  // function
    console.log(typeof /re/)  // object
    console.log(typeof {})  // object
    console.log(typeof [])  // object

  // 可以看到有一些特殊的情况
  //   1. null 被判断为对象
  //   2. (() => {}) 被判断为函数(实际上也是特殊的对象)
    
    
  // ------------- instanceof ---------------
  
  // 原理是沿着左边对象的原型链不断地寻找右边的Prototype
  // 所以右边必须是一个非箭头函数的可调用对象 i.e 普通函数

  function instanceofPolyfill (leftValue, rightValue) {
    if (leftValue === null) return false
    let rightProto = rightValue.prototype // 取右表达式的 prototype 值
    leftValue = Object.getPrototypeOf(leftValue) // 取左表达式的__proto__值
    while (true) {
    	if (leftValue === null) return false
      if (leftValue === rightProto) return true
      leftValue = Object.getPrototypeOf(leftValue)
    }
  }


  // 判断类型可以用另外一种方式 `Object.prototype.toString`, 对各种内置类型都有较好的支持

  Object.prototype.toString.call(1) // "[object Number]"
  Object.prototype.toString.call('hi') // "[object String]"
  Object.prototype.toString.call({a:'hi'}) // "[object Object]"
  Object.prototype.toString.call([1,'a']) // "[object Array]"
  Object.prototype.toString.call(true) // "[object Boolean]"
  Object.prototype.toString.call(() => {}) // "[object Function]"
  Object.prototype.toString.call(null) // "[object Null]"
  Object.prototype.toString.call(undefined) // "[object Undefined]"
  Object.prototype.toString.call(Symbol(1)) // "[object Symbol]"

  ```


* 为什么`Object instanceof Function`并且`Function instanceof Object`
  - 按照JS的规范, 所有的对象都是Object的实例, 所有函数都是Function的实例
  - 函数也是对象, 也就是说Function本身是一个对象, 所以`Function instanceof Object`
  - 同时Object本身也是一个函数, 所以`Object instanceof Function`
  ```js
    // https://stackoverflow.com/a/46895582/5817139
    // Object ---> Function.prototype ---> Object.prototype ---> null
    // Function ---> Function.prototype ---> Object.prototype ---> null

    console.log(Object.__proto__ === Function.prototype) // true
    console.log(Object.__proto__.__proto__ === Object.prototype) // true
    console.log(Object.__proto__.__proto__.__proto__ === null) // true

    console.log(Function.__proto__ === Function.prototype) // true
    console.log(Function.__proto__.__proto__ === Object.prototype) // true
  ```
  
* `'hello'`, `String('hello')`, `new String('hello')`有区别吗?
  ```js
  let str1 = 'hello'
  let str2 = String('hello')
  let str3 = new String('hello')
  
  console.log(typeof str1)    // string
  console.log(typeof str3)    // object
  console.log(str1 === str2)  // true
  console.log(str1 === str3)  // false
  ```
  
  
* 什么是闭包 (closure)，如何使用它，为什么要使用它？
  - 函数内部定义的函数作为外部函数的返回值返回后, 借助返回的内部函数仍能访问到外部函数内的变量
  - 构造私有变量
  ```js
  function createPrivate (value) {
    return {
      get () { return value },
      set (newValue) { value = newValue }
    }
  }
  ```
  - 配合IIFE实现模块
  ```
  const module1 = (function () {
    // define function, variable
    
    // return obj with defined function, variable insided
    return { }
  }())
  ```


* 深拷贝
```js
let loop = {}
loop.self = loop

function Building(type) {
  this.type = type
}
Building.prototype.canMove = function() {
  return false
}
function School(type) {
  if (!(this instanceof School)) {
    throw new Error('WHERE IS UR NEW')
  }

  Building.call(this, type)
  this.people = [{ name: 'xx' }, { name: 'yy' }]
  this.say = function() {
    console.log('school')
  }
}

function helper() {}
helper.prototype = Building.prototype
School.prototype = new helper()

let school = new School('A+')

// 1. 不支持循环引用
// 2. 不拷贝函数
// 3. 不拷贝原型链
// 4. 支持的内置对象有限, 例如正则对象/re/就不支持
// 5. 不拷贝Symbol
function deepCopy1(targetObj) {
  return JSON.parse(JSON.stringify(targetObj))
}

// 1. 非深拷贝函数
// 2. 原型链上属性以及函数被移到对象本身上
// 3. 支持的内置对象有限, 例如正则对象/re/就不支持
// 4. 前拷贝Symbol
function deepCopy2(targetObj) {
  function notObject(obj) {
    return obj === null || typeof obj !== 'object'
  }
  function helper(targetObj) {
    if (record.has(targetObj)) return record.get(targetObj)

    let re = Array.isArray(targetObj) ? [] : {}
    record.set(targetObj, re)
    for (const each in targetObj) {
      re[each] = notObject(targetObj[each])
        ? targetObj[each]
        : helper(targetObj[each])
    }
    return re
  }

  if (notObject(targetObj)) return targetObj
  let record = new WeakMap()
  return helper(targetObj)
}

// 当然 如果钻牛角尖的话 这个问题还有更多方向可以考虑
//   1. enumerable为false 考虑使用 Object.getOwnPropertyDescriptors() + Object.create() 进行浅拷贝
//   2. 由于for-in不循环Symbol, 且上面进行的是浅拷贝, 如果要深拷贝的话, 可以专门使用 Object.getOwnPropertySymbols() 进行遍历
//   3. 同时系统也提供了一部分API 例如`MessageChannel`等等.

// https://juejin.im/post/5ad6b72f6fb9a028d375ecf6
// https://zhuanlan.zhihu.com/p/33489557
// http://jerryzou.com/posts/dive-into-deep-clone-in-javascript/
```
  
  
* 请指出 JavaScript 宿主对象 (host objects) 和原生对象 (native objects) 的区别？
  - 宿主对象是由运行时环境（浏览器或 Node）提供，比如window、XMLHTTPRequest等等
  - 原生对象是由 ECMAScript 规范定义的 JavaScript 内置对象，比如String、Math、RegExp、Object、Function等等。
  
  
* 请指出以下代码的区别：`function Person(){}`、`var person = Person()`、`var person = new Person()`？
  - `function Person() {}` 是函数定义
  - `var person = Person()` 是普通的函数调用
  - `var person = new Person()` 是使用`new`操作符构建Person对象的实例
  
  
* `.call` 和 `.apply` 的区别是什么？
  - 两者第一个参数都是指定上下文this, `call`剩下的参数数量不定, 会被原样传入调用函数, 而`apply`剩下的参数是一个数组, 会将该数组中的每个变量作为参数传入调用函数.
  - 据说某些JS引擎上 `call` 的性能更好
  - https://www.zhihu.com/question/61088667
  
  
* 请说明.forEach循环和.map()循环的主要区别，它们分别在什么情况下使用
  - `forEach`主要用于依靠循环中的每一个值来进行一些操作, 本身不改变原数组也不返回新数组
  - `map`是通过传入函数来声明旧数组和新数组的关系, 通过这种方式来构建并返回新数组
  
  
* 请解释 `Function.prototype.bind`？
  ```
  // 对普通函数
  function kk (a, b, c) { console.log(this, a, b, c) }
  var jj = kk.bind({}, 1, 2)
  jj(3, 4, 5)
  
  // {} 1 2 3
  
  // 对箭头函数
  var kk = (a, b, c) => { console.log(this, a, b, c) }
  var jj = kk.bind({}, 1, 2)
  jj(3, 4, 5)
  
  // Window 1 2 3
  ```
  
  
  
* 能否写一个bind函数的polyfill
  ```
    function bindPolyfill (fn, context, ...args) {
        return function (...args2) {    
            fn.apply(context, args.concat(args2))
        }
    }
  ```

  
* 在什么时候你会使用 `document.write()`？
  - ??? 直接运行 会导致页面中 body 下内容被清空并改写为传入字符
  
  
* 请指出浏览器特性检测，特性推断和浏览器 UA 字符串嗅探的区别？
  - 功能检测包括确定浏览器是否支持某段代码，以及是否运行不同的代码（取决于它是否执行），以便浏览器始终能够正常运行代码功能，而不会在某些浏览器中出现崩溃和错误。例如
    ```js
    if ('geolocation' in navigator) {
      // 可以使用 navigator.geolocation
    } else {
      // 处理 navigator.geolocation 功能缺失
    }
    ```
  - 特性推断：功能推断与功能检测一样，会对功能可用性进行检查，但是在判断通过后，还会使用其他功能，因为它假设其他功能也可用，也就是根据一个特性的存在推断另一个特性是否存在。问题是，推断是假设并非事实，而且可能导致可维护性的问题。
  - UA字符串：这是一个浏览器报告的字符串，它允许网络协议对等方（network protocol peers）识别请求用户代理的应用类型、操作系统、应用供应商和应用版本。它可以通过navigator.userAgent访问。 然而，这个字符串很难解析并且很可能存在欺骗性。例如，Chrome 会同时作为 Chrome 和 Safari 进行报告。因此，要检测 Safari，除了检查 Safari 字符串，还要检查是否存在 Chrome 字符串。不要使用这种方式。
  - 个人感觉UA字符串和UA头一样, 都是不靠谱的, 例如爬虫中可以随意设置UA头. 只不过UA字符串即navigator.userAgent是客户端浏览器自己识别客户机器的结果, 而UA头主要放在HTTP请求中.
  
  
* 使用 Ajax 都有哪些优劣？
  - 优势主要集中在:
    + 减轻了服务器压力
    + 无刷新更新网页数据, 优化了用户体验
  - 劣势
    + 搜索引擎支持较弱
    + 不安全, 暴露了服务器更多接口
    + 不支持浏览器前进,后退功能, 网页状态无法保留 可以通过前端路由进行解决
    
    
* 请解释变量声明提升 (hoisting)。
   ```js
   // 1. 变量声明提升, 注意是声明提升
   console.log(x)
   var x = 3
   console.log(x)
   // undefined
   // 3
   原因是因为只提升了var声明, 也就是说, 以上代码等效于
   var x
   console.log(x)
   x = 3
   console.log(x)
   
   // 2. 函数提升
   // 写函数有两种方式, 其提升规则存在区别
   
   // 方式1
   go()
   function go () { console.log('go') }
   // go
   // 这种方式下, 整个函数连带定义都被提升
   
   // 方式2
   go()
   var go = function () { console.log('go') }
   // Uncaught TypeError: go is not a function
   // 这种方式下类似是变量声明提升, 所以报错
   ```
   
   
* 请描述事件冒泡机制 (event bubbling)。
  + 现代浏览器中先捕获后冒泡, 也就是说标准DOM事件触发以后, 从根节点开始到target节点进行传播, 这个过程叫事件捕获, 然后从target节点传回根节点, 这个过程叫事件冒泡
  + `addEventListener(event, listener, useCapture)` 第三个参数默认为`false`, 表示不监听事件捕获, 监听事件冒泡. 
  + 如果此时给一个节点同时添加了两个监听事件, 一个捕获一个冒泡, 那么捕获监听器首先触发, 然后才是冒泡
  
  
* "attribute" 和 "property" 的区别是什么？
 - attribute是HTML标签上的特性，它的值只能够是字符串, node.attributes是一个类数组对象, property是DOM中的属性，是JavaScript里的对象
 - 如果一个非默认也就是自定义的属性添加的html的某个tag中, 只能通过`node.attributes.属性名`查到
 - 如果对于一个有默认值的属性, 例如 input 标签中的 value, 除非显式修改过, 否则查询node.attributes.value返回undefined, 只有查询node.value才能返回空字符串.
 - property能够从attribute中得到同步, 也就是说attribute更新property会随之更新, 反过来则不行
 - 但是更改property和attribute上的任意值，都会将更新反映到HTML页面中
 ```
    我的理解是, node.attributes是对于html内容的真实映射, html中node挂了几个标签都会反映到node.attributes中, 例如
    <input /> 的 node.attributes 是 `NamedNodeMap {length: 0}`
    而node.属性名 是对DOM标准的实现, 例如input 应该有一个 `value` 的property, 所以不管写不写, input.value 都存在值.
 ```
 - https://www.cnblogs.com/elcarim5efil/p/4698980.html
 
 
* 为何你会使用 `load` 之类的事件 (event)？此事件有缺点吗？你是否知道其他替代品，以及为何使用它们？
  - load 触发比较慢, 需要等DOM以及相关资源全部加载完成之后才触发
  - 而 DOMContentLoaded 的触发无需等待样式表, 图片等多媒体资源以及iframe等子框架的加载
 
 
* 请解释什么是单页应用 (single page app), 以及如何使其对搜索引擎友好 (SEO-friendly)。
  - 开局一个HTML, 更新全靠AJAX
  - SEO的话需要SSR或者其他预渲染工具


* 使用 Promises 而非回调 (callbacks) 优缺点是什么？
  - 条例清晰, 类似线性代码执行顺序, 防止回调地狱
  - 当你无法确定请求是否真的是异步时, callback可能同步调用或者异步调用调用, 而Promise中的then总是晚于Promise中executor的执行的
  ```
  function callback (data) {
    console.log( a );
  }

  var a = 0;

  mayBeAsnyc(callback);
  a++;
  
  // 无法确定到底a 是1 还是 2
  ```
  - callback可能调用太晚, 可以用`race`确保回调发生及时
  - 确保只发生一次, 因为到了 Resolved 状态就不再改变了


* 你会使用怎样的语言结构来遍历对象属性 (object properties) 和数组内容？
  - 对象
  ```
  var child = Object.create({ inParentProp1: -1, inParentProp2: -2 })
  child.inChildProp1 = 1
  child.inChildProp2 = 2
  Object.defineProperty(child, 'hiddenChildProp', { value: 3 })
  
  // 1. for ... in 遍历原型链上所有enumerable
  for (const each in child) {console.log(each)}
  //  VM105:1 inChildProp1
  //  VM105:1 inChildProp2
  //  VM105:1 inParentProp1
  //  VM105:1 inParentProp2
  
  // 2. Object.keys 只包括该对象自身的enumerable
  Object.keys(child)
  // ["inChildProp1", "inChildProp2"]
  
  // 3. Object.getOwnPropertyNames 包括自身的所有, 包括非enumerable
  Object.getOwnPropertyNames(child)
  // ["inChildProp1", "inChildProp2", "hiddenChildProp"]
  ```
  - 数组
  ```
  var list = ['a', 'b', 'c', 'd']
  
  list.forEach((each, index) => { console.log(index + ': ' + each) })
  // 0: a
  // 1: b
  // 2: c
  // 3: d
  // undefined (返回值)
  
  for (const each of list) { console.log(each) }
  // VM3833:1 a
  // VM3833:1 b
  // VM3833:1 c
  // VM3833:1 d
  // undefined (返回值)
  
  for (let i = 0; i < list.length; ++i) {} // 
  ```


* 请解释可变 (mutable) 和不变 (immutable) 对象的区别。
  - 可变对象 在创建之后是可以被改变的, 
  - 不可变的例如 string 和 number 从设计之初就是不可变(Immutable)
  - 不可变那其实是保持一个对象状态不变，这样做的好处是使得开发更加简单，可回溯，测试友好，减少了任何可能的副作用。但是，每当你想添加点东西到一个不可变(Immutable)对象里时，它一定是先拷贝已存在的值到新实例里，然后再给新实例添加内容，最后返回新实例。相比可变对象，这势必会有更多内存、计算量消耗
  - 尽量写纯函数, 使用const, 或利用 `Object.freeze()`, `Object.seal()`


* 你会用什么工具测试你的代码功能？
  - 只用过jest
  
  
* 单元测试与功能/集成测试的区别是什么？
  - 前者的测试是以项目中的小模块为单位
  - 后者的测试时以项目小模块拼接后的整体来联合测试
  
  
* 你会用什么方式来增强网站的页面滚动效能？
  - 节流
  
* 如何实现浏览器访问url返回后图片下载
  - 服务器设置响应头中`Content-Disposition=attachement;filename=xxxx`

* 前端路由的实现思路？
  - 考虑兼容性可以使用 `hashchange`, 缺点在于url之后总是带着一个#符号. e.g. www.host.com/#/index.html
  - 不考虑兼容性的话可以使用 `pushState`, `replaceState`两个浏览器历史操作API以及`popstate`事件
  - TODO具体实现
  
* Get 和 Post 区别是什么
  - GET请求可以被添加到书签中，也可保存在浏览器历史记录中，请求可以被浏览器缓存，POST不能
  - GET请求收到URL长度限制，同时由于传递的参数完全暴露在URL中所以数据长度也受限制, POST不会
  - GET请求幂等, 也就是说多次执行的结果和只执行一次的结果完全相同
  - GET产生一个TCP数据包；POST产生两个TCP数据包。
  ```
   对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；
   而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据）。
   据研究，在网络环境好的情况下，发一次包的时间和发两次包的时间差别基本可以无视。而在网络环境差的情况下，两次包的TCP在验证数据包完整性上，有非常大的优点。 并不是所有浏览器都会在POST中发送两次包，Firefox就只发送一次。
  ```


* 如何避免浏览器缓存get请求，以便达到每次get请求都能获取最新的数据
  - 在请求的头部中加上`cache-control: no-cache`
  - 在请求的url尾部加上'?r=随机数' e.g. www.host.com/somewhere.html?r=821293


* 聊聊浏览器中关于缓存的问题
  - 首先, 根据是否向服务端发出请求, 可以将缓存分为强制缓存和对比缓存. 两者可以同时存在, 强制缓存优先级更高.
  - 强制缓存值得是浏览器在缓存未失效的情况下, 会忽略请求而直接使用本地的缓存. 相关响应头有Expire 和 Cache-Control. 其中前者兼容HTTP1.0, 而后者是HTTP1.1 提出的新规范
  - Expires的值为服务端返回的到期时间，即下一次请求时，请求时间小于服务端返回的到期时间，直接使用缓存数据, 但是当客户端和服务器端时间存在误差时, 会导致问题.
  - 所以HTTP1.1提出了Cache-Control, 两者可以同时存在, 此时以Cache-Control为准, 这样既保证做主要是为了保证兼容性. Cache-Control 可以用以下取值
    + private:             客户端可以缓存
    + public:              客户端和代理服务器都可缓存（前端的同学，可以认为public和private是一样的）
    + max-age=xxx:   缓存的内容将在 xxx 秒后失效（相对时间, 解决了客户端服务器端时差问题）
    + no-cache:          需要使用对比缓存来验证缓存数据（后面介绍）
    + no-store:           所有内容都不会缓存，强制缓存，对比缓存都不会触发
  - 如果当Cache-Control没有触发(过期, 浏览器刷新), 则应用对比缓存. 相关请求/响应头也有两套
    + Last-Modified: 服务器在响应请求时，告诉浏览器资源的最后修改时间, 以便浏览器在下一次请求时填入`If-Modified-Since`中
    + If-Modified-Since: 再次请求服务器时，通过此字段通知服务器上次请求时，服务器返回的资源最后修改事件
    + 如果服务器发现资源有变化, 则返回200, 返回新的资源. 否则返回304, 告诉浏览器使用已有缓存即可.
    + Etag / If-None-Match（优先级高于Last-Modified / If-Modified-Since）
    + Etag: 即所谓的资源唯一标识码, 有点类似文件的MD5, 用来判断资源是否发送变化, 以便浏览器在下一次请求时填入`If-None-Match`
  - 输入URL, 如果是第一次访问, 则获取新的资源并存入缓存, 否则先触发强制缓存再触发对比缓存.
  - 刷新过程, 忽略强制缓存直接触发对比缓存
  - Ctr + F5 刷新, 清空本地缓存, 强制重新下载
 
  
* 什么是 HTTP method？请罗列出你所知道的所有 HTTP method，并给出解释。
  - get
  - post
  - put
  - delete
  
* HTTP 断点续传怎么做到的
  - 如果服务器响应头中包含`Accept-Ranges:byte`, 说明当前资源支持范围请求 (因为范围请求是HTTP1.1才开始支持的)
  - 确定双端都支持范围请求后, 在请求头部中假如 `Ranges: 0 - Content-Length`, 例如已经下载了1000bytes的资源, 想继续下载余下部分, 可以设置`Ranges: bytes=1000-`
  - 请求之后响应头中添加`Content-Range: bytes 100-999/1000`来表示实际返回的范围和资源总长度
  - 除上述情况之外, 还需要考虑一点, 资源可能在几天之后更新了. 为了解决这个问题, 这里要考虑两个和缓存有点的头部标识
    + ETag：当前文件的一个验证令牌指纹，用于标识文件的唯一性。
    + Last-Modified：标记当前文件最后被修改的时间。
  - 在 If-Range 中填入 ETag 或者 Last-Modified 中的任意一个即可, 如果资源没变, 服务器返回206, 断点续传, 否则返回200, 重新下载。
  - 但是If-Range 必须配合 Range 来使用, 否则会被服务器忽略.
  - 如果随便输入一个不合理的Range, 例如总长度为1000的数据, 你输入 2000, 就会返回 416, Range Not Satisfiable.
  - https://juejin.im/post/5b555f055188251af25700aa
  
* 讲讲TCP协议 连接三次握手, 断开四次握手
  ```
  https://github.com/jawil/blog/issues/14
  https://www.zhihu.com/question/24853633/answer/115173386
  
  TCP A                                                TCP B

  1.  CLOSED                                               LISTEN

  2.  SYN-SENT    --> <SEQ=100><CTL=SYN>               --> SYN-RECEIVED

  3.  ESTABLISHED <-- <SEQ=300><ACK=101><CTL=SYN,ACK>  <-- SYN-RECEIVED

  4.  ESTABLISHED --> <SEQ=101><ACK=301><CTL=ACK><DATA> --> ESTABLISHED
                       (Data is optional)
		       ...
               
  0.  CLOSED                                               LISTEN

  1.  FIN_WAIT_1  --> <SEQ=500><ACK=384><CTL=FIN>       --> SYN-RECEIVED

  2.  FIN_WAIT_2 <-- <SEQ=384><ACK=501><CTL=SYN,ACK>   <-- SYN-RECEIVED
  
          ... A表示自己所有数据传输完成, 已经不发送数据了, 但是B仍可以向A发送数据
  
  3.  FIN_WAIT_2 <-- <SEQ=?><CTL=FIN>                  <-- LAST_ACK

  4.  TIME_WAIT --> <SEQ=?><ACK=?><CTL=ACK>            --> Closed
  
  5.  2ms later timeout  
  
  6.  Closed                                               Closed
  ```
  
  
* Websocket了解吗
  - webSocket和http一样，同属于应用层协议。它最重要的用途是实现了客户端与服务端之间的全双工通信，当服务端数据变化时，可以第一时间通知到客户端, 如果是http协议, 需要重新建立一个新的请求来发送信息.
  - http只能由客户端发起，而webSocket是双向的
  - webSocket传输的数据包相对于http而言很小，很适合移动端使用
  - 没有同源限制，可以跨域共享资源
  
  

* Long-Polling、Websockets 和 Server-Sent Event 之间有什么区别？


* 请描述以下 request 和 response headers：
  * Diff. between Expires, Date, Age and If-Modified-...
  * Do Not Track
  * Cache-Control
  * Transfer-Encoding
  * ETag
  * X-Frame-Options
  

  
* HTTP 状态码了解多少
  - 1xx ：1开头的状态码表示临时的响应
  - 2xx ：请求成功
  - 3xx ：请求被重定向
  - 4xx ：请求错误，表明客户端发送的请求有问题
  - 5xx ：服务器错误，表明服务端在处理请求时发生了错误
  ```
  一些常见的:
    301 ： Moved Permanently 客户端请求的文档在其他地方，新的URL在location头中给出
    304 ： Not Modified 客户端有缓存的文档并发出了一个条件性的请求（一般是提供If-Modified-Since头表示客户端只想到指定日期后再更新文档）。服务器告诉客户，原来缓存的文档还可以继续使用。
    400 ： Bad Request 请求出现语法错误
    401 ： Unauthorized 访问被拒绝，客户端试图未经授权访问受密码保护的页面
    403 ： Forbidden 资源不可用。服务器理解客户的请求，但拒绝处理它。通常由于服务器文件或目录的权限设置导致。
    404 ： Not Found 无法找到指定位置的资源。
    405 ： Method Not Allowed 请求方法（GET、POST、PUT等）对指定的资源不适用，用来访问本资源的HTTP方法不被允许。
    500 ： Internal Server Error 服务器遇到了意料之外的情况，不能完成客户端的请求。
    502 ： Bad Gateway 服务器作为网管或者代理时收到了无效的响应。
    503 ： Service Unavailable 服务不可用，服务器由于维护或者负载过中未能应答。
    504 ： Gateway Timeout 网关超时， 作为代理或网关的服务器不能及时的应答。
  ```


* 请解释 HTTP status 301 与 302 的区别？
  - 301 表示资源永久转移, 浏览器以后都用返回头中的新地址即可
  - 302 表示暂时转移（也许只是短暂的维护所以使用了一个临时的链接）, 浏览器以后还是用老地址


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

* 关于树的节点数, 度, 叶子节点的关系
  - 首先, 树中度指的是该节点子节点的数量, 叶节点度为0
  - 由于除根节点外每个节点都是由父节点指向的, 所以 树的节点数 = 所有节点的度数 + 1
  - 根据上一条我们有  叶子节点 + 度为1的节点 + 度为2的节点 = 1 * 度为1的节点 + 2 * 度为2的节点 + 1
  - 化简后 对于二叉树由 叶节点 = 度为2的节点 + 1

## 来源
https://github.com/h5bp/Front-end-Developer-Interview-Questions

https://zhuanlan.zhihu.com/p/54397576

https://zhuanlan.zhihu.com/p/32565654
