说明
--------
* * *
此文译自[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/) 的文章[this](http://dmitrysoshnikov.com/ecmascript/chapter-3-this/)  


概要
--------
* * *
本文将进一步讨论与[执行上下文](http://goddyzhao.tumblr.com/post/10020230352/execution-context)密切相关的概念——_this_关键字。  

事实证明，_this_这块的内容非常的复杂，它在不同执行上下文的情况下其值都会不同，并且会相应的引发一些问题。  

很多程序员一看到_this_关键字，就会把它和面向对象的编程方式联系在一起，它指向利用构造器新创建出来的对象。在ECMAScript中，也支持this，然而，
正如大家所熟知的，this不仅仅只用来表示创建出来的对象。  

接下来给大家揭开在ECMAScript中this神秘的面纱。  


定义
--------
* * *
_This_是执行上下文的一个属性：
<pre><code>activeExecutionContext = {
  VO: {...},
  this: thisValue
};</code></pre>

这里的VO就是前一章介绍的[变量对象](http://goddyzhao.tumblr.com/post/11141710441/variable-object)。  

_This_与上下文的可执行代码类型有关，其值在_进入上下文阶段_就确定了，并且在_执行代码阶段_是不能改变的。  

下面就来详细对其作个介绍。  


全局代码中This的值
--------
* * *
这种情况下，一切都变得非常简单，_this_的值总是_全局对象_本身;因此，可以间接地获取引用：  
<pre><code>// 显式定义全局对象的属性
this.a = 10; // global.a = 10
alert(a); // 10
 
// 通过赋值给不受限的标识符来进行隐式定义
b = 20;
alert(this.b); // 20
 
// 通过变量申明来进行隐式定义
// 因为全局上下文中的变量对象就是全局对象本身
var c = 30;
alert(this.c); // 30</code></pre>


