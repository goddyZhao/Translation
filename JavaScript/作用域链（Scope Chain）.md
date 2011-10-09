说明
--------
* * *
此文译自[Dmitry A.Soshnikov](http://dmitrysoshnikov.com/) 的文章[Scope Chain](http://dmitrysoshnikov.com/ecmascript/chapter-4-scope-chain/)    


概要
--------
* * *
在[第二章](http://goddyzhao.tumblr.com/post/11141710441/variable-object)变量对象时候，
已经介绍过[执行上下文](http://goddyzhao.tumblr.com/post/10020230352/execution-context)的数据是以变量对象的属性的形式进行存储的。  

还介绍了，每次进入执行上下文的时候，就会创建变量对象，并且赋予其属性初始值，随后在执行代码阶段会对属性值进行更新。  

本文要与执行上下文密切相关的另外一个重要的概念——_作用域链（Scope Chain）_。  


定义
--------
* * *

