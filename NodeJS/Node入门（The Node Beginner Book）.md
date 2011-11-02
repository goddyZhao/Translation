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
    * [路由给真正的请求处理程序](#routing-to-real-request-handlers)  
    * [让请求处理程序作出响应](#making-request-handlers-respond)  
        * [不好的实现方式](#how-to-not-do-it)  
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
事实上，JavaScript是一门“完整”的语言： 它可以使用在不同的上下文中，其能力与其他同类语言相比有过之而无不及。  

Node.js事实上就是另外一种上下文，它允许在后端（脱离浏览器环境）运行JavaScript代码。  

要实现在后台运行JavaScript代码，代码需要先被解释然后正确的执行。Node.js的原理正是如此，它使用了Google的V8虚拟机（Google Chrome浏览器使用的JavaScript执行环境），来解释和执行JavaScript代码。  

除此之外，伴随着Node.js的还有许多有用的模块，它们可以简化很多重复的劳作，比如向终端输出字符串。  

因此，Node.js事实上既是一个运行时环境，同时又是一个库。  

要使用Node.js,首先需要进行安装。关于如何安装Node.js，这里就不赘述了，可以直接参考[官方的安装指南](https://github.com/joyent/node/wiki/Installation)。安装完成后，继续回来阅读本书下面的内容。  


<a name="hello-world"></a>
### “Hello World”  
好了，“废话”不多说了，马上开始我们第一个Node.js应用：“Hello World”。  

打开你最喜欢的编辑器，创建一个_helloworld.js_文件。我们要做就是向STDOUT输出“Hello World”，如下是实现该功能的代码：  
<pre><code>console.log("Hello World");</code></pre>

保存该文件，并通过Node.js来执行：  
<pre><code>node helloword.js</code></pre>

正常的话，就会在终端输出_Hello World_。  

好吧，我承认这个应用是有点无趣，那么下面我们就来点“干货”。  


<a name="a-full-blown-web-application-with-nodejs"></a>
## 一个完整的基于Node.js的web应用  
*这部分由Gray完成*  


<a name="building-the-application-stack"></a>
## 构建应用的模块  
*这部分由Monday完成*  


<a name="making-the-request-handlers-respond"></a>
### 让请求处理程序作出响应  

很好。不过现在要是请求处理程序能够向浏览器返回一些有意义的信息而并非全是“Hello World”，那就更好了。  

这里要记住的是，浏览器发出请求后获得并显示的“Hello World”信息仍是来自于我们server.js文件中的_onRequst_函数。  

其实“处理请求”说白了就是“对请求作出响应”，因此，我们需要让请求处理程序能够像_onRequest_函数那样可以和浏览器进行“对话”。  


<a name="how-to-not-do-it"></a>
#### 不好的实现方式
对于我们这样拥有PHP或者Ruby技术背景的开发者来说，最直截了当的实现方式事实上并不是非常靠谱： 看似有效，实则未必如此。  

这里我指的”直截了当的实现方式“意思是：让请求处理程序通过_onRequest_函数直接返回（return()）他们要展示给用户的信息。  

我们先就这样去实现，然后再来看为什么这不是一种很好的实现方式。  

让我们从让请求处理程序返回需要在浏览器中显示的信息开始。我们需要将requestHandler.js修改为如下形式：  
<pre><code>function start() {
  console.log("Request handler 'start' was called.");
  return "Hello Start";
}

function upload() {
  console.log("Request handler 'upload' was called.");
  return "Hello Upload";
}

exports.start = start;
exports.upload = upload;</code></pre>

好的。同样的，请求路由需要将请求处理程序返回给它的信息返回给服务器。因此，我们需要将_router.js_修改为如下形式：  
<pre><code>function route(handle, pathname) {
  console.log("About to route a request for " + pathname);
  if (typeof handle[pathname] === 'function') {
    return handle[pathname]();
  } else {
    console.log("No request handler found for " + pathname);
    return "404 Not found";
  }
}

exports.route = route;</code></pre>

正如上述代码所示，当请求如何路由的时候，我们也返回了一些相关的错误信息。  

最后，我们需要对我们的server.js进行重构以使得它能够将请求处理程序通过请求路由返回的内容响应给浏览器，如下所示：  
<pre><code>var http = require("http");
var url = require("url");

function start(route, handle) {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");

    response.writeHead(200, {"Content-Type": "text/plain"});
    var content = route(handle, pathname)
    response.write(content);
    response.end();
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;</code></pre>

如果我们运行重构后的应用，一切都会工作的很好： 请求http://localhost:8888/start,浏览器会输出“Hello Start”，请求http://localhost:8888/upload会输出“Hello Upload”,而请求http://localhost:8888/foo会输出“404 Not found”。  

好，那么问题在哪里呢？简单的说就是： 当未来有请求处理程序需要进行非阻塞的操作的时候，我们的应用就“挂”了。  

没理解？没关系，那就来详细解释下。  


<a name="blocking-and-non-blocking"></a>
#### 阻塞与非阻塞  
正如此前所提到的，当在请求处理程序中包括非阻塞操作时就会出问题。但是，在说这之前，我们先来看看什么是阻塞操作。  

我不想去解释“阻塞”和“非阻塞”的具体含义，我们直接来看，当在请求处理程序中加入阻塞操作时会发生什么。  

这里，我们来修改下 _start_请求处理程序，我们让它等待10秒以后再返回“Hello Start”。
因为，JavaScript中没有类似 _sleep()_ 这样的操作，所以这里只能够来点小Hack来模拟实现。  

让我们将_requestHandlers.js_修改成如下形式：  
<pre><code>function start() {
  console.log("Request handler 'start' was called.");

  function sleep(milliSeconds) {
    var startTime = new Date().getTime();
    while (new Date().getTime() < startTime + milliSeconds);
  }

  sleep(10000);
  return "Hello Start";
}

function upload() {
  console.log("Request handler 'upload' was called.");
  return "Hello Upload";
}

exports.start = start;
exports.upload = upload;</code></pre>

上述代码中，当函数_start()_被调用的时候，Node.js会先等待10秒，之后才会返回“Hello Start”。当调用_upload()_的时候，会和此前一样立即返回。  

（当然了，这里只是模拟休眠10秒，实际场景中，这样的阻塞操作有很多，比方说一些长时间的计算操作等。）  

接下来就让我们来看看，我们的改动带来了哪些变化。  

如往常一样，我们先要重启下服务器。为了看到效果，我们要进行一些相对复杂的操作（跟着我一起做）： 首先，打开两个浏览器窗口或者标签页。
在第一个浏览器窗口的地址栏中输入http://localhost:8888/start，但是先不要打开它！  

在第二个浏览器窗口的地址栏中输入http://localhost:8888/upload,同样的，先不要打开它！  

接下来，作如下操作：在第一个窗口中（“/start”）按下回车，然后快速切换到第二个窗口中（“/upload”）按下回车。  

注意，发生了什么： /start URL加载花了10秒，这和我们预期的一样。但是，/upload URL居然也花了10秒，而它在对应的请求处理程序中并没有类似于_sleep()_这样的操作！  

这到底是为什么呢？原因就是_start()_包含了阻塞操作。形象的说就是“它阻塞了所有其他的处理工作”。  

这显然是个问题，因为Node一向是这样来标榜自己的：“_在node中除了代码，所有一切都是并行执行的_”。  

这句话的意思是说，Node.js可以在不新增额外线程的情况下，依然可以对任务进行并行处理 —— Node.js是单线程的。
它通过事件轮询（event loop）来实现并行操作，对此，我们应该要充分利用这一点 —— 尽可能的避免阻塞操作，取而代之，多使用非阻塞操作。  

然而，要用非阻塞操作，我们需要使用回调，通过将函数作为参数传递给其他需要花时间做处理的函数（比方说，休眠10秒，或者查询数据库，又或者是进行大量的计算）。  

对于Node.js来说，它是这样处理的：“嘿，probablyExpensiveFunction()（译者注：这里指的就是需要花时间处理的函数），你继续处理你的事情，我（Node.js线程）先不等你了，我继续去处理你后面的代码，请你提供一个callbackFunction()，等你处理完之后我会去调用该回调函数的，谢谢！”  

（如果想要了解更多关于事件轮询细节，可以阅读Mixu的博文——[理解node.js的事件轮询](http://blog.mixu.net/2011/02/01/understanding-the-node-js-event-loop/)。）  

接下来，我们会介绍一种错误的使用非阻塞操作的方式。  

和上次一样，我们通过修改我们的应用来暴露问题。  

这次我们还是拿start请求处理程序来“开刀”。将其修改成如下形式：  
<pre><code>var exec = require("child_process").exec;

function start() {
  console.log("Request handler 'start' was called.");
  var content = "empty";

  exec("ls -lah", function (error, stdout, stderr) {
    content = stdout;
  });

  return content;
}

function upload() {
  console.log("Request handler 'upload' was called.");
  return "Hello Upload";
}

exports.start = start;
exports.upload = upload;</code></pre>

上述代码中，我们引入了一个新的Node.js模块，_child_process_。之所以用它，是为了实现一个既简单又实用的非阻塞操作： _exec()_。  

exec()做了什么呢？它从Node.js来执行一个shell命令。在上述例子中，我们用它来获取当前目录下所有的文件（“ls -lah”）,然后，当/start URL请求的时候将文件信息输出到浏览器中。  

上述代码是非常直观的： 创建了一个新的变量 _content_（初始值为“empty”），执行“ls -lah”命令，将结果赋值给content，最后将content返回。  

和往常一样，我们启动服务器，然后访问“http://localhost:8888/start”。  

之后会载入了一个漂亮的web页面，其内容为“empty”。怎么回事？  

这个时候，你可能大致已经猜到了，exec()在非阻塞这块发挥了神奇的功效。它其实是个很好的东西，有了它，我们可以执行非常耗时的shell操作而无需迫使我们的应用停下来等待该操作。  

（如果想要证明这一点，可以将“ls -lah”换成比如“find /”这样更耗时的操作来效果）。  

然而，针对浏览器显示的结果来看，我们并不满意我们的非阻塞操作，对吧？  

好，接下来，我们来修正这个问题。在这过程中，让我们先来看看为什么当前的这种方式不起作用。  

问题就在于，为了进行非阻塞工作，exec()使用了回调函数。  

在我们的例子中，该回调函数就是作为第二个参数传递给exec()的匿名函数：  
<pre><code>function (error, stdout, stderr) {
  content = stdout;
}</code></pre>

现在就到了问题根源所在了： 我们的代码是同步执行的，这就意味着在调用exec()之后，Node.js会立即执行 return content;。在这个时候，_content_仍然是“empty”，因为传递给exec()的回调函数还未执行到 —— 因为exec()的操作是异步的。  

我们这里“ls -lah”的操作其实是非常快的（除非当前目录下有上百万个文件）。这也是为什么回调函数也会很快的执行到 —— 不过，不管怎么说它还是异步的。  

为了让效果更加明显，我们想象一个更耗时的命令： “find /”，它在我机器上需要执行1分钟左右的时间，
然而，尽管在请求处理程序中，我把“ls -lah”换成“find /”，当打开/start URL的时候，依然能够立即获得HTTP响应 —— 很明显，当_exec()_在后台执行的时候，Node.js自身会继续执行后面的代码。并且我们这里假设传递给_exec()_的回调函数，只会在“find /”命令执行完成之后才会被调用。  

那究竟我们要如何才能实现将当前目录下的文件列表显示给用户呢？  

好，了解了这种不好的实现方式之后，我们接下来来介绍如何让请求处理程序对浏览器请求作出正确的响应。  


<a name="responding-request-handlers-with-non-blocking-operations"></a>
#### 以非阻塞操作进行请求响应  

 