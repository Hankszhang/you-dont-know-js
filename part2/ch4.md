# 第四章：变量提升

到目前为止，读者应该已经熟悉了作用域的概念，以及变量是怎样根据声明方式和声明位置的不同而归属于不同的作用域层级。函数作用域和块级作用域都遵从同一个规则：某个作用域内声明的任何变量都归属于这个作用域。

本章我们详细讨论下作用域内不同位置声明的变量是如何添加到这个作用域内的。

## 先有鸡还是先有蛋？

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

## 再次请出编译器

回忆第一章中我们对编译器的讨论：实际上引擎会在解释之前先编译JS代码，编译阶段的一个主要作用就是找到并将所有的声明语句与它们各自的作用域联系起来。第二章解释了这正式词法作用域的核心。

因此，这个问题的正确答案是：所有的声明，包括变量和函数都会在任何代码被执行之前先被编译器处理。

对于声明语句`var a = 2;`，Javascript时间上把它当作两条语句来对待：`var a;`和`a = 2;`。第一个语句是声明，在编译阶段被处理，第二条语句是赋值则是在执行阶段被处理。

因此，上文中的第一段代码可以被认为是：
```js
// the compilation
var a;
// the execution
a = 2;
console.log( a );
```
同样的对于第二段代码，实际上被处理为：
```js
var a;
console.log( a );
a = 2;
```
综上，可以这样来理解这个处理过程：变量和函数声明从它们在代码流中的位置被“移”到了代码的顶部。这个行为被叫做“变量提升”。

也就是说，先有蛋（声明），再有鸡（赋值）。

**注意**：只有声明本身会被提升，而其他任何赋值或其他可执行逻辑都保持不变。

另外一个需要注意的是每个作用域的变量都会被提升。
```js
foo();

function foo() {
    console.log( a ); // undefined

    var a = 2;
}
```
上面代码中除了foo函数在全局作用域内会被提升之外，foo函数本身也会将`var a`提升到`foo(){ .. }`的顶部，而不是整段代码的顶部。实际效果如下：
```js
function foo() {
    var a;

    console.log( a ); // undefined

    a = 2;
}

foo();
```
需要注意的是，函数声明会被提升，但是函数表达式不会被提升。
```js
foo(); // not ReferenceError, but TypeError!

var foo = function bar() {
    // ...
};
```
变量标识符foo被提升，因此foo()被执行时不会导致`ReferenceError`。但是foo还没有值。因此foo()尝试调用undefined值，从而导致非法的操作，抛出`TypeError`。

还需要注意的是，尽管这是一个命名函数表达式，函数名标识符在作用域内也是不可用的：
```js
foo(); // TypeError
bar(); // ReferenceError

var foo = function bar() {
    // ...
};
```
这段代码实际上被解释为：
```js
var foo;

foo(); // TypeError
bar(); // ReferenceError

foo = function() {
    var bar = ...self...
    // ...
}
```
## 函数优先提升

函数声明和变量声明都会被提升，但是函数声明会先被提升，其次才是变量。
考虑：
```js
foo(); // 1

var foo;

function foo() {
    console.log( 1 );
}

foo = function() {
    console.log( 2 );
};
```
输出了1而不是2！引擎将上面的代码解释为：
```js
function foo() {
    console.log( 1 );
}

foo(); // 1

foo = function() {
    console.log( 2 );
};
```
需要注意的是`var foo`是重复声明，因此会被忽略，尽管它在function foo()...声明之前。

尽管多重的var声明会被有效的忽略，但是后面的函数声明会覆盖之前声明的函数。
```js
foo(); // 3

function foo() {
    console.log( 1 );
}

var foo = function() {
    console.log( 2 );
};

function foo() {
    console.log( 3 );
}
```
尽管JS引擎能有效处理多重声明，但是在同一个作用域内多重声明通常会导致意想不到的结果。

普通代码块内的函数声明通常会被提升到外层的作用域，而不是函数所在的条件分支：
```js
foo(); // "b"

var a = true;
if (a) {
   function foo() { console.log( "a" ); }
}
else {
   function foo() { console.log( "b" ); }
}
```
然而，一定要注意的是，上述行为是不可靠的，在未来的JS版本中可能会改变，因此要避免在代码块内声明函数。