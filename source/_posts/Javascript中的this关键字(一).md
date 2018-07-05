---
title: Javascript中的this关键字(一)
date: 2018-06-27 16:04:52
categories:
- Front-End
tags: 
- Javascript
---
该文章来自 [David Shariff](http://davidshariff.com/) 的 [Javascript's 'this' keyword](http://davidshariff.com/blog/javascript-this-keyword/#first-article)，因为正在练习英语，所以抱着试试的心态翻译一下此文章，也加入了自己所理解的一些概念，肯定会有理解不到位的地方，请多包涵。

**正文开始**
在开发过程中，虽然我们经常使用Javascript中的 `this` 关键字 ，但是它经常使人感到迷惑并且误解。`this` 真实的含义到底是什么？怎样正确的理解？

这篇文章试图搞清楚和解释这个问题。

`this` 关键字对于那些使用其他语言编程的人来说并不新鲜，它通常指通过类的构造函数实例化类时创建的新对象。举个例子，如果有一个 `Boat()` 类，其中有一个 `moveBoat()` 方法，那么我们可以确定 `moveBoat()` 方法中的 `this` 指向的是新创建的 `Boat()`。

在Javascript中，当我们使用 `new` 关键字时，**Function 构造函数** 也有这样的概念，然而这并不是全部。 `this` 通常指来自不同的执行环境（context）中的不同对象。我们来看一下例子：
```Javascript
// global scope 全局环境
foo = 'abc';
console.log(foo);  // => abc

this.foo = 'def';
console.log(foo)  // => def
```
当你在全局环境（不是在function中）中使用 `this` 关键字，它总是指向全局对象。现在让我们看一下 `this` 在function中的例子：
```Javascript
let boat = {
  size: 'normal',
  boatInfo: function() {
    console.log(this === boat);
    console.log(this.size);
  }
}

boat.boatInfo();  // => true, 'normal'

let bigBoat = {
  size: 'big'
};

bigBoat.boatInfo = boat.boatInfo;
bigBoat.boatInfo();  // => false, 'big'
```
那么在上面的代码中 `this` 是如何确定的？我们可以看一下 `boat` 对象中的属性 `size` 和 `boatInfo()` 方法。在 `boatInfo()` 中，会打印两个值，前者打印出Boolean值（如果 `this` 的值是对象 `boat`），后者打印出 `size` 的值。调用 `boat.boatInfo()` 方法就会看到 `this` 的值是对象 `boat`，对象 `boat` 属性 `size` 的值是normal。

我们创建另外一个对象 `bigBoat` ，该对象只有一个 `size` 属性，值为big。我们使用 `bigBoat.boatInfo = boat.boatInfo` ，将 `boat` 中的 `boatInfo` 方法复制给 `bigBoat` 中的 `boatInfo`。当我们调用 `bigBoat.boatInfo()` 方法时，我们看到 `this` 并不等于对象 `boat`，属性 `size` 的值为big。为什么会这样？`boatInfo()` 中的值是如何改变的？

首先你要知道，任何函数的值都不是静态的，每次调用一次函数时都要确定这个值，但是在函数实际执行之前，它是代码。*函数内部的值实际上是由调用函数的父作用域提供的*，更重要的是，实际的函数语法是如何编写的。

不管函数什么时候被调用，我们必须看一下括号 `()` 的左边。如果括号左边能够看到引用，那么传递给函数调用的 `this` 的值就是该对象所属的值，否则就是全局对象。
换句话说，`this` 的值是最后调用它的对象，也就是看它执行的时候是谁调用的。我们来看一下例子：
```Javascript
function bar() {
  console.log(this);
}
bar();  // => global，因为bar()方法被调用时属于全局对象

let foo = {
  baz: function() {
    console.log(this);
  }
}
foo.baz();  // => foo，因为baz()方法被调用时属于foo对象
```
相信上面的代码很容易理解。
我们还可以使用同一个函数来改变 `this` 的值，通过两种不同的方式编写调用语法，使其变得更复杂：
```Javascript
let foo = {
  baz: function() {
    console.log(this);
  }
}
foo.baz();  // foo，因为baz()方法被调用时属于foo对象

var anotherBaz = foo.baz;
anotherBaz(); //  global，因为baz()方法被调用时属于全局对象，不是foo
```
我们看到 `baz()` 中 `this` 的值每次都是不同的，因为它在语法上以两种不同的方式被调用。现在我们看一下在多层嵌套的对象中 `this` 的值：
```Javascript
const anum = 0;

let foo = {
  anum: 10,
  baz: {
    anum: 20,
    bar: function() {
      console.log(this.anum);
    }
  }
}
foo.baz.bar();  // => 20，因为括号左侧是 bar，当被调用时属于baz对象

let hello = foo.baz.bar;
hello();  // => 0，因为括号左侧是 hello，当被调用时属于全局对象
```
另一个经常被问到的问题是关键字 `this` 如何在事件处理程序中(event handler)确定的？答案是在事件处理程序中 `this` 总是指向被触发的元素。我们来看一下例子：
```Javascript
<div id="test">I am an element with id #test</div>

function doPrint() {
  console.log(this.innerHtml);
}

doPrint();  // => undefined

let myElem = document.getElementById('test');
myElem.onclick = doPrint;

console.log(myElem.onclick === doPrint);
myElem.onclick();  // => I am an element with id #test
```
我们看到 `doPrint()` 第一个被调用，打印的值为undefined，因为它属于全局对象。然后我们将 `doPrint` 方法复制给了 `myElem.onclick` 事件 ，这基本上意味着无论 `onclick()` 何时被触发，它都是 `myElem` 对象到一个方法，意味着 `this` 的值始终是 `myElem` 对象。

关于这个话题我想说的最后一点，`this` 的值可以使用 `call()` 和 `apply()` 方法手动设置。同样令人感兴趣的是，当在Function 构造函数中调用 `this` 时，`this` 指的是Function 构造函数中所有实例中新创建的对象。原因是Function 构造函数是用 `new` 关键字来调用的，使用 `new` 关键字创建一个新对象时，Function 构造函数中的 `this` 始终指向刚刚创建的新对象。

#### 总结
希望这篇文章能够解释清楚了所有关于对 `this` 关键字的误解，你可以正确的理解 `this` 的值。我们现在知道了 `this` 的值绝对不是静态的，并且它拥有不同的值取决于怎样调用函数。