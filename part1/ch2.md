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














