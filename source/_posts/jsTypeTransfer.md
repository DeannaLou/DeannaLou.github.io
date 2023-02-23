---
 title: js 中的隐式转换
 date: 2018-06-24
---

## js 的值类型
- number
- string
- boolean
- null
- undefined
- object

js 把 number、string、boolean、null、undefined 归成原始值。
array 与 function 都是 object 的特例。

## js 是弱类型语言
所谓的弱类型，就是说你不必在声明变量时就指定它的类型，并且你可以给一个变量赋任意类型的值。

```
var a = 1;  // 声明一个变量，并赋一个 number 类型的值
a = "34"； // 赋其他类型的值
a;  // >> "34"
```

因为它的弱类型特性，在期望某种类型的值而当前值类型不符合时，js 会根据固定的规则来进行隐式的转换。我们最常见的隐式转换之一就是 + 操作符了。

```
// 把 number 12 转换成了 string "12"，并做了字符串拼接的处理
a + 12;   //  >> "3412"
// 把 String "34" 转换成了 number 34，并做了数学减运算
a - 12;  //  >>  22
```

<h2><a id="type_convert">隐式转换规则表</a></h2>

值|转换为string|转换为number| 转换为boolean|转换为object
---|----------------|-------------------|---------------------|----------------
undefined | "undefined" | NaN | false | throws TypeError
null | "null" | **0** | false | throws TypeError
true | "true" | **1** | - | new Boolean(true)
false | "false" | **0** | - | new Boolean(false)
""(空串) | - | **0** | **false** | new String("")
"0"(非空，数字) | - | 0 | true | new String("0")
"one"(非空，非数字) | - | NaN | true | new String("one")
0 | "0" | - | **false** | new Number(0)
-0 | "0" | - | **false** | new Number(-0)
NaN | "NaN" | - | **false** | new Number(NaN)
Infinity | "Infinity" | - | true | new Number(Infinity)
-Infinity | "-Infinity" | - | true | new Number(-Infinity)
1(非0) | "1" | - | true | new Number(1)
{} | [对象的类型转换](#object_convert) | - | true | -
\[\](任意数组) | "" | **0** | true | -
\[9\](1个数字元素) | "9" | **9** | true | -
\['a'\](其他数组) | 使用 join() 方法 | NaN | true | -
function(任意函数) | [对象的类型转换](#object_convert) | NaN | true | -

来看一道题目:

```
var s = "";
s.x = "1";
console.log(s.x); // undefined
```

首先，看到 s 是一个 string 型的，那么 s.x = “1” 为什么不会报错？


> 是因为我们的 "." 运算符期望左边的值是一个对象，所以隐式转换成了一个对象，也就是上表中的 new String("")。


其次，既然我在这个隐式转换成的对象上添加了属性 x，为什么访问不到呢？

> 是因为隐式转换时使用的是 new 关键字，也就是说我们每次进行隐式转换所返回的对象都是一个新的对象。所以 s.x 输出的是 undefined。

<h2><a id="object_convert">对象的类型转换</a></h2>

虽然 null 是可以理解成一个特殊的对象（typeof null 返回 "object"），但实际上它是 null 类型的唯一一个成员，所以它的类型转换与我们本节要讨论的无关，null 的转换请查看[隐式转换规则表](#type_convert)。

> Object 上有两个方法，一个是 valueOf, 一个是 toString，把 object 转成原始值依赖于这两个方法的返回值。

内置的 valueOf 方法一般返回这个对象本身。
- Date 对象特殊实现了 valueOf 方法，它的 valueOf 返回时间戳

内置的 toString 方法一般返回 "[object Object]"。
- Date 特殊实现的 toString 方法返回一个可读的日期和时间字符串；
- Array 特殊实现的 toString 方法返回执行  join() 的结果，这也是为什么 \[9\] 转换成数字 9, 因为 [9].join() 返回字符串 "9"，而字符串 "9" 又转换成数字 9（可能有人问怎么不是执行 valueOf，那是因为 valueOf 返回值是数组自身，并不是一个原始值）。
- Function 特殊实现的 toString 方法返回定义方法的表示，是一个字符串。

![function.toString](http://upload-images.jianshu.io/upload_images/5370440-6ef0cea0075e9efc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![Object.valueOf 与 Object.toString](http://upload-images.jianshu.io/upload_images/5370440-377983feb6f1b508.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对象的类型转换规则共分为4种情况。
- object to boolean： 所有的对象都是真值，即转换成 true。这里注意 new Boolean(false) 返回的是一个对象，所以转换成 true。
- object to number: 优先调用 valueOf，如果未返回一个原始值，则调用 toString，如果都未返回原始值，则 throw TypeError。
- object to string：优先调用 toString，如果未返回一个原始值，则调用 valueOf，如果都未返回原始值，则 throw TypeError。
- object to primitive value

看起来好像还挺简单的样子，其实就是这么简单，只不过有一个特例需要记住。
\+ 运算符可以用来做数学加运算，也可以用来做字符串拼接, 与此同时还有 == 与 != 运算符，如果使用这三个运算符时，操作数存在 object 类型的话，那么 object 并不是使用的 object-to-number 或 object-to-string 规则去获得原始值，而是使用 object-to-primitive 规则。

### object-to-primitive 规则
- 对于 + 、==、!= 三个运算符来说，一般的 object 会走 object-to-number，也就是先调用 valueOf。而特殊的一点时，Date 走 object-to-string 规则，也就是先调用 toString。
- 对于其他的操作符，比如关系操作符，<、>、<=、=== 等，所有对象包括Date都会走 object-to-number 规则。
 
![Date 转换规则](http://upload-images.jianshu.io/upload_images/5370440-54b4ff7816f75552.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 除了明确需要 Boolean 或者 String 型的地方，Object 转成原始值都是优先调用 valueOf，如果执行结果是原始值，则返回该结果，否则再调用 toString，如果执行结果是原始值，则返回该结果，否则报错。需要注意的是特例 Date 型，它在做 +、==、!= 运算时，优先调用 toString。

明确需要 Boolean 的地方，很常见的例子：

```
var o = {};
if (o) {  // 所有的对象都是真值
// doSth...
}
```

## function 使用 valueOf、toString
考虑下边这道题：
> 实现 add 方法，使其可如此使用: add(1)(2,4,5,7)(9)

可能我们常常遇见的场景仅仅是做了二次调用，也就是 add(...args) 徐亚返回一个 function，对这个 function 可以进行二次调用。
可是这里的调用层级是3次甚至更多。

看下解决方案：

```
function add() {
  var args = Array.prototype.slice.call(arguments);
  var _add = function() {
    Array.prototype.push.apply(args, arguments);
    return _add;
  }
  _add.valueOf = () => args.reduce((a,b) => a+b);
  return _add;
}
var s = add(1)(4,6)(5);   // function 16 (此处 function 是标识通过 valueOf 或者 toString 返回的，使用时可以当做原始值来使用)
typeof s;  // "function";
s + ""; // "16"
+s; // 16
```