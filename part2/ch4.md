# 第四章：变量提升

到目前为止，读者应该已经熟悉了作用域的概念，以及变量是怎样根据声明方式和声明位置的不同而归属于不同的作用域层级。函数作用域和块级作用域都遵从同一个规则：某个作用域内声明的任何变量都归属于这个作用域。

本章我们详细讨论下作用域内不同位置声明的变量是如何添加到这个作用域内的。

### 先有鸡还是先有蛋？

我们通常很容易认为在Javascript程序执行时，你看到的所有代码都是按顺序从上到下逐行被解释的。虽然程序确实是这样执行的，但是有一点需要特别注意！
考虑：
```js
a = 2;

var a;

console.log( a );
```
会输出什么呢？很多开发者认为会输出`undefined`，理由是`var a`语句在`a=2`之后，自然会对变量a重新赋值，因此a的值会是默认值undefined。然而，实际输出是2。

考虑另一段代码：
```js
console.log(a);
var a = 2;
```
根据上一段代码中的行为，你可能认为输出结果会是2，或者由于使用变量a之前未定义该变量而抛出引用错误。

很遗憾的告诉你，这两种猜测都不对。输出结果是`undefined`。这是怎么回事？到底是先有鸡还是先有蛋呢？

### 再次请出编译器

回忆第一章中我们对编译器的讨论：实际上引擎会在解释之前先编译JS代码，编译阶段的一个主要作用就是找到并将所有的声明语句与它们各自的作用域联系起来。第二章解释了这正式词法作用域的核心。

因此，这个问题的正确答案是：所有的声明，包括变量和函数都会在任何代码被执行之前先被编译器处理。
