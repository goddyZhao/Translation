说明
--------
* * *
此文译自[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/) 的文章[Variable object](http://dmitrysoshnikov.com/ecmascript/chapter-2-variable-object/)  
另，此文还有另外一位同事（宋珍珍）共同参译

概要
--------
* * *
我们总是会在程序中定义一些函数和变量，之后会使用这些函数和变量来构建我们的系统。  
然而，对于解释器来说，它又是如何以及从哪里找到这些数据的（函数，变量）？当引用一个对象的时候，在解释器内部又发生了什么？  

许多ECMA脚本程序员都知道，变量和[执行上下文](http://goddyzhao.tumblr.com/post/10020230352/execution-context)是密切相关的：
<pre><code>var a = 10; // 全局上下文中的变量
 
(function () {
  var b = 20; // 函数上下文中的本地变量
})();
 
alert(a); // 10
alert(b); // "b" is not defined</code></pre>

不仅如此，许多程序员也都知道，ECMAScript标准中指出独立的作用域只有通过“函数代码”（可执行代码类型中的一种）才能创建出来。比方说，与C/C++不同的是，在ECMAScript中_for_循环的代码块是无法创建本地上下文的：
<pre><code>for (var k in {a: 1, b: 2}) {
  alert(k);
}
 
alert(k); // 尽管循环已经结束，但是变量“k”仍然在作用域中</code></pre>

下面就来详细介绍下，当申明变量和函数的时候，究竟发生了什么。  


数据申明
--------
* * *
既然变量和执行上下文有关，那它就该知道数据存储在哪里以及如何获取。这种机制就称作_变量对象_：  
>  A variable object (in abbreviated form — VO) is a special object related with an execution context and which stores:  
>
>  *  variables (var, VariableDeclaration);  
>  *  function declarations (FunctionDeclaration, in abbreviated form FD);  
>  *  and function formal parameters  
>  declared in the context.  

举个例子，可以用ECMAScript的对象来表示变量对象：
<pre><code>VO = {};</code></pre>

VO同时也是一个执行上下文的属性：
<pre><code>activeExecutionContext = {
  VO: {
    // 上下文中的数据 (变量申明（var）, 函数申明（FD), 函数形参（function arguments）)
  }
};</code></pre>


对变量的间接引用（通过VO的属性名）只允许发生在_全局上下文_中的变量对象上(全局对象本身就是变量对象，这部分会在后续作相应的介绍)。
对于其他的上下文而言，是无法直接引用VO的，因为VO是实现层的。

申明新的变量和函数的过程其实就是在VO中创建新的和变量以及函数名对应的属性和属性值的过程。

如下所示：
<pre><code>var a = 10;
 
function test(x) {
  var b = 20;
};
 
test(30);</code></pre>


上述代码对应的变量对象则如下所示：
<pre><code>// 全局上下文中的变量对象
VO(globalContext) = {
  a: 10,
  test: <reference to function>
};
 
// “test”函数上下文中的变量对象
VO(test functionContext) = {
  x: 30,
  b: 20
};</code></pre>


但是，在实现层（标准中定义的），变量对象只是一个抽象的概念。在实际执行上下文中，VO可能完全不叫VO，并且初始的结构也可能完全不同。


不同执行上下文中的变量对象
--------
* * *
变量对象上的一些操作（比如：变量的初始化）和行为对于所有的执行上下文类型来说都已一样的。从这一点来说，将变量对象表示成抽象的概念更加合适。
函数上下文还能定义额外的与变量对象相关的信息。
<pre><code>AbstractVO (generic behavior of the variable instantiation process)
 
  ║
  ╠══> GlobalContextVO
  ║        (VO === this === global)
  ║
  ╚══> FunctionContextVO
           (VO === AO, <arguments> object and <formal parameters> are added)</code></pre>
           
接下来对这块内容进行详细介绍。


全局上下文中的变量对象
--------
* * *
首先，有必要对_全局对象（Global object）_作个定义。
>  全局对象是一个在进入任何执行上下文前就创建出来的对象；此对象以单例形式存在；它的属性在程序任何地方都可以直接访问，其生命周期随着程序的结束而终止。


