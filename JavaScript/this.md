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


函数代码中This的值
--------
* * *
当_this_在函数代码中的时候，事情就变得有趣多了。这种情况下是最复杂的，并且会引发很多的问题。  

函数代码中this值的第一个特性（同时也是最主要的特性）就是：它并非静态的绑定在函数上。  

正如此前提到的，this的值是在进入上下文的阶段确定的，并且在函数代码中的话，其值每次都会大不相同。  

然而，一旦进入执行代码阶段，其值就不能改变了。比方说，要想给this赋一个新的值是不可能的，因为this根本就不是变量（相反的，在Python语言中，它显示定义的_self_对象是可以在运行时随意更改的）：  
<pre><code>var foo = {x: 10};
 
var bar = {
  x: 20,
  test: function () {
 
    alert(this === bar); // true
    alert(this.x); // 20
 
    this = foo; // error, 不能更改this的值
 
    alert(this.x); // 如果没有错误，则其值为10而不是20
 
  }
 
};
 
// 在进入上下文的时候，this的值就确定了是“bar”对象
// 至于为什么，会在后面作详细介绍
 
bar.test(); // true, 20
 
foo.test = bar.test;
 
// 但是，这个时候，this的值又会变成“foo”
// 纵然我们调用的是同一个函数
 
foo.test(); // false, 10</code></pre>

因此，在函数代码中影响this值的因素是有很多的。  

首先，在一般的函数调用中，this的值是由激活上下文代码的调用者决定的，比如说，调用函数的外层上下文。this的值是由调用表达式的形式决定的。  

理解并谨记这一点是非常必要的，有利于在任何上下文中都能准确的确定this的值。  

影响调用上下文中的this的值的只有可能是调用表达式的形式，也就是调用函数的方式。
（一些关于JavaScript的文章和书籍中指出的“_this_的值取决于函数的定义方式，如果是全局函数，则this的值就会设置为全局对象，如果是某个对象的方法，则this的值就会设置为该对象”——这纯属扯淡，根本就是在误人子弟）。
正如此前大家看到的，纵然是全局函数，this的值也会随着函数调用方式的不同而不同：  
<pre><code>function foo() {
  alert(this);
}
 
foo(); // global
 
alert(foo === foo.prototype.constructor); // true
 
// 然而，同样的函数，以另外一种调用方式的话，this的值就不同了
 
foo.prototype.constructor(); // foo.prototype</code></pre>

调用一个对象的某个方法的时候，this的值也有可能不是该对象的：  
<pre><code>var foo = {
  bar: function () {
    alert(this);
    alert(this === foo);
  }
};
 
foo.bar(); // foo, true
 
var exampleFunc = foo.bar;
 
alert(exampleFunc === foo.bar); // true
 
// 同样地，相同的函数以不同的调用方式，this的值也就不同了
 
exampleFunc(); // global, false</code></pre>

那么，究竟调用表达式的方式是如何影响this的值的呢？为了完全搞明白这其中的奥妙，首先，这里有必要先介绍一种内部类型——引用类型（the _Reference_ type）。  


引用类型
--------
* * *
引用类型的值可以用伪代码表示为一个拥有两个属性的对象——_base_属性（属性所属的对象）以及该base对象中的_propertyName_属性：  
<pre><code>var valueOfReferenceType = {
  base: <base object>,
  propertyName: <property name>
};</code></pre>

引用类型的值只有可能是以下两种情况：  

1.  当处理一个标识符的时候
2.  或者进行属性访问的时候

关于标识符的处理会在第四章——所用域链中作介绍，这里我们只要注意的是，此算法总返回一个引用类型的值（这对this的值是至关重要的）。  

标识符其实就是变量名，函数名，函数参数名以及全局对象的未受限的属性。如下所示：  
<pre><code>var foo = 10;
function bar() {}</code></pre>

中间过程中，对应的引用类型的值如下所示：  
<pre><code>var fooReference = {
  base: global,
  propertyName: 'foo'
};
 
var barReference = {
  base: global,
  propertyName: 'bar'
};</code></pre>

要从引用类型的值中获取一个对象实际的值需要_GetValue_方法，该方法用伪代码可以描述成如下形式：  
<pre><code>function GetValue(value) {
 
  if (Type(value) != Reference) {
    return value;
  }
 
  var base = GetBase(value);
 
  if (base === null) {
    throw new ReferenceError;
  }
 
  return base.[[Get]](GetPropertyName(value));
 
}</code></pre>

上述代码中的_\[\[Get\]\]_方法返回了对象属性实际的值，包括从原型链中继承的属性：  
<pre><code>GetValue(fooReference); // 10
GetValue(barReference); // function object "bar"</code></pre>

对于属性访问来说，有两种方式： _点符号_（这时属性名是正确的标识符并且提前已经知道了）或者_中括号符号_：  
<pre><code>foo.bar();
foo['bar']();</code></pre>

中间过程中，得到如下的引用类型的值：  
<pre><code>var fooBarReference = {
  base: foo,
  propertyName: 'bar'
};
 
GetValue(fooBarReference); // function object "bar"</code></pre>

问题又来了，引用类型的值又是如何影响函数上下文中this的值的呢？——非常重要。这也是本文的重点。总的来说，决定函数上下文中this的值的规则如下所示：  
>  函数上下文中this的值是函数调用者提供并且由当前调用表达式的形式而定的。
>  如果在调用括号()的左边，有引用类型的值，那么this的值就会设置为该引用类型值的base对象。
>  所有其他情况下（非引用类型），this的值总是_null_。然而，由于null对于this来说没有任何意义，因此会隐式转换为全局对象。

如下所示：  
<pre><code>function foo() {
  return this;
}
 
foo(); // global</code></pre>

上述代码中，调用括号的左侧是引用类型的值（因为_foo_是标识符）：  
<pre><code>var fooReference = {
  base: global,
  propertyName: 'foo'
};</code></pre>

相应的，this的值会设置为引用类型值的base对象，这里就是全局对象。  

属性访问也是类似的：  
<pre><code>var foo = {
  bar: function () {
    return this;
  }
};
 
foo.bar(); // foo</code></pre>

同样的，也是引用类型的值，它的base对象是foo对象，激活bar函数的时候，this的值就设置为foo对象了：  
<pre><code>var fooBarReference = {
  base: foo,
  propertyName: 'bar'
};</code></pre>

然而，同样的函数以不同的激活方式的话，this的值就完全不同了：  
<pre><code>var test = foo.bar;
test(); // global</code></pre>

因为test也是标识符，这样就产生了另外的引用类型的值，其中base对象（全局对象）就是this的值：  
<pre><code>var testReference = {
  base: global,
  propertyName: 'test'
};</code></pre>

至此，我们就可以精确的解释，为什么同样的函数，以不同的调用方式激活，this的值也会不同了——答案就是处理过程中，是不同的引用类型的值：  
<pre><code>function foo() {
  alert(this);
}
 
foo(); // global, 因为
 
var fooReference = {
  base: global,
  propertyName: 'foo'
};
 
alert(foo === foo.prototype.constructor); // true
 
// 另一种调用方式
 
foo.prototype.constructor(); // foo.prototype, 因为
 
var fooPrototypeConstructorReference = {
  base: foo.prototype,
  propertyName: 'constructor'
};</code></pre>

如下是另外一种（典型的）利用调用表达式来动态决定this值的例子：  
<pre><code>function foo() {
  alert(this.bar);
}
 
var x = {bar: 10};
var y = {bar: 20};
 
x.test = foo;
y.test = foo;
 
x.test(); // 10
y.test(); // 20</code></pre>


函数调用以及非引用类型
--------
* * *
正如此前提到过的，当调用括号左侧为非引用类型的时候，this的值会设置为null，并最终变成全局对象。  

我们来考虑下如下表达式：  
<pre><code>(function () {
  alert(this); // null => global
})();</code></pre>

上述例子中，有函数对象，但非引用类型对象（因为它不既不是标识符也不属于属性访问），因此，this的值最终设置为全局对象。

如下是更为复杂的例子：  
<pre><code>var foo = {
  bar: function () {
    alert(this);
  }
};
 
foo.bar(); // Reference, OK => foo
(foo.bar)(); // Reference, OK => foo
 
(foo.bar = foo.bar)(); // global?
(false || foo.bar)(); // global?
(foo.bar, foo.bar)(); // global?</code></pre>

看了上述代码，你可能又有疑问了：为什么明明是属性访问，但是最终this的值不是base对象而是全局对象呢？  

这里主要疑问在最后三个表达式，这三个表达式添加了特定的操作之后，调用括号左侧就不再是引用类型的值了。  

第一种情况——非常明确，是引用类型，最终this的值设置为base对象，foo。  

第二种情况有一个组操作符（_grouping operator_），该操作符不会触发调用获取引用类型实际值的方法，比如：_GetValue_方法。
相应的，处理组操作符中间过程中——获得的仍然是一个引用类型的值，这也就解释了为什么this的值设置成了base对象，_foo_。  

第三种情况是一个_赋值操作符_（_assignment operator_）,与组操作符不同的是，它会触发调用GetValue方法（参见[11.13.1](http://bclary.com/2004/11/07/#a-11.13.1)中的第三步）。
最后返回的时候就是一个函数对象了（而不是引用类型的值了），这就意味着this的值会设置为null，最终会变成全局对象。  


