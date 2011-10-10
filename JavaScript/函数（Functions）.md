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

不卖关子了，马上来揭晓。当解释器在执行代码阶段看到了有名字的FE之后，它会在创建FE之前，创建一个辅助型的特殊对象，并把它添加到当前的作用域链中。
然后，再创建FE，在这个时候（根据第四章-作用域链描述的），函数拥有了\[\[Scope\]\]属性 —— 创建函数所在上下文的作用域链（这个时候，在\[\[Scope\]\]就有了那个特殊对象）。
之后，特殊对象中唯一的属性 —— FE的名字添加到了该对象中；其值就是对FE的引用。在最后，当前上下文退出的时候，就会把该特殊对象移除。
用伪代码来描述此算法就如下所示：  
<pre><code>specialObject = {};
 
Scope = specialObject + Scope;
 
foo = FunctionExpression;
foo.[[Scope]] = Scope;
specialObject.foo = foo; // {DontDelete}, {ReadOnly}
 
delete Scope[0]; // 从作用域链的最前面移除specialObject</code></pre>

这就是为什么在函数外是无法通过名字访问到该函数的（因为它并不在上层作用域中存在），而在函数内部却可以访问到。  

而这里要注意的一点是： 在某些实现中，比如Rhino，FE的名字并不是保存在特殊对象中的，而是保存在FE的活跃对象中。
再比如微软的实现 —— JScript，则完全破坏了FE的规则，直接将该名字保存在上层作用域的变量对象中了，这样在外部也可以访问到。  


NFE和SpiderMonkey
--------
* * *
说到实现，部分版本的SpiderMonkey有一个与上述提到的特殊对象相关的特性，这个特性也可以看作是个bug（既然所有的实现都是严格遵循标准的，那么这个就是标准的问题了）。
此特性和标识符处理相关： 作用域链的分析是二维的，在标识符查询的时候，还要考虑作用域链中每个对象的原型链。  

当在_Object.prototype_对象上定义一个属性，并将该属性值指向一个“根本不存在”的变量时，就能够体现该特性。
比如，如下例子中的变量“x”，在查询过程中，通过作用域链，一直到全局对象也是找不到“x”的。
然而，在SpiderMonkey中，全局对象继承自_Object.prototype_，于是，对应的值就在该对象中找到了：  
<pre><code>Object.prototype.x = 10;
 
(function () {
  alert(x); // 10
})();</code></pre>

活跃对象是没有原型一说的。可以通过内部函数还证明。
如果在定义一个本地变量“x”并声明一个内部函数（FD或者匿名的FE），然后，在内部函数中引用变量“x”，这个时候该变量会在上层函数上下文中查询到（理应如此），而不是在_Object.prototype_中：  
<pre><code>Object.prototype.x = 10;
 
function foo() {
 
  var x = 20;
 
  // function declaration 
 
  function bar() {
    alert(x);
  }
 
  bar(); // 20, from AO(foo)
 
  // the same with anonymous FE
 
  (function () {
    alert(x); // 20, also from AO(foo)
  })();
 
}
 
foo();</code></pre>

在有些实现中，存在这样的异常：它们会在活跃对象设置原型。比方说，在_Blackberry_的实现中，上述例子中变量“x”值就会变成10。
因为，“x”从_Object.prototype_中就找到了：  
<pre><code>AO(bar FD or anonymous FE) -> no ->
AO(bar FD or anonymous FE).[[Prototype]] -> yes - 10</code></pre>

当出现有名字的FE的特殊对象的时候，在SpiderMonkey中也是有同样的异常。该特殊对象是常见对象 —— “和通过new Object()表达式产生的一样”。
相应地，它也应当继承自_Object.prototype_，上述描述只针对SpiderMonkey（1.7版本）。其他的实现（包括新的TraceMonkey）是不会给这个特殊对象设置原型的：  
<pre><code>function foo() {
 
  var x = 10;
 
  (function bar() {
 
    alert(x); // 20, but not 10, as don't reach AO(foo)
 
    // "x" is resolved by the chain:
    // AO(bar) - no -> __specialObject(bar) -> no
    // __specialObject(bar).[[Prototype]] - yes: 20
 
  })();
}
 
Object.prototype.x = 20;
 
foo();</code></pre>


NFE和JScript
--------
* * *
微软的实现——JScript，是IE的JS引擎（截至本文撰写时最新是JScript5.8——IE8），该引擎与NFE相关的bug有很多。每个bug基本上都和ECMA-262-3rd标准是完全违背的。
有些甚至会引发严重的错误。  

第一，针对上述这样的情况，JScript完全破坏了FE的规则：不应当将函数名字保存在变量对象中的。
另外，FE的名字应当保存在特殊对象中，并且只有在函数自身内部才可以访问（其他地方均不可以）。而JScript却将其直接保存在上层上下文的变量对象中。
并且，JScript居然还将FE以FD的方式处理，在进入上下文的时候就将其创建出来，并在定义之前就可以访问到：  
<pre><code>// FE 保存在变量对象中
// 和FD一样，在定义前就可以通过名字访问到
testNFE();
 
(function testNFE() {
  alert('testNFE');
});
 
// 同样的，在定义之后也可以通过名字访问到
testNFE();</code></pre>

正如大家所见，完全破坏了FE的规则。  

第二，在声明同时，将NFE赋值给一个变量的时候，JScript会创建两个不同的函数对象。
这种行为感觉完全不符合逻辑（特别是考虑到在NFE外层，其名字根本是无法访问到的）：  
<pre><code>var foo = function bar() {
  alert('foo');
};
 
alert(typeof bar); // "function", NFE 有在VO中了 – 这里就错了
 
// 然后，还有更有趣的
alert(foo === bar); // false!
 
foo.x = 10;
alert(bar.x); // undefined
 
// 然而，两个函数完全做的是同样的事情
 
foo(); // "foo"
bar(); // "foo"</code></pre>

然而，要注意的是： 当将NFE和赋值给变量这两件事情分开的话（比如，通过组操作符），在定义好后，再进行变量赋值，这样，两个对象就相同了，返回true：  
<pre><code>(function bar() {});
 
var foo = bar;
 
alert(foo === bar); // true
 
foo.x = 10;
alert(bar.x); // 10</code></pre>

这个时候就好解释了。实施上，一开始的确创建了两个对象，不过之后就只剩下一个了。这里将NFE以FD的方式来处理，然后，当进入上下文的时候，FD bar就创建出来了。
在这之后，到了执行代码阶段，又创建出了第二个对象 —— FE bar，该对象不会进行保存。相应的，由于没有变量对其进行引用，随后FE bar对象就被移除了。
因此，这里就只剩下一个对象——FD bar对象，对该对象的引用就赋值给了foo变量。  

第三，通过_arguments.callee_对一个函数进行间接引用，它引用的是和激活函数名一致的对象（事实上是——函数，因为有两个对象）：  
<pre><code>var foo = function bar() {
 
  alert([
    arguments.callee === foo,
    arguments.callee === bar
  ]);
 
};
 
foo(); // [true, false]
bar(); // [false, true]</code></pre>

第四，JScript会将NFE以FD来处理，但当遇到条件语句又不遵循此规则了。比如说，和FD那样，NFE会在进入上下文的时候就创建出来，这样最后一次定义的就会被使用：  
<pre><code>var foo = function bar() {
  alert(1);
};
 
if (false) {
 
  foo = function bar() {
    alert(2);
  };
 
}
bar(); // 2
foo(); // 1</code></pre>

上述行为从逻辑上也是可以解释通的： 当进入上下文的时候，最后一次定义的FD bar被创建出来（有alert(2)的函数），
之后到了执行代码阶段又一个新的函数 —— FE bar被创建出来，对其引用赋值给了变量foo。因此（if代码块中由于判断条件是false，因此其代码块中的代码永远不会被执行到）foo函数的调用会打印出1。
尽管“逻辑上”是对的，但是这个仍然算是IE的bug。因为它明显就破坏了实现的规则，所以我这里用了引号“逻辑上”。  

第五个JScript中NFE的bug和通过给一个未受限的标识符赋值（也就是说，没有var关键字）来创建全局对象的属性相关。
由于这里NFE会以FD的方式来处理，并相应地会保存在变量对象上，赋值给未受限的标识符（不是给变量而是给全局对象的一般属性），
当函数名和标识符名字相同的时候，该属性就不会是全局的了。  
<pre><code>(function () {
 
  // 没有var，就不是本地变量，而是全局对象的属性
 
  foo = function foo() {};
 
})();
 
// 然而，在匿名函数的外层，foo又是不可访问的
 
alert(typeof foo); // undefined</code></pre>

这里从“逻辑上”又是可以解释通的： 进入上下文时，函数声明在匿名函数本地上下文的活跃对象中。
当进入执行代码阶段的时候，因为foo这个名字已经在AO中存在了（本地），相应地，赋值操作也只是简单的对AO中的foo进行更新而已。
并没有在全局对象上创建新的属性。  


通过Function构造器创建的函数
--------
* * *
这类函数有别于FD和FE，有自己的专属特性： 它们的\[\[Scope\]\]属性中只包含全局对象：  
<pre><code>var x = 10;
 
function foo() {
 
  var x = 20;
  var y = 30;
 
  var bar = new Function('alert(x); alert(y);');
 
  bar(); // 10, "y" is not defined
 
}</code></pre>

我们看到_bar_函数的\[\[Scope\]\]属性并未包含foo上下文的AO —— 变量“y”是无法访问的，并且变量“x”是来自全局上下文。
顺便提下，这里要注意的是，Function构造器可以通过_new_关键字和省略_new_关键字两种用法。上述例子中，这两种用法都是一样的。  

此类函数其他特性则和[同类语法产生式](http://bclary.com/2004/11/07/#a-13.1.1)以及[联合对象](http://bclary.com/2004/11/07/#a-13.1.2)有关。
该机制在标准中建议在作优化的时候采用（当然，具体的实现者也完全有权利不使用这类优化）。比方说，有100元素的数组，在循环数组过程中会给数组每个元素赋值（函数），
这个时候，实现的时候就可以采用联合对象的机制了。这样，最终所有的数组元素都会引用同一个函数（只有一个函数）：  
<pre><code>var a = [];
 
for (var k = 0; k < 100; k++) {
  a[k] = function () {}; // 这里就可以使用联合对象
}</code></pre>

但是，通过Function构造器创建的函数就无法使用联合对象了：  
<pre><code>var a = [];
 
for (var k = 0; k < 100; k++) {
  a[k] = Function(''); // 只能是100个不同的函数
}</code></pre>

下面是另外一个和联合对象相关的例子：  
<pre><code>function foo() {
 
  function bar(z) {
    return z * z;
  }
 
  return bar;
}
 
var x = foo();
var y = foo();</code></pre>

上述例子，在实现过程中同样可以使用联合对象。来使得x和y引用同一个对象，因为函数（包括它们内部的\[\[Scope\]\]属性）物理上是不可分辨的。
因此，通过Function构造器创建的函数总是会占用更多内存资源。  


函数创建的算法
--------
* * *
如下所示使用伪代码表示的函数创建的算法（不包含联合对象的步骤）。有助于理解ECMAScript中的函数对象。此算法对所有函数类型都是一样的。
<pre><code>F = new NativeObject();
 
// 属性 [[Class]] is "Function"
F.[[Class]] = "Function"
 
// 函数对象的原型
F.[[Prototype]] = Function.prototype
 
// 对函数自身引用
// [[Call]] 在函数调用时F()激活
// 同时创建一个新的执行上下文
F.[[Call]] = <reference to function>
 
// 内置的构造器
// [[Construct]] 会在使用“new”关键字的时候激活
// 事实上，它会为新对象申请内存
// 然后调用 F.[[Call]]来初始化创建的对象，将this值设置为新创建的对象
F.[[Construct]] = internalConstructor
 
// 当前上下文（创建函数F的上下文）的作用域名链
F.[[Scope]] = activeContext.Scope
// 如果是通过new Function(...)来创建的，则
F.[[Scope]] = globalContext.Scope
 
// 形参的个数
F.length = countParameters
 
// 通过F创建出来的对象的原型
__objectPrototype = new Object();
__objectPrototype.constructor = F // {DontEnum}, 在遍历中不能枚举
F.prototype = __objectPrototype
 
return F</code></pre>

要注意的是，F.\[\[Prototype\]\]是函数（构造器）的原型，而F.prototype是通过该函数创建出来的对象的原型（因为通常对这两个概念都会混淆，在有些文章中会将_F.prototype_叫做“构造器的原型”，这是错误的）。  


总结
--------
* * *
本文介绍了很多关于函数的内容；不过在后面的关于对象和原型的文章中，还会提到函数作为构造器是如何工作的。  


扩展阅读
--------
* * *
ECMAScript标准：  

*  13 —— [函数定义](http://bclary.com/2004/11/07/#a-13)
*  15.3 —— [函数对象](http://bclary.com/2004/11/07/#a-15.3)

另外一篇文章：  

*  [揭秘有名字的函数表达式（来自Juriy “kangax” Zaytsev）](http://yura.thinkweb2.com/named-function-expressions/)

