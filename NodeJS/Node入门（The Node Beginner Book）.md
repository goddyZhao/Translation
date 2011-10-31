# Node入门


<a name="about"></a>
## 关于
本书致力于教会你如何用Node.js来开发应用，过程中会传授你所有所需的“高级”JavaScript知识。本书绝不是一本“Hello World”的教程。  


<a name="status"></a>
### 状态
你正在阅读的已经是本书的最终版。因此，只有当进行错误更正以及针对新版本Node.js的改动进行对应的修正时，才会进行更新。  

本书中的代码案例都在Node.js 0.4.9版本中测试过，可以正确工作。  

<a name="intended-audience"></a>
### 读者对象
本书最适合与我有相似技术背景的读者： 至少对一门诸如Ruby，Python，PHP或者Java这样面向对象的语言有一定的经验；对JavaScript处于初学阶段，并且完全是一个Node.js的新手。  

这里指的适合对其他编程语言有一定经验的开发者，意思是说，本书不会对诸如数据类型、变量、控制结构等等之类的非常基础的概念做介绍。要读懂本书，这些基础的概念我都默认你已经会了。 

然而，本书还是会对JavaScript中的函数和对象做详细介绍，因为它们与其他同类编程语言中的函数和对象有很大的不同。  


<a name="structure"></a>
### 本书结构  
读完本书之后，你将完成一个完整的web应用，该应用允许用户浏览页面以及上传文件。  

当然了，应用本身并没有什么了不起的，相比为了实现该功能书写的代码本身，我们更关注的是如何创建一个框架来对我们应用的不同模块进行干净地剥离。
是不是很玄乎？稍后你就明白了。  

本书先从介绍在Node.js环境中进行JavaScript开发和在浏览器环境中进行JavaScript开发的差异开始。  

紧接着，会带领大家完成一个最传统的“Hello World”应用，这也是最基础的Node.js应用。  

最后，会和大家讨论如何设计一个“真正”完整的应用，剖析要完成该应用需要实现的不同模块，并开始一步一步介绍如何来实现这些模块。  

可以确保的是，在这过程中，大家会学到JavaScript中一些高级的概念、如何使用它们以及为什么使用这些概念就可以实现而其他编程语言中同类的概念就无法实现。  

该应用所有的源代码都可以通过[本书Github代码仓库](https://github.com/ManuelKiessling/NodeBeginnerBook/tree/master/code/application)来获取。  


### 目录
*  [关于](#about)  
    * [状态](#status)   
    * [读者对象](#intended-audience)   
    * [本书结构](#structure)  
*  [JavaScript与Node.js](#javascript-and-nodejs)  
    * [JavaScript与你](#javascript-and-you)  
    * [简短申明](#a-word-of-warning)  
    * [服务器端JavaScript](#server-side-javascript)  
    * [“Hello World”](#hello-world)  
*  [一个完整的基于Node.js的web应用](#a-full-blown-web-application-with-nodejs)  
    * [用例](#the-use-cases)  
    * [应用不同模块分析](#the-application-stack)  
*  [构建应用的模块](#building-the-application-stack)  
    * [一个基础的HTTP服务器](#a-basic-http-server)  
    * [分析HTTP服务器](#analyzing-our-http-server)  
    * [进行函数传递](#passing-functions-around)  
    * [函数传递是如何让HTTP服务器工作的](#how-function-passing-makes-our-http-server-work)  
    * [基于事件驱动的回调](#event-driven-callbacks)  
    * [服务器是如何处理请求的](#how-our-server-handles-requests)  
    * [服务端的模块放在哪里](#finding-a-place-for-our-server-module)  
    * [如何来进行请求的“路由”](#whats-needed-to-route-requests)  
    * [行为驱动执行](#execution-in-the-kindom-of-verbs)  
    * [路由给真正的请求处理器](#routing-to-real-request-handlers)  
    * [让请求处理器作出响应](#making-request-handlers-respond)  
        * [如何不这样做](#how-to-not-do-it)  
        * [阻塞与非阻塞](#blocking-and-non-blocking)  
        * [以非阻塞操作进行请求响应](#responding-request-handlers-with-non-blocking-operations)  
    * [更有用的场景](#serving-something-useful)  
        * [处理POST请求](#handling-post-requests)  
        * [处理文件上传](#handling-file-uploads)  
* [总结与展望](#conclusion-and-outlook)  


## JavaScript与Node.js  

### JavaScript与你  
抛开技术，我们先来聊聊你以及你和JavaScript的关系。本章的主要目的是想让你看看，对你而言是否有必要继续阅读后续章节的内容。  

如果你和我一样，那么你很早就开始利用HTML进行“开发”，正因如此，你接触到了叫JavaScript有趣的东西，而对于JavaScript，你只会基本的操作——为web页面添加交互。  

而你真正想要的是“干货”，你想要知道如何构建复杂的web站点 —— 于是，你学习了一种诸如PHP，Ruby，Java之类的编程语言，并开始书写“后端”代码。  

然而，你始终还在关注着JavaScript，随着通过一些对jQuery，Prototype之类技术的介绍，你慢慢了解到了很多JavaScript中的进阶技能，同时也感受到了JavaScript绝非仅仅是_window.open()_那么简单。  

不过，这些毕竟都是前端技术，尽管当你想要增强页面的时候，使用jQuery总让你觉得很爽，但是，最终，你顶多是个JavaScript用户，而非JavaScript开发者。  

然后，出现了Node.js，服务端的JavaScript, 这会有多酷啊？  

于是，你决定，是时候重新拾起这个既熟悉又陌生的JavaScript了。但是别急，写Node.js应用是一件事情；理解为什么它们要以它们书写的这种方式来书写则意味着——你要懂JavaScript。这次是玩真的了。  

问题来了： 由于JavaScript真正意义上以两种，甚至可以说是三种形态存在（从中世纪90年代的作为对DHTML进行增强的小玩具，到像jQuery那样严格意义上的前端技术，一直到现在的服务端技术），
因此，很难找到一个“正确”的方式来学习JavaScript，使得让你书写Node.js应用的时候感觉自己是在真正开发它而不仅仅是使用它。  

因为这就是关键： 你本身已经是个有经验的开发者，你不想通过到处寻找各种解决方案（其中可能还有不正确的）来学习新的技术，你要确保自己是通过正确的方式来学习这项技术。  

当然了，外面不乏很优秀的学习JavaScript的文章。但是，有的时候光靠那些文章是远远不够的。你需要的是指导。  

本书的目标就是给你提供指导。  


<a name="a-word-of-warning"></a>
### 简短申明
业界有非常优秀的JavaScript程序员。而我并非其中一员。  

我就是上一节中描述的那个我。我熟悉如何开发后端web应用，但是对“真正”的JavaScript以及Node.js，我都只是新手。
我也只是最近学习了一些JavaScript的高级概念，并没有实践经验。  

因此，本书并不是一本“从入门到精通”的书，更像是一本“从初级入门到高级入门”的书。  

如果成功的话，那么本书就是我当初开始学习Node.js最希望拥有的教程。  


<a name="server-side-javascript"></a>
### 服务端JavaScript  
JavaScript最早是运行在浏览器中，然而浏览器只是提供了一个上下文，它定义了使用JavaScript可以做什么，但并没有“说”太多关于JavaScript语言本身可以做什么。
事实上，JavaScript是一门“完整”的语言： 它可以使用在不同的上下文中，其能力与其他同类语言有过之而无不及。  

Node.js事实上就是另外一种上下文，它允许在后端（脱离浏览器环境）运行JavaScript代码。  

要实现在后台运行JavaScript代码，代码需要先被解释然后正确的执行。Node.js的原理正是如此，它使用了Google的V8虚拟机（Google Chrome浏览器使用的JavaScript执行环境），来解释和执行JavaScript代码。  

除此之外，伴随着Node.js的还有许多有用的模块，它们可以简化很多重复的劳作，比如向终端输出字符串。  

因此，Node.js事实上既是一个运行时环境，同时又是一个库。  

要使用Node.js,首先需要进行安装。关于如何安装Node.js，这里就不赘述了，可以直接参考[官方的安装指南](https://github.com/joyent/node/wiki/Installation)。安装完成后，继续回来阅读本书下面的内容。  


<a name="hello-world"></a>
### “Hello World”  


