## 前端面试题收录

#### 常见问题

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
  + https://www.zhihu.com/question/20474326

* 请写一个简单的幻灯效果页面。

* 请解释 CSS 动画和 JavaScript 动画的优缺点。
  + CSS定制自由度差，但比较方便。JS自由度大，但需要代码开发。
  + CSS动画更流畅，JS易导致页面掉帧。
  + CSS兼容性差
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
  - https://github.com/lmk123/blog/issues/66
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

* CSS 中类 (classes) 和 ID 的区别。
  - id用来标记一个 类用来标记很多个  
  - id优先级更高
* 请问 "resetting" 和 "normalizing" CSS 之间的区别？你会如何选择，为什么？
  - https://www.jianshu.com/p/a7b9e2d20b73
* 请解释浮动 (Floats) 及其工作原理。
  - 强制block, 同时跳出文本流浮动元素从网页的正常流动中移出，但保留了部分的流动性，会影响其他元素的定位（比如文字会围绕着浮动元素）。这一点与绝对定位不同，绝对定位的元素完全从文档流脱离
  - 如果浮动元素的父元素只包含浮动元素，那么该父元素的高度会坍塌为0，我们可以通过清除（clear）从浮动元素后到父元素关闭前之间的浮动来修复这个问题(clear: both | left | right)
  - 把浮动元素的父元素属性设置为overflow: auto或overflow: hidden,会使其内部子元素形成BFC，并且父元素会扩张自己，使其能够包围它的子元素
* 描述`z-index`和叠加上下文是如何形成的。
  - https://juejin.im/post/5b876f86518825431079ddd6
  - https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/
* 请描述 BFC(Block Formatting Context) 及其如何工作。
  - http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html
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
* 你最喜欢的图片替换方法是什么，你如何选择使用。
* 你会如何解决特定浏览器的样式问题？
* 如何为有功能限制的浏览器提供网页？
  * 你会使用哪些技术和处理方法？
* 有哪些的隐藏内容的方法 (如果同时还要保证屏幕阅读器可用呢)？
* 你用过栅格系统 (grid system) 吗？如果使用过，你最喜欢哪种？
* 你用过媒体查询，或针对移动端的布局/CSS 吗？
* 你熟悉 SVG 样式的书写吗？
* 如何优化网页的打印样式？
* 在书写高效 CSS 时会有哪些问题需要考虑？
* 使用 CSS 预处理器的优缺点有哪些？
  * 请描述你曾经使用过的 CSS 预处理器的优缺点。
* 如果设计中使用了非标准的字体，你该如何去实现？
* 请解释浏览器是如何判断元素是否匹配某个 CSS 选择器？
* 请描述伪元素 (pseudo-elements) 及其用途。
* 请解释你对盒模型的理解，以及如何在 CSS 中告诉浏览器使用不同的盒模型来渲染你的布局。
* 请解释 ```* { box-sizing: border-box; }``` 的作用, 并且说明使用它有什么好处？
* 请罗列出你所知道的 display 属性的全部值
* 请解释 inline 和 inline-block 的区别？
* 请解释 relative、fixed、absolute 和 static 元素的区别
* CSS 中字母 'C' 的意思是叠层 (Cascading)。请问在确定样式的过程中优先级是如何决定的 (请举例)？如何有效使用此系统？
* 你在开发或生产环境中使用过哪些 CSS 框架？你觉得应该如何改善他们？
* 请问你有尝试过 CSS Flexbox 或者 Grid 标准规格吗？
* 为什么响应式设计 (responsive design) 和自适应设计 (adaptive design) 不同？
* 你有兼容 retina 屏幕的经历吗？如果有，在什么地方使用了何种技术？
* 请问为何要使用 `translate()` 而非 *absolute positioning*，或反之的理由？为什么？
* 手写函数防抖和函数节流
```
  
```
* 请解释事件代理 (event delegation)。
* 请解释 JavaScript 中 `this` 是如何工作的。
* 请解释原型继承 (prototypal inheritance) 的原理。
* 你怎么看 AMD vs. CommonJS？
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
