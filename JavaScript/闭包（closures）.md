说明
--------
* * *
此文译自[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/) 的文章[closures](http://dmitrysoshnikov.com/ecmascript/chapter-6-closures/)  
另，此文还有另外一位同事（彭森材）共同参译  


概要
--------
* * *
本文将介绍一个在JavaScript经常会拿来讨论的话题 —— _闭包（closure）_。闭包其实已经是个老生常谈的话题了；
有大量文章都介绍过闭包的内容（其中不失一些很好的文章，比如，扩展阅读中Richard Cornford的文章就非常好），
尽管如此，这里还是要试着从理论角度来讨论下闭包，看看ECMAScript中的闭包内部究竟是如何工作的。  

正如在此前文章中提到的，这些文章都是系列文章，相互之间都是有关联的。因此，为了更好的理解本文要介绍的内容，
建议先去阅读下[第四章 - 作用域链](http://goddyzhao.tumblr.com/post/11259644092/scope-chain)和
[第二章 - 变量对象](http://goddyzhao.tumblr.com/post/11141710441/variable-object)。  


概论
--------
* * *
在讨论ECMAScript闭包之前，先来介绍下[函数式编程](http://en.wikipedia.org/wiki/Functional_programming)（与ECMA-262-3	标准无关）中一些基本定义。
然而，为了更好的解释这些定义，这里还是拿ECMAScript来举例。  

众所周知，在函数式语言中（ECMAScript也支持这种风格），函数即是数据。就比方说，函数可以保存在变量中，可以当参数传递给其他函数，还可以当返回值返回等等。
这类函数有特殊的名字和结构。  


定义
--------
* * *
>  函数式参数（“_Funarg_”） —— 是指值为函数的参数。  

如下例子：  
<pre><code>function exampleFunc(funArg) {
  funArg();
}
 
exampleFunc(function () {
  alert('funArg');
});</code></pre>

上述例子中funarg的实参是一个传递给_exampleFunc_的匿名函数。  

反过来，接受函数式参数的函数称为 _高阶函数（high-order function 简称：HOF）_。还可以称作：_函数式函数_ 或者 偏数理的叫法：_操作符函数_。
上述例子中，_exampleFunc_ 就是这样的函数。  

此前提到的，函数不仅可以作为参数，还可以作为返回值。这类以函数为返回值的函数称为 _带函数值的函数（functions with functional value or function valued functions）。  
<pre><code>(function functionValued() {
  return function () {
    alert('returned function is called');
  };
})()();</code></pre>

可以以正常数据形式存在的函数（比方说：当参数传递，接受函数式参数或者以函数值返回）都称作 _第一类函数（一般说第一类对象）_。
在ECMAScript中，所有的函数都是第一类对象。  

接受自己作为参数的函数，称为 _自应用函数（auto-applicative function 或者 self-applicative function）_：  
<pre><code>(function selfApplicative(funArg) {
 
  if (funArg && funArg === selfApplicative) {
    alert('self-applicative');
    return;
  }
 
  selfApplicative(selfApplicative);
 
})();</code></pre>

以自己为返回值的函数称为 _自复制函数（auto-replicative function 或者 self-replicative function）_。
通常，“自复制”这个词用在文学作品中：  
<pre><code>(function selfReplicative() {
  return selfReplicative;
})();</code></pre>

在函数式参数中定义的变量，在“funarg”激活时就能够访问了（因为存储上下文数据的变量对象每次在进入上下文的时候就创建出来了）：  
<pre><code>function testFn(funArg) {
 
  // 激活funarg, 本地变量localVar可访问
  funArg(10); // 20
  funArg(20); // 30
 
}
 
testFn(function (arg) {
 
  var localVar = 10;
  alert(arg + localVar);
 
});</code></pre>

然而，我们知道（特别在第四章中提到的），在ECMAScript中，函数是可以封装在父函数中的，并可以使用父函数上下文的变量。
这个特性会引发 _funarg问题_。  


Funarg问题
--------
* * *
在[面向堆栈的编程语言](http://en.wikipedia.org/wiki/Stack-oriented_programming_language)中，函数的本地变量都是保存在 _堆栈_上的，
每当函数激活的时候，这些变量和函数参数都会压栈到该堆栈上。  

当函数返回的时候，这些参数又会从堆栈中移除。这种模型对将函数作为函数式值使用的时候有很大的限制（比方说，作为返回值从父函数中返回）。
绝大部分情况下，问题会出现在当函数有 _自由变量_的时候。  
>  自由变量是指在函数中使用的，但既不是函数参数也不是函数的局部变量的变量  

如下所示：  
<pre><code>function testFn() {
 
  var localVar = 10;
 
  function innerFn(innerParam) {
    alert(innerParam + localVar);
  }
 
  return innerFn;
}
 
var someFn = testFn();
someFn(20); // 30</code></pre>

上述例子中，对于**innerFn**函数来说，**localVar**就属于自由变量。  

对于采用 _面向堆栈_模型来存储局部变量的系统而言，就意味着当**testFn**函数调用结束后，其局部变量都会从堆栈中移除。
这样一来，当从外部对**innerFn**进行函数调用的时候，就会发生错误（因为localVar变量已经不存在了）。

而且，上述例子在 _面向堆栈_实现模型中，要想将**innerFn**以返回值返回根本是不可能的。
因为它也是**testFn**函数的局部变量，也会随着**testFn**的返回而移除。  

还有一个函数对象问题和当系统采用[动态作用域](http://en.wikipedia.org/wiki/Scope_(programming)#Dynamic_scoping)，函数作为函数参数使用的时候有关。  

看如下例子（伪代码）：  
<pre><code>var z = 10;
 
function foo() {
  alert(z);
}
 
foo(); // 10 – 静态作用域和动态作用域情况下都是
 
(function () {
 
  var z = 20;
  foo(); // 10 – 静态作用域情况下, 20 – 动态作用域情况下
 
})();
 
// 将foo函数以参数传递情况也是一样的
 
(function (funArg) {
 
  var z = 30;
  funArg(); // 10 – 静态作用域情况下, 30 – 动态作用域情况下
 
})(foo);</code></pre>


我们看到，采用动态作用域，变量（标识符）处理是通过动态堆栈来管理的。
因此，自由变量是在当前活跃的动态链中查询的，而不是在函数创建的时候保存起来的静态作用域链中查询的。  

这样就会产生冲突。比方说，即使Z仍然存在（与之前从堆栈中移除变量的例子相反），还是会有这样一个问题： 在不同的函数调用中，Z的值到底取哪个呢（从哪个上下文，哪个作用域中查询）？  

上述描述的就是两类 _funarg问题_ —— 取决于是否将函数以返回值返回（第一类问题）以及是否将函数当函数参数使用（第二类问题）。  

为了解决上述问题，就引入了 _闭包_的概念。  


闭包
--------
* * *
>  闭包是代码块和创建该代码块的上下文中数据的结合。  

让我们来看下面这个例子（伪代码）：  
<pre><code>var x = 20;
 
function foo() {
  alert(x); // 自由变量 "x" == 20
}
 
// foo的闭包
fooClosure = {
  call: foo // 对函数的引用
  lexicalEnvironment: {x: 20} // 查询自由变量的上下文
};</code></pre>

上述例子中，“fooClosure”部分是伪代码。对应的，在ECMAScript中，“foo”函数已经有了一个内部属性——创建该函数上下文的作用域链。  

这里“lexical”是不言而喻的，通常是省略的。上述例子中是为了强调在闭包创建的同时，上下文的数据就会保存起来。
当下次调用该函数的时候，自由变量就可以在保存的（闭包）上下文中找到了，正如上述代码所示，变量“z”的值总是10。  

定义中我们使用的比较广义的词 —— “代码块”，然而，通常（在ECMAScript中）会使用我们经常用到的函数。
当然了，并不是所有对闭包的实现都会将闭包和函数绑在一起，比方说，在Ruby语言中，闭包就有可能是： 一个程序对象（procedure object）, 一个lambda表达式或者是代码块。  

对于要实现将局部变量在上下文销毁后仍然保存下来，基于堆栈的实现显然是不适用的（因为与基于堆栈的结构相矛盾）。
因此在这种情况下，上层作用域的闭包数据是通过 _动态分配内存_的方式来实现的（基于“堆”的实现），配合使用垃圾回收器（_garbage collector_简称GC）和 _引用计数（reference counting）_。
这种实现方式比基于堆栈的实现性能要低，然而，任何一种实现总是可以优化的： 可以分析函数是否使用了自由变量，函数式参数或者函数式值，然后根据情况来决定 —— 是将数据存放在堆栈中还是堆中。  



ECMAScript闭包的实现
--------
* * *
讨论完理论部分，接下来让我们来介绍下ECMAScript中闭包究竟是如何实现的。
这里还是有必要再次强调下：ECMAScript只使用[静态（词法）作用域](http://en.wikipedia.org/wiki/Scope_(programming)#Lexical_scoping)（而诸如Perl这样的语言，既可以使用静态作用域也可以使用动态作用域进行变量声明）。  
<pre><code>var x = 10;
 
function foo() {
  alert(x);
}
 
(function (funArg) {
 
  var x = 20;
 
  // funArg的变量 "x" 是静态保存的，在该函数创建的时候就保存了
 
  funArg(); // 10, 而不是 20
 
})(foo);</code></pre>

从技术角度来说，创建该函数的上层上下文的数据是保存在函数的内部属性 \[\[Scope\]\]中的。
如果你还不了解什么是\[\[Scope\]\]，建议你先阅读[第四章](http://goddyzhao.tumblr.com/post/11259644092/scope-chain),
该章节对\[\[Scope\]\]作了非常详细的介绍。如果你对\[\[Scope\]\]和作用域链的知识完全理解了的话，那对闭包也就完全理解了。  

根据函数创建的算法，我们看到 _在ECMAScript中，所有的函数都是闭包，因为它们都是在创建的时候就保存了上层上下文的作用域链（除开异常的情况）_
（不管这个函数后续是否会激活 —— \[\[Scope\]\]在函数创建的时候就有了）：  
<pre><code>var x = 10;
 
function foo() {
  alert(x);
}
 
// foo is a closure
foo: &lt;FunctionObject&gt; = {
  [[Call]]: &lt;code block of foo&gt;,
  [[Scope]]: [
    global: {
      x: 10
    }
  ],
  ... // other properties
};</code></pre>

正如此前提到过的，出于优化的目的，当函数不使用自由变量的时候，实现层可能就不会保存上层作用域链。
然而，ECMAScript-262-3标准中并未对此作任何说明；因此，严格来说 —— 所有函数都会在创建的时候将上层作用域链保存在\[\[Scope\]\]中。  

有些实现中，允许对闭包作用域直接进行访问。比如Rhino，针对函数的\[\[Scope\]\]属性，对应有一个非标准的 \_\_parent\_\_属性，在第二章中作过介绍：  
<pre><code>var global = this;
var x = 10;
 
var foo = (function () {
 
  var y = 20;
 
  return function () {
    alert(y);
  };
 
})();
 
foo(); // 20
alert(foo.__parent__.y); // 20
 
foo.__parent__.y = 30;
foo(); // 30
 
// 还可以操作作用域链
alert(foo.__parent__.__parent__ === global); // true
alert(foo.__parent__.__parent__.x); // 10</code></pre>


“万能”的\[\[Scope\]\]
--------
* * *
这里还要注意的是：在ECMAScript中，同一个上下文中创建的闭包是共用一个\[\[Scope\]\]属性的。
也就是说，某个闭包对其中的变量做修改会影响到其他闭包对其变量的读取：  
<pre><code>var firstClosure;
var secondClosure;
 
function foo() {
 
  var x = 1;
 
  firstClosure = function () { return ++x; };
  secondClosure = function () { return --x; };
 
  x = 2; // 对AO["x"]产生了影响, 其值在两个闭包的[[Scope]]中
 
  alert(firstClosure()); // 3, 通过 firstClosure.[[Scope]]
}
 
foo();
 
alert(firstClosure()); // 4
alert(secondClosure()); // 3</code></pre>

正因为这个特性，很多人都会犯一个非常常见的错误： 当在循环中创建了函数，然后将循环的索引值和每个函数绑定的时候，通常得到的结果不是预期的（预期是希望每个函数都能够获取各自对应的索引值）。  
<pre><code>var data = [];
 
for (var k = 0; k &lt; 3; k++) {
  data[k] = function () {
    alert(k);
  };
}
 
data[0](); // 3, 而不是 0
data[1](); // 3, 而不是 1
data[2](); // 3, 而不是 2</code></pre>

上述例子就证明了 —— 同一个上下文中创建的闭包是共用一个\[\[Scope\]\]属性的。因此上层上下文中的变量“k”是可以很容易就被改变的。  

如下所示：  
<pre><code>activeContext.Scope = [
  ... // higher variable objects
  {data: [...], k: 3} // activation object
];
 
data[0].[[Scope]] === Scope;
data[1].[[Scope]] === Scope;
data[2].[[Scope]] === Scope;</code></pre>

这样一来，在函数激活的时候，最终使用到的k就已经变成了3了。

如下所示，创建一个额外的闭包就可以解决这个问题了：  
<pre><code>var data = [];
 
for (var k = 0; k &lt; 3; k++) {
  data[k] = (function _helper(x) {
    return function () {
      alert(x);
    };
  })(k); // 将 "k" 值传递进去
}
 
// 现在就对了
data[0](); // 0
data[1](); // 1
data[2](); // 2</code></pre>

上述例子中，函数“_helper”创建出来之后，通过参数“k”激活。其返回值也是个函数，该函数保存在对应的数组元素中。
这种技术产生了如下效果：  在函数激活时，每次“_helper”都会创建一个新的变量对象，其中含有参数“x”，“x”的值就是传递进来的“k”的值。
这样一来，返回的函数的\[\[Scope\]\]就成了如下所示：  
<pre><code>data[0].[[Scope]] === [
  ... // 更上层的变量对象
  上层上下文的AO: {data: [...], k: 3},
  _helper上下文的AO: {x: 0}
];
 
data[1].[[Scope]] === [
  ... // 更上层的变量对象
  上层上下文的AO: {data: [...], k: 3},
  _helper上下文的AO: {x: 1}
];
 
data[2].[[Scope]] === [
  ... // 更上层的变量对象
  上层上下文的AO: {data: [...], k: 3},
  _helper上下文的AO: {x: 2}
];</code></pre>


我们看到，这个时候函数的\[\[Scope\]\]属性就有了真正想要的值了，为了达到这样的目的，我们不得不在\[\[Scope\]\]中创建额外的变量对象。
要注意的是，在返回的函数中，如果要获取“k”的值，那么该值还是会是3。  

顺便提下，大量介绍JavaScript的文章都认为只有额外创建的函数才是闭包，这种说法是错误的。
实践得出，这种方式是最有效的，然而，从理论角度来说，在ECMAScript中所有的函数都是闭包。  

然而，上述提到的方法并不是唯一的方法。通过其他方式也可以获得正确的“k”的值，如下所示：  
<pre><code>var data = [];
 
for (var k = 0; k < 3; k++) {
  (data[k] = function () {
    alert(arguments.callee.x);
  }).x = k; // 将“k”存储为函数的一个属性
}
 
// 同样也是可行的
data[0](); // 0
data[1](); // 1
data[2](); // 2</code></pre>


Funarg和return
--------
* * *
另外一个特性是从闭包中返回。在ECMAScript中，闭包中的返回语句会将控制流返回给调用上下文（调用者）。
而在其他语言中，比如，Ruby，有很多中形式的闭包，相应的处理闭包返回也都不同，下面几种方式都是可能的：可能直接返回给调用者，或者在某些情况下——直接从上下文退出。  

ECMAScript标准的退出行为如下：  
<pre><code>function getElement() {
 
  [1, 2, 3].forEach(function (element) {
 
    if (element % 2 == 0) {
      // 返回给函数"forEach"，
      // 而不会从getElement函数返回
      alert('found: ' + element); // found: 2
      return element;
    }
 
  });
 
  return null;
}
 
alert(getElement()); // null, 而不是 2</code></pre>

然而，在ECMAScript中通过try catch可以实现如下效果：  
<pre><code>var $break = {};
 
function getElement() {
 
  try {
 
    [1, 2, 3].forEach(function (element) {
 
      if (element % 2 == 0) {
        // 直接从getElement"返回"
        alert('found: ' + element); // found: 2
        $break.data = element;
        throw $break;
      }
 
    });
 
  } catch (e) {
    if (e == $break) {
      return $break.data;
    }
  }
 
  return null;
}
 
alert(getElement()); // 2</code></pre>


理论版本
--------
* * *
通常，程序员会错误的认为，只有匿名函数才是闭包。其实并非如此，正如我们所看到的 —— 正是因为作用域链，使得所有的函数都是闭包（与函数类型无关： 匿名函数，FE，NFE，FD都是闭包），
这里只有一类函数除外，那就是通过Function构造器创建的函数，因为其\[\[Scope\]\]只包含全局对象。
为了更好的澄清该问题，我们对ECMAScript中的闭包作两个定义（即两种闭包）：  

ECMAScript中，闭包指的是：  

*  从理论角度：所有的函数。因为它们都在创建的时候就将上层上下文的数据保存起来了。哪怕是简单的全局变量也是如此，因为函数中访问全局变量就相当于是在访问自由变量，这个时候使用最外层的作用域。    
*  从实践角度：以下函数才算是闭包：  
   1. 即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）  
   2. 在代码中引用了自由变量  


闭包实践
--------
* * *
实际使用的时候，闭包可以创建出非常优雅的设计，允许对funarg上定义的多种计算方式进行定制。
如下就是数组排序的例子，它接受一个排序条件函数作为参数：  
<pre><code>[1, 2, 3].sort(function (a, b) {
  ... // 排序条件
});</code></pre>

同样的例子还有，数组的map方法（并非所有的实现都支持数组map方法，SpiderMonkey从1.6版本开始有支持），该方法根据函数中定义的条件将原数组映射到一个新的数组中：  
<pre><code>[1, 2, 3].map(function (element) {
  return element * 2;
}); // [2, 4, 6]</code></pre>

使用函数式参数，可以很方便的实现一个搜索方法，并且可以支持无穷多的搜索条件：  
<pre><code>someCollection.find(function (element) {
  return element.someProperty == 'searchCondition';
});</code></pre>

还有应用函数，比如常见的forEach方法，将funarg应用到每个数组元素：  
<pre><code>[1, 2, 3].forEach(function (element) {
  if (element % 2 != 0) {
    alert(element);
  }
}); // 1, 3</code></pre>

顺便提下，函数对象的 _apply_ 和 _call_方法，在函数式编程中也可以用作应用函数。
apply和call已经在讨论“this”的时候介绍过了；这里，我们将它们看作是应用函数 —— 应用到参数中的函数（在apply中是参数列表，在call中是独立的参数）：  
<pre><code>(function () {
  alert([].join.call(arguments, ';')); // 1;2;3
}).apply(this, [1, 2, 3]);</code></pre>

闭包还有另外一个非常重要的应用 —— 延迟调用：  
<pre><code>var a = 10;
setTimeout(function () {
  alert(a); // 10, 一秒钟后
}, 1000);</code></pre>

也可以用于回调函数：  
<pre><code>...
var x = 10;
// only for example
xmlHttpRequestObject.onreadystatechange = function () {
  // 当数据就绪的时候，才会调用;
  // 这里，不论是在哪个上下文中创建，变量“x”的值已经存在了
  alert(x); // 10
};
..</code></pre>

还可以用于封装作用域来隐藏辅助对象：  
<pre><code>var foo = {};
 
// initialization
(function (object) {
 
  var x = 10;
 
  object.getX = function _getX() {
    return x;
  };
 
})(foo);
 
alert(foo.getX()); // get closured "x" – 10</code></pre>


总结
--------
* * *
本文介绍了更多关于ECMAScript-262-3的理论知识，而我认为，这些基础的理论有助于理解ECMAScript中闭包的概念。


扩展阅读
--------
* * *

*  [JavaScript闭包（by Richard Cornford）](http://jibbering.com/faq/notes/closures/)  
*  [Funarg问题](http://en.wikipedia.org/wiki/Funarg_problem)  
*  [闭包](http://en.wikipedia.org/wiki/Closure_(computer_science))  

