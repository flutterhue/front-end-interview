* display: none;、visibility: hidden;、opacity: 0;的区别
  - display: none; 
    1. 不占空间, 不渲染, 所以该属性一旦改变会引起浏览器重排
    2. 不被子类继承, 但是子类也不会显示
    3. 不会触发绑定事件 (显然)
    4. transition对它无效, 所以无法用transition做动画
  - visibility: hidden;
    1. 元素隐藏但不会消失, 也就是占着空间但是看不到, 所以一旦改变只会引起浏览器重绘
    2. 被子类继承, 但是子类可以通过设置visibility: visible;来达到可见的效果.
    3. 不触发绑定事件
    4. transition对它无效, 所以无法用transition做动画
  - opacity: 0;
    1. 元素透明度100%, 也就是占着空间但是看不到, 所以一旦改变只会引起浏览器重绘
    2. 会被子元素继承,但是子元素并不能通过opacity:1来达到可见的效果
    3. 能触发绑定的事件
    4. transition对它有效, 所以可以用transition做动画


* 请解释浮动及清除浮动的方法
  - 添加了浮动的元素会被强制当成block元素来进行渲染, 同时从网页的正常流动中部分脱离，但保留了一定的流动性，会影响其他元素的定位（比如文字会围绕着浮动元素）。这一点与绝对定位不同，绝对定位的元素完全从文档流脱离
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


* 请描述 BFC(Block Formatting Context) 及其如何工作。
  - 块级格式化上下文，它是指一个独立的块级渲染区域，只有Block-level BOX参与，该区域拥有一套渲染规则来约束块级盒子的布局，且与区域外部无关
  - 形成
    1. 根元素html 而不是body, 并且body设置overflow: hidden 无效, 详见https://stackoverflow.com/questions/41506456/why-body-overflow-not-working/41507857#41507857
    2. overflow 不为 hidden
    3. display的值为inline-block、table-cell、table-caption
    4. position的值为absolute或fixed
  - 规则 
    1. BFC的区域不会与float区域重叠, 两个元素是兄弟关系
    2. BFC元素如果存在某个子元素是浮动元素, 浮动元素仍参与父元素的高度计算
    3. 属于不同BFC的两个Box的margin不会发生合并
      - https://www.cnblogs.com/ianyanyzx/p/9126402.html
      - https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing
  - 作用 (请配合进行理解 https://codesandbox.io/s/vmx59kwy00)
    1. 利用规则1可以防止布局过程中侧边栏和主栏目发生重叠
    2. 利用规则2可以防止元素塌陷
    3. 利用规则3可以防止兄弟/父子元素之间外边距合并


  
* 请描述 IFC(Inline Formatting Context) 及其如何工作。
  - 形成: 只有在一个块级元素中仅包含内联级别元素时才会生成。
  - 规则:
    1. 内部的盒子会在水平方向，一个接一个地放置, 摆放这些盒子的时候，它们在水平方向上的 padding、border、margin 所占用的空间都会被考虑在内
    2. 能把在一行上的框都完全包含进去的一个矩形区域，被称为该行的行框（line box）。行框的宽度是由包含块（containing box）和存在的浮动来决定, 高度由里面最高盒子的高度决定 (无关竖直方向的margin/padding)
    3. 在垂直方向上，这些框可能会以不同形式来对齐(vertical-align), 例如底部或顶部对齐，或者内部的文本基线（baseline）对齐, 主要配合上文提到的line-box来做垂直方向上的对齐.
    4. IFC之间如果有块级元素会被隔开成两个独立的IFC, 就算是有多行的情况下, 行高也可能不同
  - 作用:
    1. 水平居中: 给一个块级元素加上display: inline-block，然后用父元素包裹住, 那么在父元素内部就会形成IFC, 此时给父元素添加 text-align: center 就能居中该块级元素.
    2. 垂直居中: 给一个块级元素加上display: inline-block，然后用父元素包裹住, 那么在父元素内部就会形成IFC, 此时给父元素添加一个伪元素, 使其高度为100%，inline-block 同时 vertical-align: middle, 此时只需要给该块级元素也加上vertical-align: middle 就能垂直居中.


* 有哪些行内替换元素, 他们有什么特点, 和行内元素有什么区别
  - 例如`img`, `input`根据标签的属性显示内容, 可以设置四个方向上的`padding, margin`以及`width, height`
  - 行内元素例如`span`, `a`竖直方向上的`padding-top、padding-bottom、margin-top、margin-bottom`无效果, 水平方向有效果
  - 实际上行内元素的`padding-top、padding-bottom`从视觉上确实是撑开了(例如添加背景), 但并不对周围元素的位置产生影响


* 对于 line-height 的理解
  - line-height 指一行字的高度，包含了字间距，实际上是下一行基线到上一行基线距离
  - 如果一个标签没有定义 height 属性，那么其最终表现的高度为 line-height * 文字所占的行数
  - 把 line-height 值设置为 height 一样大小的值可以实现单行文字的垂直居中
  - line-height 可设置的常见单位
    1. px: 绝对单位
    2. em/纯数字/百分比: 实际上 3em/3/300% 在line-height中作用相同, 都是相当元素本身的字体大小的倍数
  
  
* 水平居中的方式
  - 对于行内元素, 为其容器添加 `text-align: center`
  - 对于有宽度的块级元素, 为其自身添加 `margin: auto`
  - 容器添加`position: relative`自身添加 `position: absolute;`的情况下
    1. 自身添加 left: 50%; transform: translate(-50%, 0)`
    2. 自身添加 left: 0; right: 0; margin: auto 0;`
  - 其容器添加 `display: flex; justify-content: center;`
  
  
* 垂直居中的方式
  - 对于单行的行内元素, 为其容器添加与height相等的line-height即可.
  - 容器添加`position: relative`自身添加 `position: absolute;`的情况下
    1. 自身添加 top: 50%; transform: translate(0, -50%)`
    2. 自身添加 top: 0; bottom: 0; margin: auto;`
  - 容器添加
    ```
    .wrapper::before {
      content: '';
      height: 100%;
      display: inline-block;
      vertical-align: middle;
    }
    ```
    自身添加
    ```
    .center {
      display: inline-block;  
      vertical-align: middle;
    }
    ```
  - 其容器添加 `display: flex; align-items: center;`


* 什么是`rem`和`em`有什么区别
  - 首先浏览器默认字体大小为16px, 即HTML元素的字体大小为16px
  - em作为font-size的单位时，其代表父元素的字体大小(计算好的大小)，也就是说如果父亲是body, 并没有设置过em, 儿子写2em, 则儿子font-size为16 * 2 = 32px. 如果父亲写了2em, 儿子也是2em, 则儿子font-size为16 * 2 * 2 = 64px
  - em作为其他属性单位时，代表自身字体大小, 比如某个元素它的父亲是16px, 它自己的font-size: 2em, 然后width为2em. 则它的font-size是32px, 而宽是相对于这个32px的2em, 所以是64px
  - rem作用于非根元素时，相对于根元素字体大小, 这里说的根元素指的是html而不是body. rem作用于根元素字体大小时，相对于其出初始字体大小
  - rem可以用来做弹性布局, 通过设置根元素的字体大小来作为其他元素大小的单位.
  - 但更好的方案是 vw —— 视口宽度的 1/100；vh —— 视口高度的 1/10
  - https://yanhaijing.com/css/2017/09/29/principle-of-rem-layout/


* 请问 "resetting" 和 "normalizing" CSS 之间的区别？你会如何选择，为什么？
  - 目的都是统一浏览器对于常见元素的表现样式
  - CSS重置更激进, 对浏览器的默认样式进行了一些重置, 一般是通过`* 通配符选择器`来达到目的, 最终使得CSS样式有一个统一的基准
  - CSS一般化修复了浏览器的自身bug并保持浏览器的一致性, 宗旨是保护有用的浏览器默认样式而不是完全去掉它们
  - `reset`的缺点在于浏览器调试工具中大段大段的继承链, 样式调试变得复杂
  - `normalized`保护了有价值的默认值(这样不需要为所有的排版元素都添加样式), 同时修复了浏览器的bug (这往往超出了Reset所能做到的范畴).
  - https://www.jianshu.com/p/a7b9e2d20b73
  
  
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
  

* CSS 中字母 'C' 的意思是叠层 (Cascading)。请问在确定样式的过程中优先级是如何决定的 (请举例)？如何有效使用此系统？
  - 首先把所有样式分成三类, `!important`, 行内(即标签内部的), 内联(style标签中)和外联(另外的css文件中)
  - 在每一类中, 从0开始，id选择器+100，一个属性选择器、class或者伪类+10，一个元素选择器，或者伪元素+1，通配符+0
  - 优先取权重高的, 权重相同, 取后定义的, 这就是层叠的来源.


* 请解释浏览器是如何判断元素是否匹配某个 CSS 选择器？
    1. CSS选择器的解析是从右向左解析的。若从左向右的匹配，通常是到后面才发现不符合规则，需要进行回溯，会损失很多性能。
    2. 若从右向左匹配，先找到所有的最右节点，一旦发现匹配失败则可以直接返回, 避免了许多无效匹配。
    3. https://www.zhihu.com/question/20185756/answer/14263713
    4. https://stackoverflow.com/questions/5797014/why-do-browsers-match-css-selectors-from-right-to-left/5813672
  
    
    
* 请描述伪类以及伪元素的用途
  - 单冒号是伪类 例如`:hover, :visited, :checked`, 表示的是元素的状态或者结构特点, 例如鼠标在其上, 浏览过, 选择了, 第3个div等等.
  - 双冒号是伪元素 例如`::before, ::after`, 表示的是一种虚拟的元素，CSS把它当成普通HTML元素, 之所以叫伪元素，就因为它们在文档树或DOM中并不实际存在, 可用来制作图标, 消除浮动等.
  
  
* 请解释你对盒模型的理解，以及如何在 CSS 中告诉浏览器使用不同的盒模型来渲染你的布局。
  - 浏览器默认盒模型是 content-box 也就是说 盒宽 = 左右margin + 左右border + 左右padding + width (宽度即内容宽度) 同理高
  - 设置```* { box-sizing: border-box; }```之后 盒宽 = 左右margin + width (width = 左右border + 左右padding + 内容宽)
  

* 谈谈负的margin
  - 首先负的margin-top/margin-left, 他们会将元素向该元素的包含元素的上/左边界或者相邻元素的上/左边界拉, 也就是说会改变元素本身的位置
  - 其次如果是负的margin-bottom/margin-right, 他们并不移动本身, 但是会把下/右的元素的边界拉近, 如果是父元素则会把父元素的高/宽度拉小.
  
  
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
  - none
  ```
    不显示, 也不占空间
  ```
  

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


* 请问为何要使用 `translate()` 而非 *absolute positioning*，或反之的理由？为什么？
  - 使用 transform 时，可以让 GPU 参与运算，动画的 FPS 更高。
  - translate不会引起浏览器的重绘和重排
  - 使用 transform 参与时，可以做到更小（动画效果更加平滑), 而使用 position 时，最小的动画变化的单位是 1px.

  
* 描述`z-index`和叠加上下文是如何形成的。
  - 如果是不同的层叠上下文, 层叠顺序决定, 如果层叠顺序相同, 则后来居上
  - 如果是相同的层叠上下文, 层叠顺序决定, 如果层叠顺序相同, 则后来居上
  - https://www.zhangxinxu.com/wordpress/2016/01/understand-css-stacking-context-order-z-index/
