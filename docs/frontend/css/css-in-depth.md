## 样式表来源

1. 作者样式表
2. 用户代理样式表/user agent stylesheet：即浏览器默认样式
   其中的 **作者样式表** 会覆盖 **用户代理样式表**

层叠的高级流程图，展示了声明的优先顺序：<img src="../css-in-depth/flowchart.png" />

## 全局选择器
```css
:root {

}

*,
::before,
::after {

}
```

## 单位
`em` 当前字体大小的相对单位。 `rem=Root em`，根CSS文件中字体大小的相对单位

!!! note "tipps"
    我一般会用rem设置字号，用px设置边框，用em设置其他大部分属性，尤其是内边距、外边距和圆角（不过我有时用百分比设置容器宽度）。 

- vh：视口高度的1/100。
- vw：视口宽度的1/100。
- vmin：视口宽、高中较小的一方的1/100（IE9中叫vm，而不是vmin）。
- vmax：视口宽、高中较大的一方的1/100（本书写作时IE和Edge均不支持vmax）￼。

可在Can I Use网站中检索Viewport units: vw, vh, vmin, vmax中的“Known Issues”。

### 无单位的数字属性值
有些属性允许无单位的值（即一个不指定单位的数）。支持这种值的属性包括line-height、z-index、font-weight（700等于bold,400等于normal，等等）。无单位的数值，因为它们继承的方式不一样。使用无单位的数值时，继承的是声明值

段落字号是32px（2em × 16px，浏览器默认字号），所以此时行高的计算值为38.4px（32px×1.2）


## 选择器
- `>` 是一个直接后代组合器。 
- `*` 所有后代recursively


## 函数
calc()函数内可以对两个及其以上的值进行基本运算。当要结合不同单位的值时，calc()特别实用。它支持的运算包括：加（+）、减（−）、乘（×）、除（÷）。

:warning: 加号和减号两边必须有空白 


## 变量
变量名前面必须有两个连字符（--），用来跟CSS属性区分，剩下的部分可以随意命名。 
⚠️ 在不支持自定义属性的浏览器上，任何使用ver（）的声明都会被忽略。请尽量为这些浏览器提供回退方案(fallback)：
```css
color: black;
color: var (--main-color);
```

应用场景有切换网站主题等。

我们可以用JS访问到该变量的值：
```html
<script>
   var rootElement = document.documentElement;
   var styles = getComputedStyle(rootElement);
   var mainColor = styles.getPropertyValue('--main-bg');
   console.log(String(mainColor).trim());
</script>
```

## 布局
高级的布局话题基于`文档流 / flow`和`盒模型 / box-model 等概念，这些是决定网页元素的大小和位置的基本规则。
### 横向：两列布局
使用浮动布局，将main和sidebar向左浮动，分别设置70%和30%的宽度。
```css
.main {
   float: left; 
   width: 70%;
}
.sidebar {
   float: left;
   width: 30%;
}
```
虽然将两列宽度设置为70%和30%，但它们总共占据的宽度超过了可用空间的100%。这是因为其中的width指的是纯内容的宽度，即 `box-sizing: content-box`（没有算上padding，border和margin）

<img src="../css-in-depth/box-model-original.png" width="300" />

解决方法：

1. 将`sidecar`的宽度设置成 `calc(30%-3em)`
2. 调整 盒模型 `box-sizing: border-box`

   <img src="../css-in-depth/box-model-border.png" width="300" />

Best Practise:
```css
:root {
   box-sizing: border-box;
}

*,
::before,
::after {
   box-sizing: inherit;
}
```

!!! note "普通文档流 / Workflow"
   普通文档流——指的是网页元素的默认布局行为。行内元素跟随文字的方向从左到右排列，当到达容器边缘时会换行。块级元素会占据完整的一行，前后都有换行。

   普通文档流是为限定的宽度和无限的高度设计的。

###  高度
普通文档流是为限定的宽度和无限的高度设计的。当明确设置一个元素的高度时，内容可能会溢出容器。用overflow属性可以控制溢出内容的行为，该属性支持以下4个值：

- visible（默认值）: 所有内容可见，即使溢出容器边缘。
- hidden: 溢出容器内边距边缘的内容被裁剪，无法看见。
- scroll: 容器出现滚动条，用户可以通过滚动查看剩余内容。在一些操作系统上，会出现水平和垂直两种滚动条，即使所有内容都可见（不溢出）。不过，在这种情况下，滚动条不可滚动（置灰）。
- auto: 只有内容溢出时容器才会出现滚动条。

:warning: 谨慎地使用滚动条!

### 垂直居中
### 等高列


!!! info IE
      IE有一个bug，它会默认将`<main>`元素渲染成行内元素，而不是块级元素，所以代码中我们用声明`display: block`来纠正。



