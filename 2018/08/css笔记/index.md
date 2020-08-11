# CSS笔记


每元素都有一个**外在盒子**和**容器盒子**(_内在盒子_)

-   外在盒子负责元素是否一行显示
-   容器盒子负责元素的宽高、内容呈现

# 块级元素

-   外在盒子是 `block` 的元素
-   都有*主块级盒子*
-   list-item 还有一个*附加盒子*(IE 伪元素不支持附加盒子)

## width : auto

-   充分利用可用空间，块级元素默认 100%
-   收缩与包裹。代表是浮动、绝对定位、inline-block、table 元素
-   收缩到最小
-   超出容器限制。内容很长的连续英文和数字，或者设置了 `white-space:nowrap`

    _（除了第一个是外部尺寸，其余都是内部尺寸。分别对应以下的外部尺寸和内部尺寸）_

#### 外部尺寸

-   正常 width 是 100%
-   格式化宽度
    > 出现在 position 为 absolute 和 fixed 元素中。  
    > 默认绝对定位元素宽度是由内部尺寸决定，除了当 left 和 right 或 top 和 bottom 同时出现时，其宽度相对于最近的具有定位特性的祖先元素计算

#### 内部尺寸

-   包裹性

    顾名思义，尺寸由内部元素决定，单永远小于包含块容器尺寸

-   首选最小宽度

    当外部容器盒子宽度为 0，内部的内联盒子就是*首选最小宽度*

    -   中文，为单个汉子宽度
    -   西文遇到空格、-、?、其他非英文字符，就会换行
    -   类似图片这样的替换元素，就是元素本身的宽度

-   最大宽度  
    等同于 _包裹性_ 元素设置 `white-sapce:nowrap` 后的宽度  
    是最大的连续内联盒子的宽度

#### height

如何让元素支持 `height:100%` 效果？

1.  设置父元素高度
2.  使用绝对定位

> 绝对定位的宽高百分比是相对于 padding box，其他是相对于 content box

#### min-width/height 和 max-width/height

> -   `min-width/min-height` 初始值是 auto，`max-*` 是 none
> -   `max-width` 会覆盖`width`，即使是`width:xpx !important`
> -   `min-width`会覆盖`max-width`

# 内联元素

外在盒子是内联盒子的元素

### 内联盒子模型

-   内容区域
    > 围绕文字的盒子
-   内联盒子
    > 是元素外在盒子
-   行框盒子
    > 每一行就是一个行框盒子
-   包含盒子（包含块）
    > 由一行一行的**行框盒子**组成，是父元素外面的盒子

### 幽灵空白节点（strut）

内联元素的解析就像每个行框盒子前面都有一个*空白节点*，宽度为 0

-   ### 替换元素

    > 内容可替换的元素，如：  
    >  `<img>`,`<object>`,`<video>`,`<iframe>`,`<input>`,`<textarea>`,`<select>`

    有如下特性：

    1. 外观不受 css 影响
    2. 有自己的尺寸
    3. 很多 css 属性有自己的一套表现规则。如 `vertical-align` ，替换元素的`baseline`是元素的下边缘，非替换元素是**x**的下边缘

    替换元素默认 dispay 值

    > 都是`inline`或`inline-block`  
    > 主要记:  
    > `<img>`是`inline`  
    > `<input>`是`inline-block`_（火狐是 inline）_

    #### `<input>`和`<button>`区别：

    > 前者`white-space`是 pre，后者是 normal

    white-space:

    -   normal _(默认)_  
        连续的空白符会被合并，换行符会被当作空白符来处理。填充 line 盒子时，必要的话会换行
    -   nowrap  
        和 normal 一样，连续的空白符会被合并。但文本内的换行无效
    -   pre  
        连续的空白符会被保留。在遇到换行符或者`<br>`元素时才会换行。
    -   pre-wrap  
        连续的空白符会被保留。在遇到换行符或者`<br>`元素，或者需要为了填充 line 盒子时才会换行。
    -   pre-line  
         连续的空白符会被合并。在遇到换行符或者`<br>`元素，或者需要为了填充 line 盒子时会换行。

        |          | 换行符 | 空格和 tap | 文字转行 |
        | -------- | :----: | :--------: | :------: |
        | normal   | 空白符 |    合并    |   yes    |
        | nowrap   | 空白符 |    合并    |    no    |
        | pre      |  换行  |    保留    |    no    |
        | pre-wrap |  换行  |    保留    |   yes    |
        | pre-line |  换行  |    合并    |   yes    |

    ### 替换元素尺寸

    -   固有尺寸

        > 原本尺寸，无法改变

    -   HTML 尺寸

        > 通过 HTML 属性改变的尺寸，如：  
        > **img**的*width*,_height_  
        > **input**的*size*属性  
        > **textarea**的*cols*和*rows*

    -   CSS 尺寸
        > 通过 CSS 改变的尺寸

    从下都上，优先级递减

---

-   Firefox 中没有*src*属性的**img**元素是*inline*元素

-   css 之所以可以改变图片的大小，是因为图片中的 comtent 默认的`object-fit`是`fill`

### 替换元素离非替换元素有多远？

-   只隔了一个*src*  
    Firefox 直接就行，Chrome 要有一个不为空的 _alt_ 值
-   只隔了一个*content*属性  
    `counter:url('...')`

## content 计数器

-   counter-reset

    > 给计数器起名，和从那个数字开始计数  
    > `.xxx{ counter-reset:name 2}`  
    > 名字就是 name ，从 2 开始，默认 0，数字不合符当 0 处理  
    > 可以多个计数器同时命名  
    > `.xxx{ counter-reset:name 2 name2 3}`

-   counter-increment

    > key 为 counter-reset 的名字，值是每次增加的值，没有则默认 1，也可以有多个 key 用空格如同 counter-reset
    >
    > ```css
    > .counter {
    >     counter-reset: szx 2;
    >     counter-increment: szx 1;
    > }
    > .counter::before {
    >     content: counter(szx);
    > }
    > ```
    >
    > counter-increment 在父元素或子元素都有效

-   方法*counter( )* / _counters( )_

    > counter(name[,style])
    >
    > -   style 支持的值就是*list-style-type*支持的值，作用是增减可以是英文字母或罗马文
    > -   一个 content 可以有多个 content( )方法

    > counters(name,string[,style])
    >
    > -   string 必须，表示子序号的链接字符,style 同上
    > -   reset 不要和 counter 同级

