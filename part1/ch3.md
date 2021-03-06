# 第三章：深入YDKJS

本系列书籍是讲什么的？简单的说，本系列书籍的目的是认真学习 *JavaScript的所有部分*，而不是这门语言的一部分而已，如有些人所谓的“精粹部分”，也不仅仅是完成工作所需掌握的最少知识点。

其他语言的优秀开发者都希望投入精力去学习它们主要使用的语言的大部分或所有部分内容，但是JS开发者似乎与众不同地不愿意学号这门语言。这不是一个好的现象，我们也不应该让这继续成为常态。

*You Don't Know JS (YDKJS)* 系列与你阅读的大部分传统学习JS的教材形成鲜明对比。它让你超越你能理解的部分，对你遇到的每一个问题都深入得询问“为什么”。你准备好接受挑战了吗？

我将用这最后一章简要概括本系列书籍你能学到的内容，以及在学习YDKJS的基础上如何最有效地夯实JS基础。

## 作用域&闭包

你需要快速掌握的最基础的知识可能就是JavaScript中的变量作用域是怎么工作的。对作用域一知半解是远远不够的。

*作用域&闭包* 一书首先纠正一个常见的误解：JS是“解释型语言”因此不需要编译。大错特错。

JS引擎会在将要执行之前（而不是执行期间）编译代码。所以我们会更深入解释编译器编译代码的方式，理解它是如何找到并处理变量和函数声明的。学习过程中，我们会掌握JS变量作用域管理中的典型比喻——“提升”。

理解了“词法作用域”之后，我们会在其基础上在最后一张探索闭包的知识。闭包可能是JS所有概念中最重要的一个，但是如果你没有先熟练掌握作用域，也不可能很好的掌握闭包。

闭包的一个重要应用是模块模式。模块模式可能是JS中最流行的代码组织模式，因此你也应该优先掌握她。

## this&对象原型

可能关于JavaScript最流行的也是被误解最多的就是函数中`this`关键字的指向问题。彻底被错误理解！

this关键字基于所在函数被执行的方式而动态改变指向，大概有四种简单的规则来理解并确定this的指向。

与this关键字紧密相关的是对象原型机制，它是一个属性的查找链，与词法作用域链类似。但是对原型的理解存在另一个对JS的巨大误解：模拟（伪造）类和继承的思想。

不幸的是，将类和继承的设计模型思想引入JavaScript是你尝试做得最坏的事情之一，因为虽然这种语法可能会让你误认为有类的存在，但是实际上原型链机制与类的行为基本上完全相反。

问题是是否忽略这种不匹配并当作在实现“继承”更好，还是学习并掌握对象原型的实际工作原理更合适。后者称为“行为委托”更合适。

这不仅仅是对于语法的偏好问题。委托是完全不同且更强大的设计模式，它可以替代类和继承的设计。这些结论绝对会出现在跟完整的JavaScript生命周期主题相关的其他博客文章、书籍以及会议演讲里面。

我的结论——委托比继承更好——不是因为不喜欢这门语言及其语法，而是希望这门语言的能力能够被正确地衡量，希望冲散无止尽的不解与困惑。

但是我认为原型和委托是一个比我在这里阐述的高深的多的主题。如果你准备重塑你对JS的“类”和“继承”的所有认识，希望你能仔细阅读 *this&对象原型* 一书的第4-6章。

## 类型&语法

本系列的第三部分主要讨论另一个充满争议性的话题：类型强制转换。可能没有什么主题能比隐式类型转换给JS开发者带来更多的困扰和挫败感。

到目前为止，普遍的看法是隐式类型转换是这门语言的“糟粕”应不惜代价避免使用。实际上，更有人认为这是JS设计上的一个“错误”。的确，现在有一些工具专门用来检查你的代码是否使用了像强制类型转换这样不好的语法。

但是强制类型转换真的这么困惑、这么不好、这么危险，以致于一旦用了它你的代码就注定有问题吗？

我的答案是：不是！第1-3章阐述类型和值的实际工作原理，第4章将主要讨论这个争论，完整地解释强制类型转换的原理。我们会明白强制类型转换的哪些部分确实是不可控的，哪些部分实际上是很有意义的。

但是我不只是认为强制类型转换是合理且值得掌握，我也认为它是非常有用且是一个应该在代码中使用的被低估的工具。我认为，如果使用得当，强制类型转换不仅有用而且会使代码更好。所有的反对者和怀疑者肯定会反对这种立场，但是我相信这是提升你的JS技能的关键之一。

你是想继续随大众所想，还是将所有假设放一边，以全新的视角来看待强制类型转换呢？ *类型&语法* 会改变你的想法的。

## 异步&性能

本系列的前三部分主要讨论JS语言的核心机制，而第四部分在语言机制的之上讨论异步编程模式。异步不仅是提升应用性能的关键，也逐渐成为提升可写性和可维护性的关键因素。

本书首先梳理了一些易混淆的术语和概念，如“异步”、“平行”和“并发”，并详细解释了这些东西为什么应用或为什么不应用与JS。

随后，我们探讨了异步的主要实现方式——回调。但是我们很快就会发现单独的回调不能完全满足现代异步编程的要求。我们指出了纯回调编程的两点不足：*控制翻转*（IoC）信任损失和缺失线性理论。

为了弥补这两点不足，ES6引入了两种新机制（模式）：promises和generators。

Promises是一个与时间无关的用来包装“将来值”的包装器，它允许你在不知道值是否已经准备好的情况下推导并组合代码。另外，通过受信任且可组合的promise机制对回调进行路由控制，有效地解决了IoC信任问题。

Generators引入一个执行JS函数的新模式，由此，generator可以在yield点暂停，然后在异步地恢复执行。这种暂停-恢复的能力使异步成为可能，根据不同的场景异步执行generator中顺序排列的代码。通过这种方式，可以解决回调中的非线性、非本地跳转的问题，因而是我们的异步代码看起来更像同步，从而看起来更合理。

但是在JavaScript中组合使用promises和generators才能使我们的异步编程模式发挥最大的效率。实际上，ES7及以后将引入的异步的大部分复杂性就是基于此。想要在异步的世界中高效地编程，你需要熟练的掌握promises和generators。

promises和generators模式讨论的如何让我们的程序运行更多地并发，从而在短时间内完成更多的操作。JS还有其他很多性能优化方面值得我们去探索。

第5章深入探讨的主题有：Web Workers的程序并发性、SIMD数据并行性、ASM.JS底层优化技术等。第6章从基准技术角度探讨性能优化，包括哪些性能需要考虑而哪些可以忽略。

编写高效的JavaScript意味着编写的代码能够在广泛的浏览器和环境中动态运行。这需要大量的复杂而详细的规划，努力让程序从“能运行”变成“很好地运行”。

*异步&性能* 帮助你掌握编写合理且高性能的JavaScript代码的所有工具和技能。

## ES6&未来

不管此时你认为自己对JavaScript掌握的又多好，事实是JavaScript总是在不停的发展，而且发展的速度非常迅速。这个事实也是本系列书籍的精神所在，去拥抱我们永远不可能完全掌握的JS的每一部分，因为当你掌握了所有部分时，又会有新的东西需要去学习。

这一部分主要讨论这门语言的短期和中期发展愿景，不仅有大家已知的东西，如ES6，也有未来可能加入的东西。

虽然本系列所有书都包含了编写时的JavaScript的最新状态，也即ES6半采纳状态，本系列主要聚焦于ES5的讨论。现在，我们需要把我们的注意力转移到ES6,ES7...

由于本书编写时ES6已经接近完成，本部分首先将ES6的具体内容划分为几个主要类别，包括新语法、新数据结构（集合）、新处理能力和新API等。我们会从不同的细节水平讨论每一个新ES6特性，也会回顾在本系列其他部分涉及的细节。

ES6中一些令人兴奋的值得阅读的内容包括：解构、默认参数值、symbols、方法简写、计算后的属性、箭头函数、块级作用域、promises、generators、迭代器、模块、代理、弱映射、等等等等！

本书的前一部分是一份关于你在接下来的几年中编写和探索的改进的JavaScript的所有内容的学习路线。

后一部分则简单介绍在不远的将来JavaScript中可能会加入的内容。这里最重要的实现是post-ES6，JS更像是一个特征一个特征，而不是一个版本一个版本向前发展，这意味着我们预期的不久的将来的内容会来得比我们想象中的快。

JavaScript的未来一片光明，现在不正是我们开始学习它的好时机吗？