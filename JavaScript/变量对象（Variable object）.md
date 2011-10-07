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

全局对象在创建的时候，诸如Math,String,Date,parseInt等等属性也会被初始化，同时，其中一些对象会指向全局对象本身——比如，DOM中，全局对象上的_window_属性就指向了全局对象（但是，并非所有的实现都是如此）：
<pre><code>global = {
  Math: <...>,
  String: <...>
  ...
  ...
  window: global
};</code></pre>

在引用全局对象的属性时，前缀通常可以省略，因为全局对象是不能通过名字直接访问的。然而，通过全局对象上的_this_值，以及通过如DOM中的window对象这样递归引用的方式都可以访问到全局对象：
<pre><code>String(10); // 等同于 global.String(10);
 
// 带前缀
window.a = 10; // === global.window.a = 10 === global.a = 10;
this.b = 20; // global.b = 20;</code></pre>

回到全局上下文的变量对象上——这里变量对象就是全局对象本身：
<pre><code>VO(globalContext) === global;</code></pre>

准确地理解这个事实是非常必要的：正是由于这个原因，当在全局上下文中申明一个变量时，可以通过全局对象上的属性来间地引用该变量（比方说，当变量名提前未知的情况下）
<pre><code>var a = new String('test');
 
alert(a); // directly, is found in VO(globalContext): "test"
 
alert(window['a']); // indirectly via global === VO(globalContext): "test"
alert(a === this.a); // true
 
var aKey = 'a';
alert(window[aKey]); // indirectly, with dynamic property name: "test"</code></pre>


函数上下文中的变量对象
--------
* * *
在函数的执行上下文中，VO是不能直接访问的。它主要扮演被称作_活跃对象（activation object）_（简称：AO）的角色。
<pre><code>VO(functionContext) === AO;</code></pre>

>  活跃对象会在进入函数上下文的时候创建出来，初始化的时候会创建一个_arguments_属性，其值就是_Arguments_对象：  

<pre><code>AO = {
  arguments: <ArgO>
};</code></pre>

_Arguments_对象是活跃对象上的属性，它包含了如下属性：  

*  _callee_ —— 对当前函数的引用  
*  _length_ —— 实参的个数  
*  _properties-indexes(数字，转换成字符串)_其值是函数参数的值（参数列表中，从左到右）。properties-indexes的个数 == arguments.length;  
arguments对象的properties-indexes的值和当前（实际传递的）形参是共享的。

如下所示：<br/>
<pre><code>function foo(x, y, z) {
 
  // 定义的函数参数（x,y,z）的个数
  alert(foo.length); // 3
 
  // 实际传递的参数个数
  alert(arguments.length); // 2
 
  // 引用函数自身
  alert(arguments.callee === foo); // true
 
  // 参数互相共享
 
  alert(x === arguments[0]); // true
  alert(x); // 10
 
  arguments[0] = 20;
  alert(x); // 20
 
  x = 30;
  alert(arguments[0]); // 30
 
  // 然而，对于没有传递的参数z，
  // 相关的arguments对象的index-property是不共享的
 
  z = 40;
  alert(arguments[2]); // undefined
 
  arguments[2] = 50;
  alert(z); // 40
 
}
 
foo(10, 20);</code></pre>

上述例子，在当前的Google Chrome浏览器中有个bug——参数z和arguments[2]也是互相共享的。


处理上下文代码的几个阶段
--------
* * *
至此，也就到了本文最核心的部分了。处理执行上下文代码分为两个阶段：  

1.  进入执行上下文  
2.  执行代码  

对变量对象的修改和这两个阶段密切相关。  

要注意的是，这两个处理阶段是通用的行为，与上下文类型无关（不管是全局上下文还是函数上下文都是一致的）。  


进入执行上下文
--------
* * *
一旦进入执行上下文（在执行代码之前），VO就会被一些属性填充（在此前已经描述过了）：

*  函数的形参（当进入函数执行上下文时）  
   —— 变量对象的一个属性，其属性名就是形参的名字，其值就是实参的值；对于没有传递的参数，其值为_undefined_  
*  函数申明（_FunctionDeclaration_, FD）
   —— 变量对象的一个属性，其属性名和值都是函数对象创建出来的；如果变量对象已经包含了相同名字的属性，则替换它的值
*  变量申明（_var_，_VariableDeclaration_）
   —— 变量对象的一个属性，其属性名即为变量名，其值为_undefined_
