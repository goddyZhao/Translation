说明
--------
* * *
此文译自[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/) 的文章[Scope Chain](http://dmitrysoshnikov.com/ecmascript/chapter-5-functions/)    


概要
--------
* * *
本文将给大家介绍ECMAScript中的一般对象之一——函数。我们将着重介绍不同类型的函数以及不同类型的函数是如何影响上下文的变量对象以及函数的作用域链的。
我们还会解释经常会问到的问题，诸如：“不同方式创建出来的函数会不一样吗？（如果会，那么到底有什么不一样呢？）”：  

<pre><code>var foo = function () {
  ...
};</code></pre>

上述方式创建的函数和如下方式创建的有什么不同？  
<pre><code>function foo() {
  ...
}</code></pre>

如下代码中，为啥一个函数要用括号包起来呢？  
<pre><code>(function () {
  ...
})();</code></pre>

由于本文和此前几篇文章都是有关联的，因此，要想完全搞懂这部分内容，建议先去阅读[第二章-变量对象](http://goddyzhao.tumblr.com/post/11141710441/variable-object)
以及[第四章-作用域链](http://goddyzhao.tumblr.com/post/11259644092/scope-chain)。  

下面，来我们先来介绍下函数类型。  


函数类型
--------
* * *
ECMAScript中包含三类函数，每一类都有各自的特性。  


函数声明（Function Declaration）
--------
* * *
>  函数声明（简称FD）是指这样的函数  
>    
>  *  有函数名  
>  *  代码位置在：要么在程序级别或者直接在另外一个函数的函数体（FunctionBody）中    
>  *  在进入上下文时创建出来的  
>  *  会影响变量对象  
>  *  是以如下形式声明的  
<pre><code>function exampleFunc() {
  ...
}</code></pre>

这类函数的主要特性是：只有它们可以影响变量对象（存储在上下文的VO中）。此特性同时也引出了非常重要的一点（变量对象的天生特性导致的） —— 它们在执行代码阶段就已经存在了（因为FD在进入上下文阶段就收集到了VO中）。    

下面是例子（从代码位置上来看，函数调用在声明之前）：  
<pre><code>foo();
 
function foo() {
  alert('foo');
}</code></pre>

从定义中还提到了非常重要的一点 —— 函数声明在代码中的位置：  
<pre><code>// 函数声明可以直接在程序级别的全局上下文中
function globalFD() {
  // 或者直接在另外一个函数的函数体中
  function innerFD() {}
}</code></pre>

除了上述提到了两个位置，其他位置均不能出现函数声明 —— 比方说，在表达式的位置或者是代码块中进行函数声明都是不可以的。  

介绍完了函数声明，接下来介绍_函数表达式_（_function expression_）。  


函数表达式
--------
* * *
>  函数表达式（简称：FE）是指这样的函数：  
>  
>  *  代码位置必须要在表达式的位置  
>  *  名字可有可无  
>  *  不会影响变量对象  
>  *  在_执行代码_阶段创建出来  

这类函数的主要特性是：它们的代码总是在表达式的位置。最简单的表达式的例子就是赋值表达式：  
<pre><code>var foo = function () {
  ...
};</code></pre>

上述例子中将一个匿名FE赋值给了变量“foo”，之后该函数就可以通过“foo”来访问了—— foo()。  

正如定义中提到的，FE也可以有名字：  
<pre><code>var foo = function _foo() {
  ...
};</code></pre>

这里要注意的是，在FE的外部可以通过变量“foo”——foo()来访问，而在函数内部（比如递归调用），还可以用“_foo”（译者注：但在外部是无法使用“_foo”的）。  

当FE有名字的时候，它很难和FD作区分。不过，如果仔细看这两者的定义的话，要区分它们还是很容易的： FE总是在表达式的位置。
如下例子展示的各类ECMAScript表达式都属于FE：  
<pre><code>// 在括号中(grouping operator)只可能是表达式
(function foo() {});
 
// 在数组初始化中 —— 同样也只能是表达式
[function bar() {}];
 
// 逗号操作符也只能跟表达式
1, function baz() {};</code></pre>

定义中还提到FE是在执行代码阶段创建的，并且不是存储在变量对象上的。如下所示：  
<pre><code>// 不论是在定义前还是定义后，FE都是无法访问的
// (因为它是在代码执行阶段创建出来的),
 
alert(foo); // "foo" is not defined
 
(function foo() {});
 
// 后面也没用，因为它根本就不在VO中
 
alert(foo);  // "foo" is not defined</code></pre>

问题来了，FE要来干嘛？其实答案是很明显的 —— 在表达式中使用，从而避免对变量对象造成“污染”。最简单的例子就是将函数作为参数传递给另外一个函数：  
<pre><code>function foo(callback) {
  callback();
}
 
foo(function bar() {
  alert('foo.bar');
});
 
foo(function baz() {
  alert('foo.baz');
});</code></pre>

上述例子中，部分变量存储了对FE的引用，这样函数就会保留在内存中并在之后，可以通过变量来访问（因为变量是可以影响VO的）：  
<pre><code>var foo = function () {
  alert('foo');
};
 
foo();</code></pre>

如下例子是通过创建一个封装的作用域来对外部上下文隐藏辅助数据（例子中我们使用FE使得函数创建后就立马执行）：  
<pre><code>var foo = {};
 
(function initialize() {
 
  var x = 10;
 
  foo.bar = function () {
    alert(x);
  };
 
})();
 
foo.bar(); // 10;
 
alert(x); // "x" is not defined</code></pre>

我们看到函数“foo.bar”（通过其\[\[Scope\]\]属性）获得了对函数“initialize”内部变量“x”的访问。
而同样的“x”在外部就无法访问到。很多库都使用这种策略来创建“私有”数据以及隐藏辅助数据。通常，这样的情况下FE的名字都会省略掉：  
<pre><code>(function () {
 
  // 初始化作用域
 
})();</code></pre>

还有一个FE的例子是：在执行代码阶段在条件语句中创建FE,这种方式也不会影响VO：  
<pre><code>var foo = 10;
 
var bar = (foo % 2 == 0
  ? function () { alert(0); }
  : function () { alert(1); }
);
 
bar(); // 0</code></pre>


“有关括号”的问题
--------
* * *
让我们回到本文之初，来回答下此前提到的问题 —— “为什么在函数创建之后立即进行函数调用时，需要用括号将其包起来？”。
要回答此问题，需要先介绍下关于表达式语句的限制。  

标准中提到，表达式语句（_ExpressionStatement_）不能以左大括号**{**开始 —— 因为这样一来就和代码块冲突了，
也不能以**function**关键字开始，因为这样一来又和函数声明冲突了。比方说，以如下所示的方式来定义一个立马要执行的函数：  
<pre><code>function () {
  ...
}();
 
// or with a name
 
function foo() {
  ...
}();</code></pre>

对于这两种情况，解释器都会抛出错误，只是原因不同。  

如果我们是在全局代码（程序级别）中这样定义函数，解释器会以函数声明来处理，因为它看到了是以**function**开始的。
在第一个例子中，会抛出**语法错误**，原因是既然是个函数声明，则缺少函数名了（一个函数声明其名字是必须的）。  

而在第二个例子中，看上去已经有了名字了（foo），应该会正确执行。然而，这里还是会抛出**语法错误** —— 组操作符内部缺少表达式。
这里要注意的是，这个例子中，函数声明后面的**()**会被当组操作符来处理，而非函数调用的**()**。因此，如果我们有如下代码：  
<pre><code>// "foo" 是函数声明
// 并且是在进入上下文的时候创建的
 
alert(foo); // function
 
function foo(x) {
  alert(x);
}(1); // 这里只是组操作符，并非调用!
 
foo(10); // 这里就是调用了, 10</code></pre>  

上述代码其实就是如下代码：  
<pre><code>// function declaration
function foo(x) {
  alert(x);
}
 
// 含表达式的组操作符
(1);
 
// 另外一个组操作符
// 包含一个函数表达式
(function () {});
 
// 这里面也是表达式
("foo");
 
// etc</code></pre>

当这样的定义出现在语句位置时，也会发生冲突并产生语法错误：  
<pre><code>if (true) function foo() {alert(1)}</code></pre>

上述结构根据标准规定是不合法的。（表达式是不能以**function**关键字开始的），然而，正如我们在后面要看到的，没有一种实现对其抛出错误，
它们各自按照自己的方式在处理。  

讲了这么多，那究竟要怎么写才能达到创建一个函数后立马就进行调用的目的呢？
答案是很明显的。它必须要是个函数表达式，而不能是函数声明。而创建表达式最简单的方式就是使用上述提到的组操作符。因为在组操作符中只可能是表达式。
这样一来解释器也不会纠结了，会果断将其以FE的方式来处理。这样的函数将在执行阶段创建出来，然后立马执行，随后被移除（如果有没有对其的引用的话）：  
<pre><code>(function foo(x) {
  alert(x);
})(1); // 好了，这样就是函数调用了，而不再是组操作符了，1</code></pre>

要注意的是，在下面的例子中，函数调用，其括号就不再是必须的了，因为函数本来就在表达式的位置了，解释器自然会以FE来处理，并且会在执行代码阶段创建该函数：  
<pre><code>var foo = {
 
  bar: function (x) {
    return x % 2 != 0 ? 'yes' : 'no';
  }(1)
 
};
 
alert(foo.bar); // 'yes'</code></pre>

因此，对“括号有关”问题的完整的回答则如下所示：  
>  如果要在函数创建后立马进行函数调用，并且函数不在表达式的位置时，括号就是必须的 —— 这样情况下，其实是手动的将其转换成了FE。
>  而当解释器直接将其以FE的方式处理的时候，说明FE本身就在函数表达式的位置 —— 这个时候括号就不是必须的了。  

另外，除了使用括号的方式将函数转换成为FE之外，还有其他的方式，如下所示：  
<pre><code>1, function () {
  alert('anonymous function is called');
}();
 
// 或者这样
!function () {
  alert('ECMAScript');
}();
 
// 当然，还有其他很多方式
 
...</code></pre>

不过，括号是最通用也是最优雅的方式。  

顺便提下，组操作符既可以包含没有调用括号的函数，又可以包含有调用括号的函数，这两者都是合法的FE：  
<pre><code>(function () {})();
(function () {}());</code></pre>


实现扩展： 函数语句
--------
* * *
看如下代码，符合标准的解释器都无法解释这样的代码：  
<pre><code>if (true) {
 
  function foo() {
    alert(0);
  }
 
} else {
 
  function foo() {
    alert(1);
  }
 
}
 
foo(); // 1 还是 0 ? 在不同引擎中测试</code></pre>

这里有必要提下：根据标准，上述代码结构是不合法的，因为，此前我们就介绍过，函数声明是不能出现在代码块中的（这里if和else就包含代码块）。
此前提到的，函数声明只能出现在两个位置： 程序级别或者另外一个函数的函数体中。  

为什么这种结构是错误的呢？因为在代码块中只允许_语句_。函数要想在这个位置出现的唯一可能就是要成为_表达式语句_。
但是，根据定义表达式语句又不能以左大括号开始（这样会与代码块冲突）也不能以**function**关键字开始（这样又会和FD冲突）。  

然而，在错误处理部分，标准允许实现对程序语法进行扩展。而上述例子就是其中一种扩展。目前，所有的实现中都不会对上述情况抛出错误，都会以各自的方式进行处理。  

因此根据标准，上述if-else中应当需要FE。然而，绝大多数实现中都在进入上下文的时候在这里简单地创建了FD，并且使用了最后一次的声明。
最后“foo”函数显示了1，尽管理论上else中的代码根本不会被执行到。  

而SpiderMonkey（TraceMonkey也是）实现中，会将上述情况以两种方式来处理： 一方面它不会将这样的函数以函数声明来处理（也就意味着函数会在执行代码阶段才会创建出来），
然而，另外一方面，它们又不属于真正的函数表达式，因为在没有括号的情况是不能作函数调用的（同样会有解析错误——和FD冲突），它们还是存储在变量对象中。  

我认为SpiderMonkey单独引入了自己的中间函数类型——（FE+FD），这样的做法是正确的。这样的函数会根据时间和对应的条件正确创建出来，不像FE。
和FD有点类似，可以在外部对其进行访问。SpiderMonkey将这种语法扩展命名为函数语句（_Function Statement_）（简称FS）；这部分理论在MDC中有[具体的介绍](https://developer.mozilla.org/En/Core_JavaScript_1.5_Reference:Functions#Conditionally_defining_a_function)。
JavaScript的发明者 Brendan Eich也[提到过](https://mail.mozilla.org/pipermail/es-discuss/2008-February/005314.html)这类函数类型。  


有名字的函数表达式的特性（NFE）
--------
* * *
当FE有名字之后（named function expression，简称：NFE），就产生了一个重要的特性。
正如在定义中提到的，函数表达式是不会影响上下文的变量对象的（这就意味着不论是在定义前还是在定义后，都是不可能通过名字来进行调用的）。
然而，FE可以通过自己的名字进行递归调用：  
<pre><code>(function foo(bar) {
 
  if (bar) {
    return;
  }
 
  foo(true); // "foo" name is available
 
})();
 
// but from the outside, correctly, is not
 
foo(); // "foo" is not defined</code></pre>

这里“foo”这个名字究竟保存在哪里呢？在foo的活跃对象中吗？非也，因为在foo函数中根本就没有定义任何“foo”。
那么是在上层上下文的变量对象中吗？也不是，因为根据定义——FE是不会影响VO的——正如我们在外层对其调用的结果所看到的那样。
那么，它究竟保存在哪里了呢？


