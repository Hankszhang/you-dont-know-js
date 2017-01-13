# 第三章：函数 VS 块级作用域

第二章中我们讨论了嵌套作用域由一系列作为容器保存标识符（变量、函数）的“气泡”组成。这些“气泡”整齐地嵌套在彼此的里面，这种嵌套关系是在编写代码是就定义好的。

但是什么情况下会生成一个新的气泡呢？只有函数可以吗？JavaScript中的其他结构可以创建作用域气泡吗？

JavaScript中最常见的作用域是基于函数的作用域，但是也有其他的结构可以创建新作用域。我先来看看函数作用域及其含义。

## 函数的作用域

考虑如下代码：
```js
function foo(a) {
    var b = 2;
    // some code
    function bar() {
        // ...
    }
    // more code
    var c = 3;
}
```
函数foo的作用域中包括标识符a、b、c和bar，函数bar的作用域有自己的气泡。全局作用域也有自己的气泡，其中只有一个对应的标识符：foo。

因为a、b、c和bar属于foo的作用域，因此它们在foo函数之外是不能被访问到的，否则会导致`RenferenceError`错误。但是它们可以在函数foo内被访问到，而且在bar函数内也能被访问到。

函数作用域的思想是：所有变量都属于函数，在整个函数内都可以使用和重复使用这些变量（即使在内部嵌套的作用域中也可以被访问到）。这种设计方法非常有用，且可以充分利用JavaScript变量的“动态”特性来按需接收不同类型的值。

另一方面，如果不注意的话，存在于整个作用域的变量可能会产生一些意想不到的坑。

## 从普通作用域中隐藏

通常，我们定义一个函数，然后在函数内部添加代码。但是如果反过来思考：写任意一段代码片段，然后用一个函数声明来包裹它，这实际上将这段代码“藏起来”了。实际效果是在这段代码外面创建了一个“气泡”，也就意味着这段代码中所有的声明（函数或变量）都被绑定到这个新建函数的作用域上，而非原来包裹在外面的作用域。也就是说，通过将这些变量和函数包裹在函数作用域中可以把它们“藏起来”。

为什么“隐藏”变量和函数是一个很有用的技术呢？

很多情况下我们需要这种基于作用域的隐藏。根据软件设计原则“最低权限”原则，有时也叫“最小授权”或“最小暴露”原则。这一原则要求在软件设计中，如一个模块/对象的API接口，应仅仅暴露所需的最小量，而将其他所有东西“隐藏”。

将这一原则应用到作用域中。如果所有的变量和函数都在全局作用域，当然在所有嵌套作用域中都可以访问到它们，但是这就违背了“最小”原则，因为你将应该保持私有的所有变量和函数都暴露出来了，而这些变量是你不希望被外部代码访问到的。
例如：
```js
function doSomething(a) {
    b = a + doSomethingElse( a * 2 );
    console.log( b * 3 );
}
function doSomethingElse(a) {
    return a - 1;
}
var b;
doSomething( 2 ); // 15
```
变量b和函数doSomethingElse()应该是doSomething()的私有成员，将它们定义在函数外部不仅是没有必要，而且是很危险的，因为别的代码可能以意想不到的方式使用它们。一个更“合适的”设计是将这些私有细节隐藏在函数内部，如：
```js
function doSomething(a) {
    function doSomethingElse(a) {
        return a - 1;
    }

    var b;

    b = a + doSomethingElse( a * 2 );

    console.log( b * 3 );
}

doSomething( 2 ); // 15
```
现在变量b和函数doSomethingElse()在函数外部不能被访问到了，而仅受doSomething()函数的控制。

### 避免冲突
将变量和函数“隐藏”在作用域中的另一个好处是可以避免标识符之间的命名冲突。
如：
```js
function foo() {
    function bar(a) {
        i = 3; // changing the `i` in the enclosing scope's for-loop
        console.log( a + i );
    }

    for (var i=0; i<10; i++) {
        bar( i * 2 ); // oops, infinite loop ahead!
    }
}

foo();
```
for循环内调用bar函数时，bar函数内的赋值语句`i = 3`会覆盖for循环中的i变量，for循环判断中止条件时i永远小于10，从而导致无限循环。要解决这个问题，bar函数应该声明一个局部变量，如`var i = 3;`，这样就不会覆盖for循环中的同名变量i。

#### 全局“命名空间”
在全局作用域中最容易出现变量名冲入的情况。程序中加载的多个库如果没有很好的隐藏它们的内部/私有函数和变量的话，很容易发生冲突。这些库一般会在全局作用域中创建一个单例变量，通常是一个名字很独特的对象。之后这个对象就被用作这个库的“命名空间”，所有对外暴露的特定函数都是这个对象（命名空间）的属性，而不是在这些标识符之上的更高一层的词法作用域。
例如：
```js
var MyReallyCoolLibrary = {
    awesome: "stuff",
    doSomething: function() {
        // ...
    },
    doAnotherThing: function() {
        // ...
    }
};
```

#### 模块管理
另一种避免命名冲突的方案是更现代化的利用各种依赖管理器的“模块化”方法。使用这些工具的话，不会将这些库的任何标识符添加到全局作用域，而是通过依赖管理器的各种机制将这些库的标识符显式地载入另一个指定作用域。

需要注意的是这些工具并没有可以不遵守词法作用域规则的“魔法”。他们只不过是根据作用域规则来保证不会有标识符被注入到任何共享作用域中，而是保存在私有的、不易冲突的作用域中，从而避免意外的作用域冲突。

## 函数作用域

我们已经知道了将任意代码片段放入函数中可以将代码中的变量和函数声明“隐藏”在函数内部作用域，而对外层作用域不可见。尽管这种方法有用，但是也会带来一些问题。首先是我们需要声明一个命名函数，而这个函数的名字本身会“污染”外部作用域。另外，我们必须显示地根据名字调用函数来执行包裹在函数中的代码。如果函数不需要名字（或，名字不会污染作用域）且可以自动执行，将会更理想。

幸运的是JavaScript提供了一种可以同时解决这个两个问题的方案。
```js
var a = 2;

(function foo(){ // <-- insert this

    var a = 3;
    console.log( a ); // 3

})(); // <-- and this

console.log( a ); // 2
```
我们来分步看下这里发生了什么。

首先，注意函数声明前面是以`(`开始的，这虽然是一个很微小的改动，但是却是最重要的改变。这种情况下函数会被当作函数表达式，而不是一个标准的函数声明。

**注**：区别声明和表达式最简单的方法是判断语句中关键字“function”的位置，如果“function”在一条语句的最开始处，那么就是一个函数声明，否则就是一个函数表达式。

`(function foo(){..})`作为表达式意味着标识符`foo`只能在`..`表示的作用域中被访问到，在外部作用域是访问不到的。将名字foo隐藏在自身内部意味着不会污染外部作用域。

### 匿名 VS. 命名
你可能很熟悉将函数表达式作为回调参数，如：
```js
setTimeout( function(){
    console.log("I waited 1 second!");
}, 1000 );
```
这称为“匿名函数表达式”，因为`function()...`没有名字标识符。函数表达式可以是匿名的，但是函数声明不能省略名字——这是非法的JS语法。

匿名函数灵活且易于书写，很多库和工具都推崇这种惯用编程风格。但是也有几个缺点需要注意：
1. 在堆栈跟踪时匿名函数没有名字，这会增加调试的难度；
2. 如果函数没有名字，则它没法引用自己，如递归调用等。已经废弃的`arguments.callee`引用也不符合要求。另一种需要调用自己的情况是事件处理函数在触发事件之后想要对自己解除绑定；
3. 匿名函数缺少名字会影响程序的可读性。

**内联函数表达式** 强大而实用。给你的函数表达式一个名字可以很好的避免所有这些缺点，而不会引入其他缺点。因此最佳实践是总是命名你的函数表达式：
```js
setTimeout( function timeoutHandler(){ // <-- Look, I have a name!
    console.log( "I waited 1 second!" );
}, 1000 );
```

### 立即执行函数表达式
```js
var a = 2;

(function foo(){

    var a = 3;
    console.log( a ); // 3

})();

console.log( a ); // 2
```
我们可以在用括号'()'包裹的函数表达式后面再用一对括号‘()’来执行这个函数，像这样的形式`(function foo(){ ..})()`。第一对括号将这个函数变为一个表达式，第二对括号执行这个函数。

这种写法非常常见，因此社区用一个术语：IIFE 来给它命名，表示“Immediately Invoked Function Expression”（立即执行函数表达式）。

当然，IIFE 可以没有名字 —— 大多数情况下IIFE使用匿名函数表达式。但是尽管用的不多，但是给 IIFE 表达式命名总是会带来好处（上文已述）。

```js
var a = 2;

(function IIFE(){

    var a = 3;
    console.log( a ); // 3

}());

console.log( a ); // 2
```

有些人喜欢用这种IIFE 形式：`(function(){...}())`。仔细辨别这种两种形式的区别。第一种形式中，函数表达式包裹在`( )`中，然后再在最后通过`()`调用函数。在第二种形式中，用于调用函数的括号对`()`被放到了外层括号的里面。这两种形式的作用是完全一样的，纯粹根据个人喜好来选择即可。

调用立即执行函数的时候我们可以给他传入参数，如：
```js
var a = 2;

(function IIFE( global ){

    var a = 3;
    console.log( a ); // 3
    console.log( global.a ); // 2

})( window );

console.log( a ); // 2
```
这种传参形式可用于解决默认的undefined 标识符可能被覆盖的问题。为IIFE定义一个参数`undefined`， 但是给这个参数传入值，就可以保证立即执行函数中的 undefined 标识符确实等于 undefined 值：
```js
undefined = true; // setting a land-mine for other code! avoid!

(function IIFE( undefined ){

    var a;
    if (a === undefined) {
        console.log( "Undefined is safe here!" );
    }

})();
```
IIFE还有一种可以改变代码执行顺序的变体形式：
```js
var a = 2;

(function IIFE( def ){
    def( window );
})(function def( global ){

    var a = 3;
    console.log( a ); // 3
    console.log( global.a ); // 2

});
```
这段代码中，def 函数作为参数传入 IIFE 中，然后 def 函数被调用时传入参数window（定义中的形式参数 global）。

这种形式通常用在 通用模块定义中（UMD，Universal Module Definition）。

## 块级作用域

ES6之前，JS 不支持块级作用域，块级作用域中的变量可以在定义其的代码块之外使用。因此，开发者应该强制自己养成一种块级作用域习惯，在块级作用域中定义的变量只在块级作用域中使用。

### `with`
with 语句中，对象创建的作用域仅存在于 with 语句内，而不属于外层作用域。

### `try/catch`
ES3中指出在`try/catch`语句的`catch`子句中声明的变量只属于`catch`块中，例如：
```js
try {
    undefined(); // illegal operation to force an exception!
}
catch (err) {
    console.log( err ); // works!
}

console.log( err ); // ReferenceError: `err` not found
```

### `let`
ES6中引入了关键字`let`，以不同于`var`的方式声明变量。let 关键字声明的变量只属于包含该条声明语句的代码块（通常是一个{}括号对）中。如：
```js
var foo = true;

if (foo) {
    let bar = foo * 2;
    bar = something( bar );
    console.log( bar );
}

console.log( bar ); // ReferenceError
```
需要注意的是，let 关键字声明的变量不会提升到其所在作用域的顶部，因此这些变量在声明语句出现之前都是不可用的。
```js
{
    console.log( bar ); // ReferenceError!
    let bar = 2;
}
```

#### 内存回收
块级作用域有助于闭包相关的内存回收。闭包机制会在第五章中详细阐述。
考虑：
```js
function process(data) {
    // do something interesting
}

var someReallyBigData = { .. };

process( someReallyBigData );

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
    console.log("button clicked");
}, /*capturingPhase=*/false );
```
这个例子中，click 处理函数完全不会用到someReallyBigData变量，理论上，process 函数执行完后，这个占用大量内存的数据结构就会被当做垃圾回收了。然而，很有可能 JS 引擎仍会保留这个变量，因为 click 函数的作用域外有一个闭包。

块级作用域就可以很好的解决这个问题，它明确的告诉 JS 引擎不需要保留 someReallyBigData 变量：
```js
function process(data) {
    // do something interesting
}

// anything declared inside this block can go away after!
{
    let someReallyBigData = { .. };

    process( someReallyBigData );
}

var btn = document.getElementById( "my_button" );

btn.addEventListener( "click", function click(evt){
    console.log("button clicked");
}, /*capturingPhase=*/false );
```
为变量显示的声明作用域块以达到局部绑定的目的，是可以加入到你编码技能包里的一个强大技能。

#### 循环中的`let`








  

