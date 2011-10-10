说明
--------
* * *
此文译自[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/) 的文章[Execution Context](http://dmitrysoshnikov.com/ecmascript/chapter-1-execution-contexts/)


概要
--------
* * *
本文将向大家介绍ECMAScript的执行上下文以及相关的可执行代码类型。  

定义
--------  
* * *
每当控制器到达ECMAScript可执行代码的时候，控制器就进入了一个执行上下文。  
执行上下文（简称：EC）是个抽象的概念，ECMA-262标准中用它来区分不同类型的可执行代码。 

标准中并没有从技术实现的角度来定义执行上下文的具体结构和类型；这是实现标准的ECMAScript引擎所要考虑的问题。

一系列活动的执行上下文从逻辑上形成一个栈。栈底总是全局上下文，栈顶是当前（活动的）执行上下文。当在不同的执行上下文间切换（退出的而进入新的执行上下文）的时候，栈会被修改（通过压栈或者退栈的形式）。 

可执行代码类型
--------
* * *
可执行代码类型和执行上下文相关。有的时候，当提到代码类型的时候，其实就是在说执行上下文。

举个例子，我们将执行上下文的栈以数组的形式来表示：
<pre><code>ECStask = [ ];</code></pre>

每次控制器进入一个函数（哪怕该函数被递归调用或者作为构造器），都会发生压栈的操作。内置eval函数工作的时候也不例外。

全局代码
--------
* * *
这类代码是在“程序”级别上被处理的：比如，加载一个外部的js文件或者内联的js代码（被包含在&lt;script&gt;&lt;/script&gt;标签内）。全局代码不包含任何函数体内的代码。

在初始化的时候（程序开始），ECStack如下所示：
<pre><code>ECStack = [
    globalContext
];</code></pre>

函数代码
--------
* * *
一旦控制器进入函数代码（各类函数），就会有新的元素会被压栈到ECStack。要注意的是：实体函数代码并不包括内部函数的代码。如下所示，我们调用一个函数，该函数递归调用自己一次：
<pre><code>(function foo(bar){
    if (bar){

    return;

}

foo(true);
})();</code></pre>

之后，ECStack就被修改成如下所示:
<pre><code>//首先激活foo函数
ECStack = [
    <foo> functionContext
    globalContext
];
//递归激活foo函数
ECStack = [
    <foo> functionContext - recursively
    <foo> functionContext
    globalContext
];</code></pre>

每次函数返回，退出当前活动的执行上下文时，ECStack就会被执行对应的退栈操作——先进后出——和传统的栈实现一致。同样的，当抛出未捕获的异常时，也会退出一个或者多个执行上下文，ECStack也会做相应的退栈操作。待这些代码完成之后，ECStack中就只剩下一个执行上下文（globalContext）——直到整个程序结束。

Eval代码
--------
* * *
说到eval代码就比较有意思了。这里要提到一个叫做调用上下文的概念，比如：调用eval函数时候的上下文，就是一个调用上下文，eval函数中执行的动作（例如：变量声明或者函数声明）会影响整个调用上下文：
<pre><code>eval(‘var x = 10’);
(function foo(){
    eval(‘ var y = 20’);
})();
alert(x); // 10
alert(y); // ”y” is not defined</code></pre>

ECStack会被修改为：
<pre><code>ECStack = [
    globalContext
];
//eval(‘var x = 10’);
ECStack.push(
    evalContext,
    callingContext: globalContext
);

// eval exited context
ECStack.pop();

//foo function call
ECStack.push(<foo> functionContext);

//eval(‘ var y = 20’);
ECStack.push(
    evalContext,
    callingContext: <foo> functionContext
);

//return from eval
ECStack.pop();

//return from foo
ECStack.pop();</code></pre>

在1.7以上版本SpiderMonkey的实现中（Firefox，Thunderbird浏览器内置的JS引擎），允许在调用eval函数的时候，将调用上下文作为第二个参数传递给eval函数。因此，如果传入的调用上下文存在的话，就有可能会影响该上下文中原有的私有变量（在该上下文中声明的变量）：
<pre><code>function foo(){
    var x = 1;
    return function() { alert(x); }
};

var bar = foo();

bar(); // 1
eval(‘x = 2’, bar); //传递上下文，影响了内部变量“var x”
bar(); // 2</code></pre>

总结
--------
* * *
这些基本理论对于后面执行上下文相关的细节（诸如变量对象、作用域链等等）分析是非常必要的。

扩展阅读
--------
* * *
ECMA-363-3标准文档的对应的章节—— [10. 执行上下文](http://bclary.com/2004/11/07/#a-10)
