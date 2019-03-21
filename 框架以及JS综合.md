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
  

* React 中 setState 是异步吗 ？
  - 实际上分情况, 有可能同步有可能异步, 生命周期函数中(例如componentDidMount)中以及合成事件中是异步的, 而在原生事件或者setTimeout中是同步的.
  - 而batchingStrategy中的 isBatchingUpdates用来控制是立即更新还是之后更新, 在生命周期函数以及合成事件中, 这些钩子被调用的时候的 isBatchingUpdates 都是 true, 这导致了此时调用的多次setStates结果被放进了dirtyComponents的列表中, 所以会出现连续两次`setState({count: this.state.count+1})`, count是加1的现象. 因为在dirtyComponents会在下一次更新前把所有的partialState合并, 由于此时前后两次this.state.count 的值都相同, 所以效果相当于1次.
  - 然而在原生或者setTimeout中, 此时的isBatchingUpdates已经是false, 所以setState相当于是执行了同步代码, 并不会采用批处理的方式.
  - 这个问题的关键在于React中一个概念叫Transaction, 它的作用是给任何方法套上两个wrapper, 每个wrapper对象都有initialize和close方法, 而对于`ReactDefaultBatchingStrategyTransaction`中的wrapper 的作用主要是在交易结束后将isBatchingUpdates设为false, 然后调用`flushBatchedUpdates`批量处理沉积的DirtyComponents.
  - https://github.com/MrErHu/blog/issues/20
  - https://undefinedblog.com/what-happened-after-set-state/
  - http://undefinedblog.com/understand-react-batch-update/
  - https://zhuanlan.zhihu.com/p/39512941


* Webpack 打包的原理
  - Webpack实际上原理很简单, 如果不考虑Chunks的话, 最终所有文件打包进同一文件, Webpack从入口文件开始分析依赖, 其中可能包括你import的其他js, css, 甚至图片. 每个类型的资源都会被当成模块, 经过相应的loader只有最终均被转化为合法的js代码. 然后最终的打包文件是一个IIFE, 其传入参数就是对象, 该对象的key是文件名value是由对应文件内的js代码构成的函数. 该对象被传入后, 函数体中定义了一个对象叫做installedModules对象用来缓存已经被加载的对象. 并定义了工具函数__webpack_require__来加载模块. 然后从入口函数开始, 一旦遇到新模块, 直接调用webpack_require进行加载 (如果已经缓存则直接返回).
  - 另外, 传入的对象中key对于的value是一个JS代码构成的函数. 更具体将, 是一个用eval包裹的函数, 而源代码则在eval包裹的字符串中. 用eval包裹的原因似乎是为了缓存SourceMap进而提高构建速度.
  - https://www.jianshu.com/p/e8ec61954748
  - https://zhuanlan.zhihu.com/p/37864523
  - https://juejin.im/post/5badd0c5e51d450e4437f07a


* Webpack 中Chunk的优势
  - 通俗来讲Chunk就是把最终打包成的单一JS文件分离出多个JS. 主要目的还是优化异步加载, 也即是说客户端可以按需加载代码而不是一口气加载所有的代码.
  - 合理使用的情况下可以优化页面加载时间.


* Tree shaking 的概念
  - 通常用于描述移除 JavaScript 上下文中的未引用代码(dead-code)。它依赖于 ES2015 模块系统中的静态结构特性，例如 import 和 export
  - 传统的DCE(dead code elimination)是主要是`代码不可到达`, `代码结果未被使用`, `变量只写不读`的三类代码消除. 传统编译型语言中由编译器完成, JS中由uglify工具完成.
  - 而ES6模块系统是静态分析的, 也就是说在预处理阶段就可以判断到底加载了哪些代码, TreeShaking就是通过这种方式减少不必要代码的引入, 再引入过程中只考虑实际要使用的那部分代码. 例如`import { functionA } from lib.js`, 假设functionB和functionA没有互相依赖关系但都是lib.js的导出函数, 那么在这种情况有tree shaking 的话只会引入functionA. 结合上面Webpack打包原理来看就是传入对象中于key`lib.js`对于的value只有`eval('function functionA() {...}')`


* Webpack 热更新HMR的原理
  - Webpack 热更新需要配合 webpack-dev-server 或者 webpack-hot-middleware 来实现.
  - 以webpack-dev-server为例:
    1. 首先对于webpack本身而言, 处于监听模式下他会对文件的变化重新打包, 并将事件通知给感兴趣的监听函数
    2. 然后对于webpack-dev-server, 其中间件 webpack-dev-middleware 就是所谓的监听函数, 它将webpack重新打包的代码储存在内存中.
    3. 另外实际上webpack-dev-server与客户端之间实际是有连接的(WebSocket长连接), 检测到变化之后他将通知webpack-dev-server的客户端模块, 此时只是通知, 具体内容仅仅是一个包含新模块的哈希, 方便之后HotModuleReplaceRuntime通过该哈希获取对应跟新的模块.
    4. 客户端模块的作用并不是更新模块, 而是通知webpack本身, 然后webpack自身的HotModuleReplaceRuntime会负责具体的模块更新问题. 它通过AJAX向webpack-dev-server发出请求, 然后webpack-dev-server通过JSONP的方式返回更新模块的具体内容.
    5. 如果过程失败将导致live reload 也就是刷新操作.
  - 通过上述过程可以看出, 实际上webpack-dev-server在整个热模块替换过程中起到的只是一个两段之间信息传递的作用, 他实际上并不参与具体的模块传输及更新. 这就是为什么 webpack-hot-middleware 也能完成这个工作, 只不过它基于EventSource而不是WebSocket.
  - 另一方面, HMR它实际上是可选功能, 如一个模块并没有在module.hot中定义模块更新之后的具体行为(没有HMR处理函数), 它将向依赖树的上层冒泡, 直至找到处理函数, 否则将自动刷新页面. 这也是为什么你用style-loader不需要任何配置就能自动热更新, 因为它内部实现已经包含了HMR处理函数, 而自己写的只有一行的`console.log(1)`改为`console.log(2)`时却会导致页面刷新. 你需要加入`module.hot.accept(errorHandler);`来进行模块自更新.
  - https://webpack.docschina.org/api/hot-module-replacement
  - https://zhuanlan.zhihu.com/p/30669007


* JS作用域链
  - 作用域的顶端肯定是全局作用域, 例如浏览器环境中的window即在全局作用域.
  -  作用域链的具体表现可以看做是一个栈, 全局作用域在栈底部
  - 然后每次函数创建的时候, 引擎会在函数内部挂上一个内部属性, 该属性将保存父环境的作用域链.
  - 每次函数执行过程中, 首先浅拷贝内部属性创建作用域链(注意浅拷贝复制, 也就是说如果重复调用, 属于父环境中的变量是会有影响的), 然后扫描函数内的所有形参, 函数, 变量声明创建新的作用域放在链顶(这里是创建, 也就是对于自身的作用域, 每次都是新的). 如果遇到变量, 则从顶部开始查找, 直至找到位于底部的全局作用域.
  ```js
  function pf () {
    let pv = 1
    return function ps() {
      pv++
      console.log(pv)
    }
  }
  let ps = pf()
  ps() // 2
  ps() // 3
  ps() // 4

  ps = pf()
  ps() // 2

  // 以上例子充分说明了我提到的最后一点, 就是函数的执行每次都会创建新的上下文, 但是对于它的上方作用域链, 实际上只是浅拷贝.
  ```
  - 网上的说法是EC(environment context)包含三个东西, this指针, 作用域链 和 VO (variable object)
    1. this指针就不说了, 依情况而定
    2. 作用域链主要是一个最终能追溯的全局作用域的链表
    3. VO 放的是当前作用域下声明的函数, 变量以及函数传入的arguments
    4. 另一个概念叫AO (activiation object), 当函数被调用, 则该函数的 VO 就是 AO (当然参数会被传入的arguments给注入)
  - 所以结合我上面的说法来看的话, 作用域链实际上就是一个个的VO串接而成的, 在栈顶的VO就是AO. 同一个函数多次调用, 每次AO不同但是AO以上的VO相同.
  

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
  - 一个值得一看的问题 (结合了promise 以及 async) https://segmentfault.com/q/1010000016147496/
  
  
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


* 能手动实现 Promise.all 和 Promise.race 吗? Promise 在中途取消有什么思路 ?
```js
EasyPromise.all = function (promises) {
  return new Promise((resolve, reject) => {
    const result = []
    let cnt = 0
    for (let i = 0; i < promises.length; ++i) {
      promises[i].then(value => {
        cnt++
        result[i] = value
        if (cnt === promises.length) resolve(result)
      }, reject)
    }
  })
}

EasyPromise.race = function (promises) {
  return new Promise((resolve, reject) => {
    for (let i = 0; i < promises.length; ++i) {
      promises[i].then(resolve, reject)
    }
  })
}

// 最简单的方式是直接返回一个永远 pending 的Promise
// 缺陷是内存泄漏

new Promise(r => r(1))
    .then(val => {
        console.log('执行到这里不想执行下去')
        return new Promise(() => {})
    }).then(val => {
        console.log('从这里往下面都是不该被执行的代码')
    })

// 另一个思路是利用 throw 扔出一个特殊的错误信号
// 下方的错误处理函数接收到此信号后不处理原样跑出
// 所以需要下放错误函数进行手动处理 比较麻烦
// 所以可以对这种方式进行封装, 类似的封装思想在于
// 对于then或者catch外围加一层判断, 如果是特殊信号
// 则原样抛出
// https://github.com/xieranmaya/blog/issues/5
function Break () {}
new Promise(r => r(1))
    .then(val => {
        console.log(2)
        throw new Break()
    }).then(val => {
        console.log(3)
    }, (reason) => {
        if (reason instanceof Break) throw reason
        console.log(reason)
    }).catch(reason => {
        console.log('break')
    })
```
 
  
* JavaScript中对象的属性定义与赋值的区别
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
    - http://www.cnblogs.com/ziyunfei/archive/2012/10/31/2738728.html
    
    
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
    
    
* JavaScript的sort方法内部使用的什么排序？
  - Chrome中数组规模超过10用快排否则用插入排序， Firefox是归并
  
  
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
//   4. 如果想支持更多原生对象, 可以使用lodash, 实现原理就是根据不同原生类型使用Object.prototype.toString判断之后再自行创建

// https://juejin.im/post/5ad6b72f6fb9a028d375ecf6
// https://zhuanlan.zhihu.com/p/33489557
// http://jerryzou.com/posts/dive-into-deep-clone-in-javascript/
```


* 用JavaScript的异步实现sleep函数
  ```js
  async function test() {
    console.log('Hello')
    await sleep(1000)
    console.log('world!')
  }

  function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms))
  }

  test()
  ```


* 请指出 JavaScript 宿主对象 (host objects) 和原生对象 (native objects) 的区别？
  - 宿主对象是由运行时环境（浏览器或 Node）提供，比如window、XMLHTTPRequest等等
  - 原生对象是由 ECMAScript 规范定义的 JavaScript 内置对象，比如String、Math、RegExp、Object、Function等等。
  
  
* `.call` 和 `.apply` 的区别是什么？
  - 两者第一个参数都是指定上下文this, `call`剩下的参数数量不定, 会被原样传入调用函数, 而`apply`剩下的参数是一个数组, 会将该数组中的每个变量作为参数传入调用函数.
  - 据说某些JS引擎上 `call` 的性能更好
  
  
* 请解释 `Function.prototype.bind`？
  ```js
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
  ```js
    function bindPolyfill (fn, context, ...args) {
        return function (...args2) {    
            fn.apply(context, args.concat(args2))
        }
    }
  ```

  
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

// Hello World
// Ref Error
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

* 下面两个的结果是什么
```js
console.log(typeof (new (class { class() { console.log(1) } } )))
// object

(function class () { console.log(1) })()
// Uncaught SyntaxError: Unexpected token class
```

* 实现一个LazyMan，可以按照以下方式调用:
```
LazyMan("Hank")输出:
Hi! This is Hank!
 
LazyMan("Hank").sleep(10).eat("dinner")输出
Hi! This is Hank!
//等待10秒..
Wake up after 10
Eat dinner~
 
LazyMan("Hank").eat("dinner").eat("supper")输出
Hi This is Hank!
Eat dinner~
Eat supper~
 
LazyMan("Hank").sleepFirst(5).eat("supper")输出
//等待5秒
Wake up after 5
Hi This is Hank!
Eat supper
 
以此类推。
```
```js
function LazyMan(name) {
	let hasInit = false
    let tasks1 = []
    let tasks2 = []
	function next () {
		if (tasks1.length !== 0) {
			tasks1.shift()()
        } else {
			if (!hasInit) {
    			console.log(`Hi This is ${name}!`)
				hasInit = true
            }
			if (tasks2.length !== 0) {
				tasks2.shift()()
            }
        }
    }

	let man = {
		sleepFirst: function (second) {
			tasks1.push(() => {
                setTimeout(() => {
                    console.log(`Wake up after ${second}`)
                    next()
                }, second * 1000)
            })
			return this
        },

		sleep: function (second) {
			tasks2.push(() => {
				setTimeout(() => {
					console.log(`Wake up after ${second}`)
					next()
				}, second * 1000)
            })
			return this
        },
		eat: function (kind) {
			tasks2.push(() => {
				console.log(`eat ${kind}`)
				next()
			})
			return this
		}
	}
	setTimeout(next)
	return man
}
```
