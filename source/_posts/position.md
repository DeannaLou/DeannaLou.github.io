---
title: position 定位
---
在学习 position 之前，先来看一下几个适用于所有 css 属性的值： inherit、initial、unset、revert、all

# initial、inherit、unset、revert 与 all 关键字

## initial 指定设置为默认值
默认值列表存在 css 的属性汇总里(这个表网上没有找到资料，猜想和设备与浏览器也有关系，例如 font-family 根据不同浏览器和设备而定，position 默认 "static"，color 默认 "#000" 等)。

PS：initial 在 IE11 下不兼容

> postion: initial;
> color: initial;
> width: inital;
> // other properties...

## inherit 指定继承父节点的该属性

> 所有 css 属性的 inheritance 都被设置为 “yes” 或者 “no”，“yes” 表示继承， “no” 表示不继承。

inherit 允许用户去手动把 inheritance 置为 “yes”。

1）根元素会被默认设置 initial 的值，如果属性是inheritance = “yes”的话，会去找父节点的属性值，例如 color 等。

对此类属性设置 inherit 的话，它的父节点如果未设值，那么父节点会继承祖父节点的值，如果都未设置，最终找到根节点的 initial。

2）inheritance = “no” 的属性会被默认设置为 initial 的值，例如 width、border、position 等。

对此类属性设置 inherit 的话，它的父节点如果未设值，那么父节点会默认 initial 的值，也就是说它要么继承了父节点设置的值要么 initial。

> postion: inherit;
> color: inherit;
> width: inherit;
> // other properties...

## unset ：initial 与 inherit 的结合

如果该属性的inheritance = “yes” 的话，就使用 inherit 继承，inheritance = “no” 就使用 initial 默认值。

注意：unset 与直接不设是不同的，如果不设，那么它会有一个优先级的判断（在 1.4 中会提到这个优先级），例如 div 上设置了 color1，父元素上也设置了 color2，这个 div 会使用 color1，而不是直接继承父元素的 color2。

> postion: unset;
> color: unset;
> width: unset;
> // other properties...

## revert ：指定恢复 user-agent 的设置

UA（user-agent）是浏览器给定的用户代理样式表，优先级比较低，继承(只作用与inheritance = “yes” 的属性) < UA < 用户样式表 < 内联样式，如图：

![内联样式](https://upload-images.jianshu.io/upload_images/5370440-7262d45fdea6cc21.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

revert 与 initial 是不同的，display 的 initial 默认值是 inline，而 UA 把 div 标签 display 定义为 block，revert 指定恢复到 UA 的设置。

如果设置为 revert 的属性在 UA 中未被设值的话，属性的行为与 unset 一致。

注意：据官网数据，目前 Chrome 与 FF 都不支持，safari 9.1+ 支持，其他浏览器待验证。

> display: revert;
> // other properties...

## all 属性

all 用来重设除了unicode-bidi和direction之外的所有属性至它们的 initial 或 inherit 值。可选值为 initial，inherit，unset，revert。

> all: initial;
> all: inherit;
> all: unset;
> all: revert;
> // other properties...

all 设置为 inherit 后，给这个元素设置的其他优先级更高（加 !important、内联样式或在all:inherit之后设置）的属性都进行覆盖。例如下图，font-size 会继承，而 height、width 等都会生效。

![inherit](https://upload-images.jianshu.io/upload_images/5370440-23009150206d9218.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

all 设置为 initial 后，官网给的说法是设置为初始值，优先级更高的属性理应和 inherit 一样被覆盖掉，但实际的效果却有一点奇怪，给这个元素设置的其他优先级更高的大多属性都可进行覆盖，margin-top，margin-bottom 在 safari、chrome 下不占位、ff 表现正常，width、height 不能覆盖。但 margin-top、margin-bottom、width、height在绝对定位（absolute 或 fixed）下可正常显示或覆盖。

all 设置为 unset 后，与 initial 一样，margin-top、margin-bottom、width、height 表现奇怪，但在绝对定位下可正常显示或覆盖。另外 safari 中，color 属性在绝对定位下也不可以正常的覆盖。

all 的 revert 属性，会把用户设置的属性重设为 UA 表中设置的属性，但在safari 9+的浏览器上，color 属性不可以正常的覆盖，在绝对定位下也不可以。

## postion

> postion 属性支持以下值：
> position: static;  // position 的默认值
> position: relative;
> position: absolute;
> position: fixed;
> position: sticky; // 这个属性在实验阶段，兼容性比较差: IE/opera 不支持，FF 32+，chrome 56+，Safari 6.1+）

一个定位元素是指 postion 被计算为relative,absolute,fixed或sticky的一个元素。（注意：元素如果设置了__非 none 的 transform 属性__，那么它就变成了定位元素）

相对定位元素：被计算为 relative 的元素，

绝对定位元素（这是官方的说法，不同于我之前以为的区分绝对定位和固定定位）：被计算为 absolute 或 fixed 的元素，

粘性定位元素：被计算为 sticky 的元素。

偏移 top、bottom、left、right 只对定位元素有效。

### static

static 是默认定位，即 initial 所指定的 position 的值，此时偏移值 和z-index 都无效。

### relative

relative 是相对定位，relative 会根据它的大小给它留出固定的空间，再去做它的偏移，也就是说 relative 的偏移是在它固定留白的基础上做的，并且偏移后并不会改变它的留白和大小。

position：relative未定义在table-*-group, table-row, table-column, table-cell, table-caption 元素上的效果。

### absolute

absolute 是绝对定位，不会为元素预留空间，它相对于最近的祖先定位元素来定位，可以设置margin，并且不会被合并（相邻元素的margin-top 和 margin-bottom总是会被合并）。

### fixed

fixed 是绝对定位，与 absolute 很像，只是它相对于屏幕视口 viewport 来定位，屏幕滚动的时候元素位置不变。

### sticky：relative 与 fixed 的结合

sticky 必须设置了偏移量（相对于屏幕视口） top/left/bottom/right 才生效，否则就和 relative 同效。在滚动到该偏移量时，采用 fixed 的定位，但当往下滚动到该元素的底部与父节点的内容（不包括 padding、margin、padding）的底部相重叠、或者往上滚动到该元素在 relative 布局下相对于父元素所在位置时，快照它当前的偏移位置，并采用 relative 定位。

sticky可以用来做首字母搜索，A标题的区域内容显示时，A标题一直固定在顶部，屏幕滚动到 B区域内容时，B标题代替A一直固定在顶部。

![sticky](https://upload-images.jianshu.io/upload_images/5370440-4bd466d758a8a05c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)