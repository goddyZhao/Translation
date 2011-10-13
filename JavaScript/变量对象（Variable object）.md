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
  var b = 20; // 函数上下文中的局部变量
})();
 
alert(a); // 10
alert(b); // "b" is not defined</code></pre>

不仅如此，许多程序员也都知道，ECMAScript标准中指出独立的作用域只有通过“函数代码”（可执行代码类型中的一种）才能创建出来。比方说，与C/C++不同的是，在ECMAScript中_for_循环的代码块是无法创建本地上下文的：
<pre><code>for (var k in {a: 1, b: 2}) {
  alert(k);
}
 
alert(k); // 尽管循环已经结束，但是变量“k”仍然在作用域中</code></pre>

下面就来详细介绍下，当声明变量和函数的时候，究竟发生了什么。  


数据声明
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
    // 上下文中的数据 (变量声明（var）, 函数声明（FD), 函数形参（function arguments）)
  }
};</code></pre>


对变量的间接引用（通过VO的属性名）只允许发生在_全局上下文_中的变量对象上(全局对象本身就是变量对象，这部分会在后续作相应的介绍)。
对于其他的上下文而言，是无法直接引用VO的，因为VO是实现层的。

声明新的变量和函数的过程其实就是在VO中创建新的和变量以及函数名对应的属性和属性值的过程。

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

准确地理解这个事实是非常必要的：正是由于这个原因，当在全局上下文中声明一个变量时，可以通过全局对象上的属性来间地引用该变量（比方说，当变量名提前未知的情况下）
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
*  函数声明（_FunctionDeclaration_, FD）
   —— 变量对象的一个属性，其属性名和值都是函数对象创建出来的；如果变量对象已经包含了相同名字的属性，则替换它的值
*  变量声明（_var_，_VariableDeclaration_）
   —— 变量对象的一个属性，其属性名即为变量名，其值为_undefined_;如果变量名和已经声明的函数名或者函数的参数名相同，则不会影响已经存在的属性。  

看下面这个例子：  
<pre><code>function test(a, b) {
  var c = 10;
  function d() {}
  var e = function _e() {};
  (function x() {});
}
 
test(10); // call</code></pre>

当以10为参数进入“test”函数上下文的时候，对应的AO如下所示：  
<pre><code>AO(test) = {
  a: 10,
  b: undefined,
  c: undefined,
  d: &lt;reference to FunctionDeclaration "d"&gt;
  e: undefined
};</code></pre>

注意了，上面的AO并不包含函数“x”。这是因为这里的“x”并不是函数声明而是_函数表达式_（_FunctionExpression_，简称FE），函数表达式不会对VO造成影响。
尽管函数“_e”也是函数表达式，然而，正如我们所看到的，由于它被赋值给了变量“e”，因此它可以通过“e”来访问到。关于函数声明和函数表达式的区别会在第五章——函数作具体介绍。  

至此，处理上下文代码的第一阶段介绍完了，接下来介绍第二阶段——_执行代码_阶段。


执行代码
--------
* * *
此时，AO/VO的属性已经填充好了。（尽管，大部分属性都还没有赋予真正的值，都只是初始化时候的_undefined_值）。  

继续以上一例子为例，到了执行代码阶段，AO/VO就会修改成为如下形式：  
<pre><code>AO['c'] = 10;
AO['e'] = <reference to FunctionExpression "_e">;</code></pre>

再次注意到，这里函数表达式“_e”仍在内存中，这是因为它被保存在声明的变量“e”中，而同样是函数表达式的“x”却不在AO/VO中：
如果尝试在定义前或者定义后调用“x”函数，这时会发生“x为定义”的错误。未保存的函数表达式只有在定义或者递归时才能调用。  

如下是更加典型的例子：  
<pre><code>alert(x); // function
 
var x = 10;
alert(x); // 10
 
x = 20;
 
function x() {};
 
alert(x); // 20</code></pre>

上述例子中，为何“x”打印出来是函数呢？为何在声明前就可以访问到？又为何不是10或者20呢？原因在于，根据规则——在进入上下文的时候，VO会被填充函数声明；
同一阶段，还有变量声明“x”，但是，正如此前提到的，变量声明是在函数声明和函数形参之后，并且，变量声明不会对已经存在的同样名字的函数声明和函数形参发生冲突，
因此，在进入上下文的阶段，VO填充为如下形式：  
<pre><code>VO = {};
 
VO['x'] = &lt;引用了函数声明“x”&gt;
 
// 发现var x = 10;
// 如果函数“x”还未定义
// 则 "x" 为undefined, 但是，在我们的例子中
// 变量声明并不会影响同名的函数值
 
VO['x'] = &lt;值不受影响，仍是函数&gt;</code></pre>

随后，在执行代码阶段，VO被修改为如下所示：  
<pre><code>VO['x'] = 10;
VO['x'] = 20;</code></pre>

正如在第二个和第三个alert显示的那样。  

如下例子再次看到在进入上下文阶段，变量存储在VO中（因此，尽管else的代码块永远都不会执行到，而“b”却仍然在VO中）：  
<pre><code>if (true) {
  var a = 1;
} else {
  var b = 2;
}
 
alert(a); // 1
alert(b); // undefined, but not "b is not defined"</code></pre>

关于变量
--------
* * *
大多数讲JavaScript的文章甚至是JavaScript的书通常都会这么说：“声明全局变量的方式有两种，一种是使用_var_关键字（在全局上下文中），另外一种是不用_var_关键字（在任何位置）”。
而这样的描述是错误的。要记住的是：  
_使用var关键字是声明变量的唯一方式_  

如下赋值语句：  
<pre><code>a = 10;</code></pre>

仅仅是在全局对象上创建了新的属性（而不是变量）。“不是变量”并不意味着它无法改变，它是ECMAScript中变量的概念（它之后可以变为全局对象的属性，因为VO(globalContext) === global，还记得吧？）  

不同点如下所示：  
<pre><code>alert(a); // undefined
alert(b); // "b" is not defined
 
b = 10;
var a = 20;</code></pre>

接下来还是要谈到VO和在不同阶段对VO的修改（进入上下文阶段和执行代码阶段）：  
进入上下文：  
<pre><code>VO = {
  a: undefined
};</code></pre>

我们看到，这个阶段并没有任何“b”，因为它不是变量，“b”在执行代码阶段才出现。（但是，在我们这个例子中也不会出现，因为在“b”出现前就发生了错误）  

将上述代码稍作改动：  
<pre><code>alert(a); // undefined, we know why
 
b = 10;
alert(b); // 10, created at code execution
 
var a = 20;
alert(a); // 20, modified at code execution</code></pre>

这里关于变量还有非常重要的一点：与简单属性不同的是，变量是不能删除的{DontDelete},这意味着要想通过**delete**操作符来删除一个变量是不可能的。  

<pre><code>a = 10;
alert(window.a); // 10
 
alert(delete a); // true
 
alert(window.a); // undefined
 
var b = 20;
alert(window.b); // 20
 
alert(delete b); // false
 
alert(window.b); // still 20</code></pre>

但是，这里有个例外，就是“eval”执行上下文中，是可以删除变量的：
<pre><code>eval('var a = 10;');
alert(window.a); // 10
 
alert(delete a); // true
 
alert(window.a); // undefined</code></pre>

利用某些debug工具，在终端测试过这些例子的童鞋要注意了：其中Firebug也是使用了eval来执行终端的代码。因此，这个时候var也是可以删除的。  


实现层的特性：\_\_parent\_\_属性
--------
* * *
正如此前介绍的，标准情况下，是无法直接访问活跃对象的。然而，在某些实现中，比如知名的SpiderMonkey和Rhino,函数有个特殊的属性\_\_parent\_\_，
该属性是对该函数创建所在的活跃对象的引用（或者全局变量对象）。  

如下所示（SpiderMonkey,Rhino）：  
<pre><code>var global = this;
var a = 10;
 
function foo() {}
 
alert(foo.__parent__); // global
 
var VO = foo.__parent__;
 
alert(VO.a); // 10
alert(VO === global); // true</code></pre>

上述例子中，可以看到函数_foo_是在全局上下文中创建的，相应的，它的\_\_parent\_\_属性设置为全局上下文的变量对象，比如说：全局对象。  

然而，在SpiderMonkey中以相同的方式获取活跃对象是不可能的：不同的版本表现都不同，内部函数的\_\_parent\_\_属性会返回null或者全局对象。  
在Rhino中，以相同的方式获取活跃对象是允许的：  

如下所示（Rhino）：  
<pre><code>var global = this;
var x = 10;
 
(function foo() {
 
  var y = 20;
 
  // the activation object of the "foo" context
  var AO = (function () {}).__parent__;
 
  print(AO.y); // 20
 
  // __parent__ of the current activation
  // object is already the global object,
  // i.e. the special chain of variable objects is formed,
  // so-called, a scope chain
  print(AO.__parent__ === global); // true
 
  print(AO.__parent__.x); // 10
 
})();</code></pre>


总结
--------
* * *
本文，我们介绍了与执行上下文相关的对象。希望，本文能够对大家有所帮助，同时也希望本文能够起到解惑的作用。


扩展阅读
--------
* * *

*  10.1.3 —— [变量初始化](http://bclary.com/2004/11/07/#a-10.1.3)  
*  10.1.5 —— [全局对象](http://bclary.com/2004/11/07/#a-10.1.5)  
*  10.1.6 —— [活跃对象](http://bclary.com/2004/11/07/#a-10.1.6)  
*  10.1.8 —— [参数对象](http://bclary.com/2004/11/07/#a-10.1.8)  

