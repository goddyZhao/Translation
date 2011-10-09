说明
--------
* * *
此文译自[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/) 的文章[Scope Chain](http://dmitrysoshnikov.com/ecmascript/chapter-4-scope-chain/)    


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
