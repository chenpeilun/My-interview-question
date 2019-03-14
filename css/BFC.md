块级格式化上下文BFC

> 来源：https://www.cnblogs.com/libin-1/p/7098468.html

#### 什么是 BFC?

**BFC(Block Fromatting Context) 块级格式化上下文**

W3C对BFC的定义如下

> 浮动元素和绝对定位元素，非块级盒子的块级容器（例如 inline-blocks, table-cells, 和 table-captions），以及overflow值不为“visiable”的块级盒子，都会为他们的内容创建新的BFC（块级格式上下文）。

一个HTML元素要创建BFC，则满足下列的任意一个或多个条件即可:

- float的值不是none。

- position的值不是static或者relative

- display的值是inline-block、table-cell、flex、table-capton或者inline-flex

- overflow的值不是visible

  

BFC是一个独立的布局环境，其中的元素布局是不受外界的影响，并且在一个BFC中，块盒与行盒（行盒由一行中所有的内联元素所组成）都会垂直的沿着其父元素的边框排列。

#### 创建BFC

创建BFC只需要满足上述的4个css条件之一即可。

在类container中添加类似 overflow: scroll，overflow: hidden，display: flex，float: left，或 display: table 的规则来显示创建BFC。虽然添加上述的任意一条都能创建BFC，但会有一些副作用：

1. display: table 可能引发响应性问题
2. overflow: scroll 可能产生多余的滚动条
3. float: left 将把元素移至左侧，并被其他元素环绕
4. overflow: hidden 将裁切溢出元素

#### 外边距塌陷

创建新的BFC避免两个相邻 `<div>` 之间的 **外边距合并** 问题

```html
<div class="container">
    <p>Sibling 1</p>
    <p>Sibling 2</p>
    <p>Sibling 3</p>
</div>
```

css

```css
.container {
  background-color: red;
  overflow: hidden; /* creates a block formatting context */
}
p {
  background-color: lightgreen;
  margin: 10px 0;
}
```

结果和上面一样，由于外边距折叠，三个相邻P元素之间的垂直距离是10px，这是因为三个 p 标签都从属于同一个BFC。

修改第三个P元素，使之创建一个新的BFC：

```html
<div class="container">
    <p>Sibling 1</p>
    <p>Sibling 2</p>
    <div class="newBFC">
        <p>Sibling 3</p>
    </div>
</div>
```

css

```
.container {
    background-color: red;
    overflow: hidden; /* creates a block formatting context */
}
p {
    margin: 10px 0;
    background-color: lightgreen;
}
.newBFC {
    overflow: hidden;  /* creates new block formatting context */
}
```

![img](https://camo.githubusercontent.com/c7c77701b984c7d4f957a7729db2cf534ef92c3d/687474703a2f2f7365676d656e746661756c742e636f6d2f696d672f62566d327153)

因为第二个和第三个P元素现在分属于不同的BFC，它们之间就不会发生外边距折叠了。



#### BFC避免文字环绕

#### ![img](https://camo.githubusercontent.com/f09eac6e73acf8ecfcc329b1bff19c778912219e/687474703a2f2f7365676d656e746661756c742e636f6d2f696d672f62566d327159)

如上图所示，对于浮动元素，可能会造成文字环绕的情况(Figure1)，但这并不是我们想要的布局(Figure2才是想要的)。要解决这个问题，我们可以用外边距，但也可以用BFC。

#### 使用display：flow-root

从上述的几种创建BFC的方法都可以看到，都会产生副作用，所以css工作组增加了一个新的display属性：flow-root，它会创建一个新的有用的BFC，它包含浮动，阻止外边距塌陷，并且阻止元素环绕浮动。

但是现在flow-root属性不是兼容所有浏览器

![img](https://mmbiz.qpic.cn/mmbiz_png/eXCSRjyNYcbdSyR5Gc52ZEb2gm7pdliaa1oSibRlQuiaz3sxY5fXmPxibFvHeFJQjP6LE2qPCCwKHfQvFQPu0HVlCw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)