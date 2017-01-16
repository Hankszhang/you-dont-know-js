# 第五章：作用域闭包

**JavaScript中闭包无处不在，你必须要意识到这一点并拥抱它！**

根据词法作用域来编写代码就会写出闭包，这是自然而然的。

## 闭包的实质

定义：
> 当一个函数在其词法作用域之外执行时仍能记住并访问其词法作用域时，这个函数就是闭包。

我们先通过一段代码来解释这个定义：
```js
function foo() {
    var a = 2;
    function bar() {
        console.log(a); //2
    }
    bar();
}
foo();
```
函数 bar() 通过词法作用域查找规则（这里是 RHS 引用查找）能访问外层作用域的变量a。这是闭包吗？

从纯学术角度来说，上面的代码中，函数`bar()`有一个对`foo()`作用域的闭包。换种不同的说法：`bar()`结束了`foo()`的作用域。为什么？因为`bar()`出现在 foo() 函数内部。简单明了。

但是这种方式定义的闭包不是很直观。下面看另一种使用闭包的方式：
```js
function foo() {
    var a = 2;

    function bar() {
        console.log( a );
    }

    return bar;
}

var baz = foo();

baz(); // 2 -- Whoa, closure was just observed, man.
```
这段代码中，函数bar通过词法作用域访问foo的内部作用域。但是我们将bar函数本身当做一个值返回了。当执行foo函数时，将其返回的值（内部的bar函数）赋给变量baz，然后再调用baz()函数，而实际上是调用内部函数bar()，只不过是使用了不同的引用标识符罢了。

bar函数被执行了，而且是在其被声明时所在的词法作用域之外被执行的。

通常来说，foo函数被执行之后，foo作用域内的所有内容都会消失，因为JS引擎的垃圾回收机制会在释放不再使用的内存。

闭包的神奇之处就在于它能防止被回收。实际上内部的作用域仍处于“使用中”状态，因此不会消失。**函数bar在使用它自己**。bar函数定义在foo函数内部，因此它对foo的内部作用域有一个闭包，这就使得foo函数的作用域得以保持，以便之后被bar函数引用。

**bar函数仍然能够引用到这个作用域，这个引用就是闭包。** 闭包使得函数能够一直访问的到该函数被定义时所在的词法作用域。

当然，任何将函数作为值传递然后在别的地方调用该函数的方式都是闭包。
```js
function foo() {
    var a = 2;

    function baz() {
        console.log( a ); // 2
    }

    bar( baz );
}

function bar(fn) {
    fn(); // look ma, I saw closure!
}
```
将函数作为值传递也可以是间接的：
```js
var fn;

function foo() {
    var a = 2;

    function baz() {
        console.log( a );
    }

    fn = baz; // assign `baz` to global variable
}

function bar() {
    fn(); // look ma, I saw closure!
}

foo();

bar(); // 2
```
无论我们通过什么方式将一个内部函数放到其词法作用域之外的地方，它都会保持一个对其原始定义所在作用域的引用，不管它在哪里被执行，这个闭包都会生效。

## 其他形式的闭包

```js
function wait(message) {

    setTimeout( function timer(){
        console.log( message );
    }, 1000 );

}

wait( "Hello, closure!" );
```
在JS引擎中，内置方法`setTimeout(..)`会引用一些参数，可能叫做fn或func，引擎会调用这个函数，也即调用内部的timer函数。

实际上，无论何时无论何地当你将函数作为值来传递时，这些函数都会使用闭包。定时器、事件处理函数、Ajax请求、跨窗口通信、web workers，或者任意其他异步（或同步）任务中，当你传递一个 *回调函数* 时，就会产生闭包！

按照上文的定义，IIFE模式不能算是严格的闭包：
```js
var a = 2;

(function IIFE(){
    console.log( a );
})();
```
函数IIFE被执行时所在的作用域与其引用的变量a处于同一个作用域，执行时只是进行普通的词法作用域查找，而不是通过闭包。虽然IIFE不是闭包，但是它可以创建作用域，也是最常用的用于创建可被关闭的作用域的方式之一。因此，IIFE与闭包联系非常密切。

## 循环+闭包

for循环是永不阐述闭包的一个典型例子：
```js
for (var i=1; i<=5; i++) {
    setTimeout( function timer(){
        console.log( i );
    }, i*1000 );
}
```
上述代码中，定时函数在for循环之后才被执行，尽管5个定时函数在各自的迭代中被定义，但是它们引用的是同一个作用域里的同一个变量，因此结果是都打印出6。
再尝试一下使用IIFE模式：
```js
for (var i=1; i<=5; i++) {
    (function(){
        setTimeout( function timer(){
            console.log( i );
        }, i*1000 );
    })();
}
```
IIFE的确在每个迭代中创建了一个可关闭的作用域，但是这些作用域都是空的，空的作用域等于没有，因为它什么都没干。我们将IIFE形成的作用域中保存i的值：
```js
for (var i=1; i<=5; i++) {
    (function(){
        var j = i;
        setTimeout( function timer(){
            console.log( j );
        }, j*1000 );
    })();
}
```
成功了！
另一种写法是：
```js
for (var i=1; i<=5; i++) {
    (function(j){
        setTimeout( function timer(){
            console.log( j );
        }, j*1000 );
    })( i );
}
```

### 重新审视块级作用域

上面的示例中，我们使用IIFE在每个迭代中创建一个新的块级作用域，这个块级作用域也可以通过使用let关键字来创建。let关键字声明的变量也会将声明坐在的代码块变为可关闭的作用域，如：
```js
for (var i=1; i<=5; i++) {
    let j = i; // yay, block-scope for closure!
    setTimeout( function timer(){
        console.log( j );
    }, j*1000 );
}
```
如果是在for循环头部用let声明变量，该变量会在每次迭代中被声明一次。这样下面的代码就等效于上面的代码：
```js
for (let i=1; i<=5; i++) {
    setTimeout( function timer(){
        console.log( i );
    }, i*1000 );
}
```
是不是很酷？

## 模块

除了容易理解的使用回调函数之外，模块化模式是充分践行闭包优点的另一种方式。
```js
function foo() {
    var something = "cool";
    var another = [1, 2, 3];

    function doSomething() {
        console.log( something );
    }

    function doAnother() {
        console.log( another.join( " ! " ) );
    }
}
```
这段代码没有什么特别，现在考虑这段代码：
```js
function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];

    function doSomething() {
        console.log( something );
    }

    function doAnother() {
        console.log( another.join( " ! " ) );
    }

    return {
        doSomething: doSomething,
        doAnother: doAnother
    };
}

var foo = CoolModule();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```
首先，CoolModule()只是一个函数，但是这里它必须调用来创建一个模块实例。如果不执行外部的函数，内部作用域和闭包就不会被创建。

然后，CoolModule()返回一个字面量表示的对象。这个返回的对象引用了内部的函数，但没有引用内部的数据变量，这些变量对外是隐藏的。可以将这个对象返回值作为 **模块的公共API**。这个对象返回值最后被赋给了变量foo，然后我们就可以访问API上的属性方法，如foo.doSomething()。

doSomething()和doAnother()函数有对模块内部作用域的闭包（调用CoolModule()时形成）。当我们在词法作用域的外部通过API使用这些函数时，就构成了闭包的条件。

因此，实践模块模式有两个要求：
1. 必须有一个外层包裹函数，且这个函数必须至少被调用一次（每调用一次创建一个新模块实例）。
2. 外层包裹函数必须返回至少一个内部函数，这样内部函数才能形成对私有作用域的闭包。

单例模式示例：
```js
var foo = (function CoolModule() {
    var something = "cool";
    var another = [1, 2, 3];

    function doSomething() {
        console.log( something );
    }

    function doAnother() {
        console.log( another.join( " ! " ) );
    }

    return {
        doSomething: doSomething,
        doAnother: doAnother
    };
})();

foo.doSomething(); // cool
foo.doAnother(); // 1 ! 2 ! 3
```
这里，我们将模块函数作为一个IIFE使用。

模块也是函数，也可以接受参数：
```js
function CoolModule(id) {
    function identify() {
        console.log( id );
    }

    return {
        identify: identify
    };
}

var foo1 = CoolModule( "foo 1" );
var foo2 = CoolModule( "foo 2" );

foo1.identify(); // "foo 1"
foo2.identify(); // "foo 2"
```

模块模式的另一个变体是：通过保留模块实例内部对公共API的内部引用，这样就可以从内部修改模块实例，比如增加、删除、修改方法和属性等，如：
```js
var foo = (function CoolModule(id) {
    function change() {
        // modifying the public API
        publicAPI.identify = identify2;
    }

    function identify1() {
        console.log( id );
    }

    function identify2() {
        console.log( id.toUpperCase() );
    }

    var publicAPI = {
        change: change,
        identify: identify1
    };

    return publicAPI;
})( "foo module" );

foo.identify(); // foo module
foo.change();
foo.identify(); // FOO MODULE
```

### 现代的模块

各种各样的依赖加载器、管理器基本上将模块定义模式包装成了友好的API。这里只做简要阐述：
```js
var MyModules = (function Manager() {
    var modules = {};

    function define(name, deps, impl) {
        for (var i=0; i<deps.length; i++) {
            deps[i] = modules[deps[i]];
        }
        modules[name] = impl.apply(impl, deps);
    }

    function get(name) {
        return modules[name];
    }

    return {
        define: define,
        get: get
    };
})();
```
代码的核心是`modules[name] = impl.apply(impl, deps);`。这行代码调用定义包装函数对模块进行封装（传入任意依赖），并将返回值、模块API保存到内部的模块列表中。

可以通过下面的方式来定义模块：
```js
MyModules.define( "bar", [], function(){
    function hello(who) {
        return "Let me introduce: " + who;
    }

    return {
        hello: hello
    };
} );

MyModules.define( "foo", ["bar"], function(bar){
    var hungry = "hippo";

    function awesome() {
        console.log( bar.hello( hungry ).toUpperCase() );
    }

    return {
        awesome: awesome
    };
} );

var bar = MyModules.get( "bar" );
var foo = MyModules.get( "foo" );

console.log(
    bar.hello( "hippo" )
); // Let me introduce: hippo

foo.awesome(); // LET ME INTRODUCE: HIPPO
```
### 未来的模块

ES6增加了对模块的原生语法支持。当通过模块系统加载一个文件时，ES6将其当做一个独立的模块。每个模块都可以导入其他模块或指定的API成员，也可以导出它们自己的公共API成员。

**注意**：基于函数的模块不是静态识别模式（编译器知道怎么做），因此这些模块的API语义只有在运行时才会被考虑。也就是说，实际上你可以在运行时修改模块的API。

相比之下，ES6的模块API是静态的。编译器在编译时会检查引用的模块API是否存在，如果不存在就会更早的抛出错误，而不是等到动态执行时才报错。

ES6模块不支持”内联“，每个模块必须定义在单独的文件中。浏览器/引擎有默认的”模块加载器“（可被重写，超出我们的讨论范围），会同步加载模块文件。
考虑：
**bar.js**
```js
function hello(who) {
    return "Let me introduce: " + who;
}

export hello;
```
**foo.js**
```js
// import only `hello()` from the "bar" module
import hello from "bar";

var hungry = "hippo";

function awesome() {
    console.log(
        hello( hungry ).toUpperCase()
    );
}

export awesome;
```
```js
// import the entire "foo" and "bar" modules
module foo from "foo";
module bar from "bar";

console.log(
    bar.hello( "rhino" )
); // Let me introduce: rhino

foo.awesome(); // LET ME INTRODUCE: HIPPO
```
`import`将模块API中的一个或多个成员导入到当前作用域，分别赋给一个变量。`module`导入整个模块的API并绑定到一个变量上。`export`将一个标识符（变量或函数）导出到当前模块的公共API。这些操作符可以在模块中使用多次。

模块文件内的内容就好像被一个作用域闭包包住一样，就好像前面阐述的函数闭包模块。