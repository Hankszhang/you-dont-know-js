# 第二章：词法作用域

目前主要有两种模型来解释作用域的工作原理：**词法作用域** 和 **动态作用域**。我们这里主要讨论JavaScript使用的词法作用域，附录A中对动态作用域进行了阐述。

## 词法分析
正如我们在第一章中讨论过的，标准的语言编译器的第一步一般都是词法分析，它正是理解词法作用域的基础。

词法作用域是在进行词法分析时定义的作用域。也就是说，词法作用域基于你编写代码时创建的变量和作用域块，因此在分词器执行你的代码时它是不可更改的。

**注**：采取某些方法可以欺骗词法作用域，从而在分词之后还可以修改作用域，但是不提倡这么做。

考虑下面这段代码：
```js
function foo(a) {
    var b = a * 2;
    function bar(c) {
        console.log( a, b, c );
    }
    bar(b * 3);
}
foo( 2 ); // 2 4 12
```
这段代码中包含了三层嵌套作用域。可以将其想象成三个依次包裹的气泡。
<img src="/assets/fig5.png" width="500">

**气泡1**：包含全局作用域，只有一个标识符：foo；

**气泡2**：包含函数foo的作用域，有三个标识符：a、bar和b；

**气泡3**：包含函数bar的作用域，只有一个标识符：c

作用域气泡在作用域块被书写的位置定义，一个嵌套在另一个内。气泡bar被完全包含在气泡foo内，因为这正是我们定义函数bar的位置。

### 查找
上文中的这些气泡的结构和相对位置，很好的解释了引擎是如何找到它需要的标识符的。

上面的代码中，引擎执行`congsole.log(..)`语句时，会查找三个相关的变量a、b和c。首先从最内层的气泡，也即bar函数的作用域，开始查找。没找到a，因此往上一层，来到上一层嵌套的气泡，也即foo函数的作用域，在这里找到了a，因此直接使用它。查找b也是同样的过程，但是对于c，可以直接在bar函数内找到。

**作用域查找一旦找到第一个匹配则立即停止**。在嵌套作用域的不同层级可以使用相同名字的标识符，内部的标识符会“遮蔽”外部的同名标识符。不论有无“遮蔽”效应，作用域查找总是从当前正在执行的作用域开始，然后一次向外/向上直到找到第一个匹配，则立即停止。

**注**：全局变量是全局对象（浏览器中是window对象）的的属性，因此也可以不直接通过词法作用域方式引用全局变量，而是作为全局对象的属性间接引用：window.a。这种方法使得全局变量即使被遮蔽了依然可以被访问到。

不管函数在哪里被调用、也不管是怎样被调用，它的词法作用域**只**由其被声明的位置确定。

## 欺骗词法

JavaScript中有两种方法来在执行时“修改”函数的词法作用域，但是JS社区中都不提倡使用这两种方法，因为欺骗词法作用域会影响程序性能。

### `eval`
JavaScript中的`eval(..)`函数接受一个字符串参数，将其当前程序中运行这段字符串的内容，就好像这代字符串是写在这里一样。换句话说，你可以在你写的代码中用程序生成代码并且运行，就好像这段代码在你写程序时就在这里一样。

`eval(..)`执行之后的后续代码时，引擎不会知道，也不会关心之前执行的代码是动态插入的，因此会修改词法作用域环境。引擎会按照正常情况进行词法查找。
例如下面的代码：
```js
function foo(str, a) {
    eval( str ); // cheating!
    console.log( a, b );
}
var b = 2;
foo( "var b = 3;", 1 ); // 1, 3
```
eval(..)函数被调用时，字符串"var b = 3;"被执行了，就好像它是一直就在这里的代码一样。而这段代码中正好声明了变量b，它会修改函数foo的已有的词法作用域。实际上，这段代码创建了一个变量b，这个变量会把全局作用域中的b遮蔽掉。

默认情况下，如果eval(..)执行的字符串包含一个或多个声明（变量或函数）时，它会修改eval(..)所在位置的词法作用域。技术上来讲，可以通过一些技巧间接地调用eval，这会使得它在全局作用域的上下文中执行，并修改全局作用域。

JavaScript中的一些其他函数也有类似于eval(..)的作用。`setTimeout(..)`和`setInterval(..)`可以接受一个字符串作为第一个参数，字符串的内容被当作动态生成的函数执行。这种方法已经被废弃了，不要再使用！

类似地，函数构造函数`new Function(..)`的最后一个参数也可以接受一个字符串，并将其转为动态生成的函数（前面的参数则作为新函数的命名参数）。这个函数构造器的语法比eval(..)更安全一些，但在代码中也应避免使用。

### `with`
with关键字欺骗词法作用域的方式现在已经被弃用。下面也简单介绍下它是如何与词法作用域打交道并影响它的。

with一般作为引用一个对象的多个属性而不重复引用对象本身时的一种简写方式。例如：
```js
var obj = {
    a: 1,
    b: 2,
    c: 3
};

// more "tedious" to repeat "obj"
obj.a = 2;
obj.b = 3;
obj.c = 4;

// "easier" short-hand
with (obj) {
    a = 3;
    b = 4;
    c = 5;
}
```
但是实际使用过程中，with带来的问题比访问对象属性带来的便捷性多得多。例如：
```js
function foo(obj) {
    with (obj) {
        a = 2;
    }
}

var o1 = {
    a: 3
};

var o2 = {
    b: 3
};

foo( o1 );
console.log( o1.a ); // 2

foo( o2 );
console.log( o2.a ); // undefined
console.log( a ); // 2 -- Oops, leaked global!
```
`with`语句接受一个对象，这个对象可以有零个或多个属性，并将这个对象当作一个 *完全隔离的词法作用域*，因此在这个“作用域”中这个对象的属性会被当作词法上定义的变量。

**注意**：尽管with代码块将对象当作一个词法作用域，with代码块中正常的var变量声明不会被包含在with语句的作用域内，而是声明在with语句所在的函数中。

这就很好理解了，当我们将o1传给with语句时，它声明的“作用域”就是o1，这个“作用域”有一个对应于o1.a属性的“标识符”。但是当我们用o2作为“作用域”时，它没有“标识符”a，因此会按正常的LHS标识符规则进行查询。

o2的“作用域”、foo(..)的作用域，甚至是全局作用域都没有a标识符，因此当a=2被执行时，会自动在全局作用域创建这个变量。

### 性能
eval和with都是通过在运行时修改或创建新词法作用域来达到欺骗编写代码时定义好的词法作用域的目的。那么这有什么影响呢？

JavaScript引擎在编译阶段做了很多性能优化，其中有些优化要求JS引擎在分词时能基本上做到静态分析代码，并需要预先确定所有变量和函数声明的位置，这样在执行的时候就可以很快的解析变量。但是如果引擎发现代码中有eval或with语句，它将不得不先假设所有已知的变量位置都是无效的，因为引擎在词法分析时不知道会将什么代码传入eval中来修改词法作用域，也不知道传入with语句的对象的内容是什么。

换言之，如果使用了eval或with语句，引擎做得这些优化就没有意义了，所以它干脆不做任何优化。因此不管在你的代码的任何位置出现了eval或with语句，你的整个代码都会运行的更慢。