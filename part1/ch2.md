# 第二章：深入JavaScript

在上一章中，我介绍了编程的基本要素，如变量、循环、条件语句和函数等。当然，所有的代码也都是用JavaScript表示的。本章中，我们将主要关注一个JS开发者从入门到进阶需要了解的JavaScript的方方面面。

本章我我将介绍一些在以后的 *YDKJS* 书籍中才会详细讲解的概念。你可以把本章作为一个后续书籍深入讨论的主题的一个总览。

如果你是JavaScript的初学者，你应该多花些时间反复复习本章的概念和代码示例。任何伟大的建筑都是由一砖一瓦堆砌而成的，所以不要期望能够第一遍就能完全掌握所有的知识点。

现在让我们开始深入学习JavaScript之旅吧！

## 值&类型
我们在第一章中提到过，JavaScript的值是有类型的，而变量是没有类型的。JavaScript有下列的内置类型：
* `string`
* `number`
* `boolean`
* `null` 和 `undefined`
* `object`
* `symbol` (ES6中新引入)
JavaScript中可以用`typeof`操作符来查看某个值属于什么类型：
```js
var a;
typeof a;				// "undefined"

a = "hello world";
typeof a;				// "string"

a = 42;
typeof a;				// "number"

a = true;
typeof a;				// "boolean"

a = null;
typeof a;				// "object" -- weird, bug

a = undefined;
typeof a;				// "undefined"

a = { b: "c" };
typeof a;				// "object"
```
`typeof`操作符的返回值永远是六种类型之一（ES6中是7种类型，包括'symbol'类型）的字符串值。例如，`typeof "abc"`返回`"string"`，而不是`string`。

注意上面的代码中变量a可以保存不同类型的值，`typeof a`不是获取变量a的类型，而是获取变量a中当前值的类型。JavaScript中只有值才有类型，而变量仅仅是存放这些值的容器。

`typeof null`是一个有意思的例子，因为它会错误地返回"object"，而不是预料中的"null"。
**注意：** 这是JS的一个遗留bug，但是可能永远也不会修复。由于Web中有太多的代码依赖于这个bug，因此修正这个bug可能会导致更多的bug！

还要注意`a = undefined`。这里显示地给变量a赋值为undefined值，这在表现上与没有给变量a设置任何值的情况是一样的，如上面代码第一行的var a。有几种情况会使得变量的值为"undefined"，包括没有返回值和使用了void操作符的函数。

### 对象
`object`类型表示一类可以设置属性（命名的位置符）的复合值，每个属性保存它们自己的任意类型的值。对象可能是JavaScirpt中最有用的值类型了。
```js
var obj = {
	a: "hello world",
	b: 42,
	c: true
};

obj.a;		// "hello world"
obj.b;		// 42
obj.c;		// true

obj["a"];	// "hello world"
obj["b"];	// 42
obj["c"];	// true
```
将这个obj对象可视化为如下形式有助于理解：

|a             |b   |c     |
|:------------:|:--:|:----:|
|"hello world" | 42 | true |
可以用 *点记法*（即`obj.a`）或 *括号记法*（即`obj['a']`）来访问对象的属性。点记法更简单且易读，因此尽可能的使用点记法。
如果属性名中包含特殊字符，则应该使用括号记法，如`obj["hello world!"]`——通过括号记法访问的属性通常作为 *键值*。`[ ]`中要么是一个变量（稍后解释），要么是一个 *字面量字符串*（包裹在`".."`或`'..'`中）。
当然如果你想访问的键值对的名字保存在另一个变量中，也应该使用括号记法，如：
```js
var obj = {
	a: "hello world",
	b: 42
};

var b = "a";

obj[b];			// "hello world"
obj["b"];		// 42
```
**注：** 查看更多有关JavaScirpt对象的知识点，参考 *this&对象原型* 一书，特别是第三章。

JavaScirpt中还有两种常见的值类型：*数组* 和 *函数*。它们不是内置的值类型，而更像是子类型——特殊的`object`类型。

#### 数组
数组是按数字索引顺序保存属性值的对象，如：
```js
var arr = [
	"hello world",
	42,
	true
];

arr[0];			// "hello world"
arr[1];			// 42
arr[2];			// true
arr.length;		// 3

typeof arr;		// "object"
```
**注：** 编程语言从0开始计数，JS也不例外，将`0`作为数组第一个元素的索引。
`arr`的可视化形式如下：

|0             |1   |2     |
|:------------:|:--:|:----:|
|"hello world" | 42 | true |
由于数组是特殊的对象（如`typeof`的结果所示），因此也可以有属性，包括自动更新的`length`属性。

理论上通过自定义属性名，可以把数组当作正常的对象使用，也可以用`object`来模拟数组，只需将其属性用数字（0，1，2等）即可。当然这样的用法不利于区别不同的值类型。

最好且最自然的方式是用数组表示以数字做索引的值，而用`object`表示具有命名属性的值。

#### 函数
JS程序中常见的另一个`object`子类型是函数：
```js
function foo() {
	return 42;
}

foo.bar = "hello world";

typeof foo;			// "function"
typeof foo();		// "number"
typeof foo.bar;		// "string"
```
再次强调，函数是`object`的子类型——`typeof`返回`"function"`，表明function是一个主要类型——因此可以有属性，但是一般只有在少数情况下才会用函数的对象属性（如`foo.bar`）。

**注：** 更多关于JS值及其类型的知识，参考 *类型&语法* 一书的第二章。

### 内置类型方法











