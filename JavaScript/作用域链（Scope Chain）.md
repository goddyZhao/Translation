说明
--------
* * *
此文译自[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/) 的文章[Scope Chain](http://dmitrysoshnikov.com/ecmascript/chapter-4-scope-chain/)    
另，此文还有另外一位同事（邵信衡）共同参译


概要
--------
* * *
在[第二章](http://goddyzhao.tumblr.com/post/11141710441/variable-object)变量对象的时候，
已经介绍过[执行上下文](http://goddyzhao.tumblr.com/post/10020230352/execution-context)的数据是以变量对象的属性的形式进行存储的。  

还介绍了，每次进入执行上下文的时候，就会创建变量对象，并且赋予其属性初始值，随后在执行代码阶段会对属性值进行更新。  

本文要与执行上下文密切相关的另外一个重要的概念——_作用域链（Scope Chain）_。  


定义
--------
* * *
若要简单扼要对作用域脸做个解释，那就是：作用域链和内部函数息息相关。  

众所周知，ECMAScript允许创建内部函数，甚至可以将这些内部函数作为父函数的返回值。  
<pre><code>var x = 10;
 
function foo() {
 
  var y = 20;
 
  function bar() {
    alert(x + y);
  }
 
  return bar;
 
}
 
foo()(); // 30</code></pre>

每个上下文都有自己的变量对象：对于全局上下文而言，其变量对象就是_全局对象_本身，对于函数而言，其变量对象就是_活跃对象_。  

作用域链其实就是所有内部上下文的变量对象的列表。用于变量查询。比如，在上述例子中，“bar”上下文的作用域链包含了AO(bar),AO(foo)和VO(global)。  

下面就来详细介绍下作用域链。  

先从定义开始，随后再结合例子详细介绍：
>  作用域链是一条变量对象的链，它和执行上下文有关，用于在处理标识符时候进行变量查询。  

函数上下文的作用域链在函数调用的时候创建出来，它包含了活跃对象和该函数的内部\[\[Scope\]\]属性。关于\[\[Scope\]\]会在后面作详细介绍。  

大致表示如下：  
<pre><code>activeExecutionContext = {
    VO: {...}, // 或者 AO
    this: thisValue,
    Scope: [ // 所用域链
      // 所有变量对象的列表
      // 用于标识符查询
    ]
};</code></pre>

上述代码中的_Scope_定义为如下所示：  
<pre><code>Scope = AO + [[Scope]]</code></pre>

针对我们的例子来说，可以将Scope和\[\[Scope\]\]用普通的ECMAScript数组来表示：  
<pre><code>var Scope = [VO1, VO2, ..., VOn]; // 作用域链</code></pre>

除此之外，还可以用分层对象链的数据结构来表示，链中每一个链接都有对父作用域（上层变量对象）的引用。这种表示方式和第二章中讨论的某些实现中\_\_parent\_\_的概念相对应：  
<pre><code>var VO1 = {__parent__: null, ... other data}; --&gt;
var VO2 = {__parent__: VO1, ... other data}; --&gt;
// etc.</code></pre>  

然而，使用数组来表示作用域链会更方便，因此，我们这里就采用数组的表示方式。
除此之外，不论在实现层是否采用包含\_\_parent\_\_特性的分层对象链的数据结构，标准自身对其做了抽象的定义“作用域链是一个对象_列表_”。
数组就是实现列表这一概念最好的选择。  

下面将要介绍的 AO+\[\[Scope\]\]以及标识符的处理方式，都和函数的生命周期有关。  


函数的生命周期
--------
* * *
函数的生命周期分为_创建_阶段和_激活（调用）_阶段。下面就来详细对其作介绍。  


函数的创建
--------
* * *
众所周知，在进入上下文阶段，函数申明会存储在变量/活跃对象中（VO/AO）。让我们来看一个全局上下文中变量申明和函数申明的例子（这种情况下，变量对象就是全局对象本身，应该还没忘记吧？）：  
<pre><code>var x = 10;
 
function foo() {
  var y = 20;
  alert(x + y);
}
 
foo(); // 30</code></pre>

在函数激活后，我们看到了正确（预期）的结果——30。不过，这里有一个非常重要的特性。  

在说当前上下文的变量对象前。上述代码中我们看到变量“y”是在“foo”函数中定义的（意味着它存储在“foo”上下文的AO对象中），
然而变量“x”则并没有在“foo”上下文中定义，自然也不会添加到“foo”的AO中。乍一眼看过去，变量“x”压根就不在“foo”中存在；
然而，正如我们下面要看到的——仅仅只是“乍一眼看过去“而已。我们看到“foo”上下文的活跃对象中只包含一个属性——“y”：  
<pre><code>fooContext.AO = {
  y: undefined // undefined – 在进入上下文时, 20 – 在激活阶段
};</code></pre>

那么，“foo”函数到底是如何访问到变量“x”的呢？一个顺其自然的想法是：函数应当有访问更高层上下文变量对象的权限。
而事实也恰是如此，就是通过函数的内部属性\[\[Scope\]\]来实现这一机制的。  
>  \[\[Scope\]\]是一个包含了所有上层变量对象的分层链，它属于当前函数上下文，并在函数创建的时候，保存在函数中。  

这里要注意的很重要的一点是：\[\[Scope\]\]是在函数创建的时候保存起来的——静态的（不变的），只有一次并且一直都存在——直到函数销毁。
比方说，哪怕函数永远都不能被调用到，\[\[Scope\]\]属性也已经保存在函数对象上了。  

另外要注意的一点是：\[\[Scope\]\]与Scope(作用域链)是不同的，前者是函数的属性，后者是上下文的属性。
以上述例子来说，“foo”函数的\[\[Scope\]\]如下所示：  
<pre><code>foo.[[Scope]] = [
  globalContext.VO // === Global
];</code></pre>

之后，有了函数调用，就会进入函数上下文，这个时候会创建活跃对象并且this的值和Scope（作用域链）都会确定。下面来详细介绍下。  


函数的激活
--------
* * *
正如在“定义”这节提到的，在进入上下文，AO/VO创建之后，上下文的_Scope_属性（作用域链，用于变量查询）会定义为如下所示：  
<pre><code>Scope = AO|VO + [[Scope]]</code></pre>

这里要注意的是活跃对象是_Scope_数组的第一个元素。添加在作用域链的最前面：  
<pre><code>Scope = [AO].concat([[Scope]]);</code></pre>

此特性对处理标识符非常重要。  
>  处理标识符其实就是一个确定变量（或者函数申明）属于作用域链中哪个变量对象的过程。  

此算法返回的总是一个引用类型的值，其base属性就是对应的变量对象（或者如果变量不存在的时候则返回null），其property name属性的名字就是要查询的标识符。
要详细了解引用类型可以参看[第三章-this](http://goddyzhao.tumblr.com/post/11218727474/this)。  

标识符处理过程包括了对应的变量名的属性查询，比如：在作用域链中会进行一系列的变量对象的检测，从作用域链的最底层上下文一直到最上层上下文。  

因此，在查询过程中上下文中的本地变量相比较上层上下文的变量会优先被查询到，换句话说，如果两个相同名字的变量存在于不同的上下文中时，处于底层上下文的变量会优先被找到。  

下面是一个相对比较复杂的例子：  
<pre><code>var x = 10;
 
function foo() {
 
  var y = 20;
 
  function bar() {
    var z = 30;
    alert(x +  y + z);
  }
 
  bar();
}
 
foo(); // 60</code></pre>

针对上述代码，对应了如下的变量/活跃对象，函数的\[\[Scope\]\]属性以及上下文的作用域链：  

全局上下文的变量对象如下所示：  
<pre><code>globalContext.VO === Global = {
  x: 10
  foo: <reference to function>
};</code></pre>

在“foo”函数创建的时候，其\[\[Scope\]\]属性如下所示：  
<pre><code>foo.[[Scope]] = [
  globalContext.VO
];</code></pre>

在“foo”函数激活的时候（进入上下文时），“foo”函数上下文的活跃对象如下所示：  
<pre><code>fooContext.AO = {
  y: 20,
  bar: <reference to function>
};</code></pre>

同时，“foo”函数上下文的作用域链如下所示：  
<pre><code>fooContext.Scope = fooContext.AO + foo.[[Scope]] // i.e.:
 
fooContext.Scope = [
  fooContext.AO,
  globalContext.VO
];</code></pre>

在内部“bar”函数创建的时候，其\[\[Scope\]\]属性如下所示：  
<pre><code>bar.[[Scope]] = [
  fooContext.AO,
  globalContext.VO
];</code></pre>

在“bar”函数激活的时候，其对应的活跃对象如下所示：  
<pre><code>barContext.AO = {
  z: 30
};</code></pre>

同时，“bar”函数上下文的作用域链如下所示：  
<pre><code>barContext.Scope = barContext.AO + bar.[[Scope]] // i.e.:
 
barContext.Scope = [
  barContext.AO,
  fooContext.AO,
  globalContext.VO
];</code></pre>

如下是“x”，“y”和“z”标识符的查询过程：  
<pre><code>- "x"
-- barContext.AO // not found
-- fooContext.AO // not found
-- globalContext.VO // found - 10</code></pre>

<pre><code>- "y"
-- barContext.AO // not found
-- fooContext.AO // found - 20</code></pre>

<pre><code>- "z"
-- barContext.AO // found - 30</code></pre>


作用域的特性
--------
* * *
接下来为大家介绍一些与作用域链和函数的\[\[Scope\]\]属性相关的重要特性。  


闭包
--------
* * *
在ECMAScript中，闭包和函数的\[\[Scope\]\]属性息息相关。正如此前介绍的，\[\[Scope\]\]是在函数创建的时候就保存在函数对象上了，并且直到函数销毁的时候才消失。
事实上，闭包就是函数代码和其\[\[Scope\]\]属性的组合。因此，\[\[Scope\]\]包含了函数创建所在的_词法环境_（上层变量对象）。
上层上下文中的变量，可以在函数激活的时候，通过变量对象的词法链（函数创建的时候就保存起来了）查询到。  

如下例子所示：  
<pre><code>var x = 10;
 
function foo() {
  alert(x);
}
 
(function () {
  var x = 20;
  foo(); // 10, but not 20
})();</code></pre>

我们看到变量“x”是在“foo”函数的\[\[Scope\]\]中找到的。对于变量查询而言，词法链是在函数创建的时候就定义的，而不是在使用的调用的动态链（这个时候，变量“x”才会是20）。  

下面是另外一个（典型的）闭包的例子：  
<pre><code>function foo() {
 
  var x = 10;
  var y = 20;
 
  return function () {
    alert([x, y]);
  };
 
}
 
var x = 30;
 
var bar = foo(); // anonymous function is returned
 
bar(); // [10, 20]</code></pre>

上述例子再一次证明了处理标识符的时候，词法作用域链是在函数创建的时候定义的——变量“x”的值是10，而不是30。
并且，上述例子清楚的展示了函数（上述例子中指的是函数“foo”返回的匿名函数）的\[\[Scope\]\]属性，即使在创建该函数的上下文结束的时候依然存在。  

更多关于ECMAScript对闭包的实现细节会在第六章-闭包中做介绍。  


通过函数构造器创建的函数\[\[Scope\]\]属性
--------
* * *
在前面的例子中，我们看到

