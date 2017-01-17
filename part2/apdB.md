# 附录B：词法作用域的this

ES6中引入了一个新的函数语法：箭头函数。先看下面的代码：
```js
var obj = {
    id: "awesome",
    cool: function coolFn() {
        console.log( this.id );
    }
};

var id = "not awesome";

obj.cool(); // awesome

setTimeout( obj.cool, 100 ); // not awesome
```
这里的问题是cool()函数的this没有绑定到对象obj上。有很多解决这个问题的办法，其中最常用的是`var self = this;`。如下：
```js
var obj = {
    count: 0,
    cool: function coolFn() {
        var self = this;

        if (self.count < 1) {
            setTimeout( function timer(){
                self.count++;
                console.log( "awesome?" );
            }, 100 );
        }
    }
};

obj.cool(); // awesome?
```
这种方式其实是通过词法作用域来实现的：self只不过是一个可以被词法作用域和闭包解析的标识符，保存了this的指向，但是它并不知道this到底发生了什么。

ES6的解决方案——箭头函数，引入了一种称为“词法this”的行为：
```js
var obj = {
    count: 0,
    cool: function coolFn() {
        if (this.count < 1) {
            setTimeout( () => { // arrow-function ftw?
                this.count++;
                console.log( "awesome?" );
            }, 100 );
        }
    }
};

obj.cool(); // awesome?
```
与普通函数相比，箭头函数对this的处理不一样。箭头函数会忽视this绑定的所有正常规则，而是永远将this值设置为当前函数被包裹的词法作用域。因此这个例子中，箭头函数的this直接”继承”自cool()函数的this。