# 第一章：`this`还是that？

## 为什么要用`this`？

先看一段使用`this`机制的代码：
```js
function identify() {
    return this.name.toUpperCase();
}

function speak() {
    var greeting = "Hello, I'm " + identify.call( this );
    console.log( greeting );
}

var me = {
    name: "Kyle"
};

var you = {
    name: "Reader"
};

identify.call( me ); // KYLE
identify.call( you ); // READER

speak.call( me ); // Hello, I'm KYLE
speak.call( you ); // Hello, I'm READER
```
再看一段普通的显示参数传递的代码：
```js
function identify(context) {
    return context.name.toUpperCase();
}

function speak(context) {
    var greeting = "Hello, I'm " + identify( context );
    console.log( greeting );
}

identify( you ); // READER
speak( me ); // Hello, I'm KYLE
```
上面两段代码的功能完全一致，但是`this`机制更加优雅，使得API的设计更简洁，使代码更易复用。

## 困惑

在解释`this`的工作原理之前，我们先消除几个对`this`的误解。

第一个误解是把`this`视为函数本身。有两种情况要在函数内部引用它自己，一是递归，二是时间处理函数在内部对自身进行解绑。JS新手一般会认为将函数（JS中所有函数都是对象）作为对象来引用就可以在函数调用之间保存状态（属性值）。尽管这种方法可行，也有一定的应用，但是本书剩下的部分会介绍其他更好的模式来保存状态。
考虑下面的代码，尝试记录foo函数被调用了几次：
```js
function foo(num) {
    console.log( "foo: " + num );

    // keep track of how many times `foo` is called
    this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
    if (i > 5) {
        foo( i );
    }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 0 -- WTF?
```
foo.count仍然为0，这是为什么呢？当执行`foo.count=0`时，实际上给函数对象foo添加了一个属性count。但是函数内部`this.count`的用法中，this实际上不是指向函数对象本身。因此，尽管属性名字是一样，但是属性所属的对象却不一样。

想要在一个函数内引用它自身，仅使用this是不够的。一般需要通过一个指向该函数对象的词法标识符（变量）来引用他自己。
例如：
```js
function foo() {
    foo.count = 4; // `foo` refers to itself
}

setTimeout( function(){
    // anonymous function (no name), cannot
    // refer to itself
}, 10 );
```
另外一种强制`this`指向foo函数对象的方式是：
```js
function foo(num) {
    console.log( "foo: " + num );

    // keep track of how many times `foo` is called
    // Note: `this` IS actually `foo` now, based on
    // how `foo` is called (see below)
    this.count++;
}

foo.count = 0;

var i;

for (i=0; i<10; i++) {
    if (i > 5) {
        // using `call(..)`, we ensure the `this`
        // points at the function object (`foo`) itself
        foo.call( foo, i );
    }
}
// foo: 6
// foo: 7
// foo: 8
// foo: 9

// how many times was `foo` called?
console.log( foo.count ); // 4
```

第二个误解是将`this`理解为函数的作用域。这是一个对错参半的解释。需要明白的是，`this`不表示函数的词法作用域。实际上，在JS引擎内部，作用域是一个类似于包含所有可用标识符的对象。但是这个作用域“对象”不能对JS代码访问到。

## `this`是什么？

`this`不是在编写代码时绑定，而是在运行代码时绑定。它根据函数被调用的语境而确定。`this`绑定与函数在哪里定义无关，但是却与函数被调用的行为相关。

当一个函数被调用时，会创建一个激活记录，也称为执行上下文。这个记录包含了函数在哪里（堆栈）被调用、被调用的方式、传入了什么参数等信息。`this`引用就是这个记录的一个属性，它会在函数执行过程中被使用。