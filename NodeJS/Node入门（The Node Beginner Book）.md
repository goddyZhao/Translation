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
本书最适合与我有相似技术背景的读者： 至少对一门诸如Ruby、Python、PHP或者Java这样面向对象的语言有一定的经验；对JavaScript处于初学阶段，并且完全是一个Node.js的新手。  

这里指的适合对其他编程语言有一定经验的开发者，意思是说，本书不会对诸如数据类型、变量、控制结构等等之类非常基础的概念作介绍。要读懂本书，这些基础的概念我都默认你已经会了。 

然而，本书还是会对JavaScript中的函数和对象作详细介绍，因为它们与其他同类编程语言中的函数和对象有很大的不同。  


<a name="structure"></a>
### 本书结构  
读完本书之后，你将完成一个完整的web应用，该应用允许用户浏览页面以及上传文件。  

当然了，应用本身并没有什么了不起的，相比为了实现该功能书写的代码本身，我们更关注的是如何创建一个框架来对我们应用的不同模块进行干净地剥离。
是不是很玄乎？稍后你就明白了。  

本书先从介绍在Node.js环境中进行JavaScript开发和在浏览器环境中进行JavaScript开发的差异开始。  

紧接着，会带领大家完成一个最传统的“Hello World”应用，这也是最基础的Node.js应用。  

最后，会和大家讨论如何设计一个“真正”完整的应用，剖析要完成该应用需要实现的不同模块，并一步一步介绍如何来实现这些模块。  

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

如果你和我一样，那么你很早就开始利用HTML进行“开发”，正因如此，你接触到了这个叫JavaScript有趣的东西，而对于JavaScript，你只会基本的操作——为web页面添加交互。  

而你真正想要的是“干货”，你想要知道如何构建复杂的web站点 —— 于是，你学习了一种诸如PHP、Ruby、Java这样的编程语言，并开始书写“后端”代码。  

与此同时，你还始终关注着JavaScript，随着通过一些对jQuery，Prototype之类技术的介绍，你慢慢了解到了很多JavaScript中的进阶技能，同时也感受到了JavaScript绝非仅仅是 _window.open()_ 那么简单。  

不过，这些毕竟都是前端技术，尽管当想要增强页面的时候，使用jQuery总让你觉得很爽，但到最后，你顶多是个JavaScript用户，而非JavaScript开发者。  

然后，出现了Node.js，服务端的JavaScript，这有多酷啊？  

于是，你觉得是时候该重新拾起既熟悉又陌生的JavaScript了。但是别急，写Node.js应用是一件事情；理解为什么它们要以它们书写的这种方式来书写则意味着——你要懂JavaScript。这次是玩真的了。  

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

打开你最喜欢的编辑器，创建一个 _helloworld.js_ 文件。我们要做就是向STDOUT输出“Hello World”，如下是实现该功能的代码：  
<pre><code>console.log("Hello World");</code></pre>

保存该文件，并通过Node.js来执行：  
<pre><code>node helloword.js</code></pre>

正常的话，就会在终端输出 _Hello World_ 。  

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

这里要记住的是，浏览器发出请求后获得并显示的“Hello World”信息仍是来自于我们server.js文件中的 _onRequest_ 函数。  

其实“处理请求”说白了就是“对请求作出响应”，因此，我们需要让请求处理程序能够像 _onRequest_ 函数那样可以和浏览器进行“对话”。  


<a name="how-to-not-do-it"></a>
#### 不好的实现方式
对于我们这样拥有PHP或者Ruby技术背景的开发者来说，最直截了当的实现方式事实上并不是非常靠谱： 看似有效，实则未必如此。  

这里我指的“直截了当的实现方式”意思是：让请求处理程序通过 _onRequest_ 函数直接返回（return()）他们要展示给用户的信息。  

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

好的。同样的，请求路由需要将请求处理程序返回给它的信息返回给服务器。因此，我们需要将 _router.js_ 修改为如下形式：  
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

正如上述代码所示，当请求无法路由的时候，我们也返回了一些相关的错误信息。  

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

如果我们运行重构后的应用，一切都会工作的很好： 请求http://localhost:8888/start, 浏览器会输出“Hello Start”，请求http://localhost:8888/upload 会输出“Hello Upload”,而请求http://localhost:8888/foo 会输出“404 Not found”。  

好，那么问题在哪里呢？简单的说就是： 当未来有请求处理程序需要进行非阻塞的操作的时候，我们的应用就“挂”了。  

没理解？没关系，下面就来详细解释下。  


<a name="blocking-and-non-blocking"></a>
#### 阻塞与非阻塞  
正如此前所提到的，当在请求处理程序中包括非阻塞操作时就会出问题。但是，在说这之前，我们先来看看什么是阻塞操作。  

我不想去解释“阻塞”和“非阻塞”的具体含义，我们直接来看，当在请求处理程序中加入阻塞操作时会发生什么。  

这里，我们来修改下 _start_ 请求处理程序，我们让它等待10秒以后再返回“Hello Start”。
因为，JavaScript中没有类似  _sleep()_ 这样的操作，所以这里只能够来点小Hack来模拟实现。  

让我们将 _requestHandlers.js_ 修改成如下形式：  
<pre><code>function start() {
  console.log("Request handler 'start' was called.");

  function sleep(milliSeconds) {
    var startTime = new Date().getTime();
    while (new Date().getTime() &lt; startTime + milliSeconds);
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

上述代码中，当函数 _start()_ 被调用的时候，Node.js会先等待10秒，之后才会返回“Hello Start”。当调用 _upload()_ 的时候，会和此前一样立即返回。  

（当然了，这里只是模拟休眠10秒，实际场景中，这样的阻塞操作有很多，比方说一些长时间的计算操作等。）  

接下来就让我们来看看，我们的改动带来了哪些变化。  

如往常一样，我们先要重启下服务器。为了看到效果，我们要进行一些相对复杂的操作（跟着我一起做）： 首先，打开两个浏览器窗口或者标签页。
在第一个浏览器窗口的地址栏中输入http://localhost:8888/start， 但是先不要打开它！  

在第二个浏览器窗口的地址栏中输入http://localhost:8888/upload， 同样的，先不要打开它！  

接下来，做如下操作：在第一个窗口中（“/start”）按下回车，然后快速切换到第二个窗口中（“/upload”）按下回车。  

注意，发生了什么： /start URL加载花了10秒，这和我们预期的一样。但是，/upload URL居然也花了10秒，而它在对应的请求处理程序中并没有类似于 _sleep()_ 这样的操作！  

这到底是为什么呢？原因就是 _start()_ 包含了阻塞操作。形象的说就是“它阻塞了所有其他的处理工作”。  

这显然是个问题，因为Node一向是这样来标榜自己的：“ _在node中除了代码，所有一切都是并行执行的_ ”。  

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

上述代码中，我们引入了一个新的Node.js模块， _child_process_ 。之所以用它，是为了实现一个既简单又实用的非阻塞操作： _exec()_ 。  

exec()做了什么呢？它从Node.js来执行一个shell命令。在上述例子中，我们用它来获取当前目录下所有的文件（“ls -lah”）,然后，当/start URL请求的时候将文件信息输出到浏览器中。  

上述代码是非常直观的： 创建了一个新的变量 _content_ （初始值为“empty”），执行“ls -lah”命令，将结果赋值给content，最后将content返回。  

和往常一样，我们启动服务器，然后访问“ http://localhost:8888/start ” 。  

之后会载入一个漂亮的web页面，其内容为“empty”。怎么回事？  

这个时候，你可能大致已经猜到了，exec()在非阻塞这块发挥了神奇的功效。它其实是个很好的东西，有了它，我们可以执行非常耗时的shell操作而无需迫使我们的应用停下来等待该操作。  

（如果想要证明这一点，可以将“ls -lah”换成比如“find /”这样更耗时的操作来效果）。  

然而，针对浏览器显示的结果来看，我们并不满意我们的非阻塞操作，对吧？  

好，接下来，我们来修正这个问题。在这过程中，让我们先来看看为什么当前的这种方式不起作用。  

问题就在于，为了进行非阻塞工作，exec()使用了回调函数。  

在我们的例子中，该回调函数就是作为第二个参数传递给exec()的匿名函数：  
<pre><code>function (error, stdout, stderr) {
  content = stdout;
}</code></pre>

现在就到了问题根源所在了： 我们的代码是同步执行的，这就意味着在调用exec()之后，Node.js会立即执行 return content;。在这个时候，_content_ 仍然是“empty”，因为传递给exec()的回调函数还未执行到 —— 因为exec()的操作是异步的。  

我们这里“ls -lah”的操作其实是非常快的（除非当前目录下有上百万个文件）。这也是为什么回调函数也会很快的执行到 —— 不过，不管怎么说它还是异步的。  

为了让效果更加明显，我们想象一个更耗时的命令： “find /”，它在我机器上需要执行1分钟左右的时间，
然而，尽管在请求处理程序中，我把“ls -lah”换成“find /”，当打开/start URL的时候，依然能够立即获得HTTP响应 —— 很明显，当 _exec()_ 在后台执行的时候，Node.js自身会继续执行后面的代码。并且我们这里假设传递给 _exec()_ 的回调函数，只会在“find /”命令执行完成之后才会被调用。  

那究竟我们要如何才能实现将当前目录下的文件列表显示给用户呢？  

好，了解了这种不好的实现方式之后，我们接下来来介绍如何以正确的方式让请求处理程序对浏览器请求作出响应。  


<a name="responding-request-handlers-with-non-blocking-operations"></a>
#### 以非阻塞操作进行请求响应  
我刚刚提到了这样一个短语 —— “正确的方式”。而事实上通常“正确的方式”一般都不简单。  

不过，用Node.js就有这样一种实现方案： 函数传递。下面就让我们来具体看看如何实现。  

到目前为止，我们的应用已经可以通过应用各层之间传递值的方式（请求处理程序 -> 请求路由 -> 服务器）将请求处理程序返回的内容（请求处理程序最终要显示给用户的内容）传递给HTTP服务器。  

现在我们采用如下这种新的实现方式：相对采用将内容传递给服务器的方式，我们这次采用将服务器“传递”给内容的方式。
从实践角度来说，就是将 _response_ 对象（从服务器的回调函数 _onRequest()_ 获取）通过请求路由传递给请求处理程序。
随后，处理程序就可以采用该对象上的函数来对请求作出响应。  

原理就是如此，接下来让我们来一步步实现这种方案。  

先从 _server.js_ 开始：  
<pre><code>var http = require("http");
var url = require("url");

function start(route, handle) {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");

    route(handle, pathname, response);
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;</code></pre>

相对此前从 _route()_ 函数获取返回值的做法，这次我们将response对象作为第三个参数传递给route()函数，并且，我们将onRequest()处理程序中所有有关response的函数调用都移除，因为我们希望这部分工作让route()函数来完成。  

下面就来看看我们的 _router.js_ ：  
<pre><code>function route(handle, pathname, response) {
  console.log("About to route a request for " + pathname);
  if (typeof handle[pathname] === 'function') {
    handle[pathname](response);
  } else {
    console.log("No request handler found for " + pathname);
    response.writeHead(404, {"Content-Type": "text/plain"});
    response.write("404 Not found");
    response.end();
  }
}

exports.route = route;</code></pre>

同样的模式：相对此前从请求处理程序中获取返回值，这次取而代之的是直接传递response对象。  

如果没有对应的请求处理器处理，我们就直接返回“404”错误。  

最后，我们将requestHandler.js修改为如下形式：  
<pre><code>var exec = require("child_process").exec;

function start(response) {
  console.log("Request handler 'start' was called.");

  exec("ls -lah", function (error, stdout, stderr) {
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write(stdout);
    response.end();
  });
}

function upload(response) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Hello Upload");
  response.end();
}

exports.start = start;
exports.upload = upload;</code></pre>

我们的处理程序函数需要接收response参数，为了对请求作出直接的响应。  

_start_ 处理程序在exec()的匿名回调函数中做请求响应的操作，而 _upload_ 处理程序仍然是简单的回复“Hello World”，只是这次是使用response对象而已。  

这时再次我们启动应用（node index.js），一切都会工作的很好。  

如果想要证明/start处理程序中耗时的操作不会阻塞对/upload请求作出立即响应的话，可以将 _requestHandlers.js_ 修改为如下形式：  
<pre><code>var exec = require("child_process").exec;

function start(response) {
  console.log("Request handler 'start' was called.");

  exec("find /",
    { timeout: 10000, maxBuffer: 20000*1024 },
    function (error, stdout, stderr) {
      response.writeHead(200, {"Content-Type": "text/plain"});
      response.write(stdout);
      response.end();
    });
}

function upload(response) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Hello Upload");
  response.end();
}

exports.start = start;
exports.upload = upload;</code></pre>

这样一来，当请求http://localhost:8888/start 的时候，会花10秒钟的时间才载入，而当请求http://localhost:8888/upload 的时候，会立即响应，纵然这个时候/start响应还在处理中。  


<a name="serving-something-userful"></a>
### 更有用的场景  
到目前为止，我们做的已经很好了，但是，我们的应用没有实际用途。  

服务器，请求路由以及请求处理程序都已经完成了，下面让我们按照此前的用例给网站添加交互： 用户选择一个文件，上传该文件，然后在浏览器中看到上传的文件。
为了保持简单，我们假设用户只会上传图片，然后我们应用将该图片显示到浏览器中。  

好，下面就一步步来实现，鉴于此前已经对JavaScript原理性技术性的内容做过大量介绍了，这次我们加快点速度。  

要实现该功能，分为如下两步： 首先，让我们来看看如何处理POST请求（非文件上传），之后，我们使用Node.js的一个用于文件上传的外部模块。之所以采用这种实现方式有两个理由。  

第一，尽管在Node.js中处理基础的POST请求相对比较简单，但在这过程中还是能学到很多。  
第二，用Node.js来处理文件上传（multipart POST请求）是比较复杂的，它不在本书的范畴，但，如何使用外部模块却是在本书涉猎内容之内。  


<a name="handling-post-requests"></a>
#### 处理POST请求  
考虑这样一个简单的例子：我们显示一个文本区（textarea）供用户输入内容，然后通过POST请求提交给服务器。最后，服务器接受到请求，通过处理程序将输入的内容展示到浏览器中。  

/start请求处理程序用于生成带文本区的表单，因此，我们将 _requestHandlers.js_ 修改为如下形式：  
<pre><code>function start(response) {
  console.log("Request handler 'start' was called.");

  var body = '&lt;html&gt;'+
    '&lt;head&gt;'+
    '&lt;meta http-equiv="Content-Type" content="text/html; '+
    'charset=UTF-8" /&gt;'+
    '&lt;/head&gt;'+
    '&lt;body&gt;'+
    '&lt;form action="/upload" method="post"&gt;'+
    '&lt;textarea name="text" rows="20" cols="60"&gt;&lt;/textarea&gt;'+
    '&lt;input type="submit" value="Submit text" /&gt;'+
    '&lt;/form&gt;'+
    '&lt;/body&gt;'+
    '&lt;/html&gt;';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}

function upload(response) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Hello Upload");
  response.end();
}

exports.start = start;
exports.upload = upload;</code></pre>

好了，现在我们的应用已经很完善了，都可以获得威比奖（Webby Awards）了，哈哈。（译者注：威比奖是由国际数字艺术与科学学院主办的评选全球最佳网站的奖项，具体参见[详细说明](http://zh.wikipedia.org/wiki/%E5%A8%81%E6%AF%94%E5%A5%96)）
通过在浏览器中访问 http://localhost:8888/start 就可以看到简单的表单了，要记得重启服务器哦！  

你可能会说：这种直接将视觉元素放在请求处理程序中的方式太丑陋了。说的没错，但是，我并不想在本书中介绍诸如MVC之类的模式，因为这对于你了解JavaScript或者Node.js环境来说没多大关系。  

余下的篇幅，我们来探讨一个更有趣的问题： 当用户提交表单时，触发/upload请求处理程序处理POST请求的问题。  

现在，我们已经是新手中的专家了，很自然会想到采用异步回调来实现非阻塞地处理POST请求的数据。  

这里采用非阻塞方式处理是明智的，因为POST请求一般都比较“重” —— 用户可能会输入大量的内容。用阻塞的方式处理大数据量的请求必然会导致用户操作的阻塞。  

为了使整个过程非阻塞，Node.js会将POST数据拆分成很多小的数据块，然后通过触发特定的事件，将这些小数据块传递给回调函数。这里的特定的事件有 _data_ 事件（表示新的小数据块到达了）以及 _end_ 事件（表示所有的数据都已经接收完毕）。  

我们需要告诉Node.js当这些事件触发的时候，回调哪些函数。怎么告诉呢？ 我们通过在 _request_ 对象上注册 _监听器（listener）_ 来实现。这里的request对象是每次接收到HTTP请求时候，都会把该对象传递给 _onRequest_ 回调函数。  

如下所示：  
<pre><code>request.addListener("data", function(chunk) {
  // 当新的数据块到达的时候会触发
});

request.addListener("end", function() {
  // 所有数据都接收完毕后会触发
});</code></pre>

问题来了，这部分逻辑写在哪里呢？ 我们现在只是在服务器中获取到了 _request_ 对象 —— 我们并没有像之前 _response_ 对象那样，把 _request_ 对象传递给请求路由和请求处理程序。  

在我看来，获取所有来自请求的数据，然后将这些数据给应用层处理，应该是HTTP服务器要做的事情。因此，我建议，我们直接在服务器中处理POST数据，然后将最终的数据传递给请求路由和请求处理器，让他们来进行进一步的处理。  

因此，实现思路就是： 将 _data_ 和 _end_ 事件的回调函数直接放在服务器中，在 _data_ 事件回调中收集所有的POST数据，当接收到所有数据，触发 _end_ 事件后，其回调函数调用请求路由，并将数据传递给它，然后，请求路由再将该数据传递给请求处理程序。  

还等什么，马上来实现。先从 _server.js_ 开始：  
<pre><code>var http = require("http");
var url = require("url");

function start(route, handle) {
  function onRequest(request, response) {
    var postData = "";
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");

    request.setEncoding("utf8");

    request.addListener("data", function(postDataChunk) {
      postData += postDataChunk;
      console.log("Received POST data chunk '"+
      postDataChunk + "'.");
    });

    request.addListener("end", function() {
      route(handle, pathname, response, postData);
    });

  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;</code></pre>

上述代码做了三件事情： 首先，我们设置了接收数据的编码格式为UTF-8，然后注册了“data”事件的监听器，
用于收集每次接收到的新数据块，并将其赋值给 _postData_ 变量，
最后，我们将请求路由的调用移到 _end_ 事件处理程序中，以确保它只会当所有数据接收完毕后才触发，并且只触发一次。
我们同时还把POST数据传递给请求路由，因为这些数据，请求处理程序会用到。  

上述代码在每个数据块到达的时候输出了日志，这对于最终生产环境来说，是很不好的（数据量可能会很大，还记得吧？），但是，在开发阶段是很有用的，有助于让我们看到发生了什么。  

我建议可以尝试下，尝试着去输入一小段文本，以及大段内容，当大段内容的时候，就会发现 _data_ 事件会触发多次。  

再来点酷的。我们接下来在/upload页面，展示用户输入的内容。要实现该功能，我们需要将 _postData_ 传递给请求处理程序，修改 _router.js_ 为如下形式：  
<pre><code>function route(handle, pathname, response, postData) {
  console.log("About to route a request for " + pathname);
  if (typeof handle[pathname] === 'function') {
    handle[pathname](response, postData);
  } else {
    console.log("No request handler found for " + pathname);
    response.writeHead(404, {"Content-Type": "text/plain"});
    response.write("404 Not found");
    response.end();
  }
}

exports.route = route;</code></pre>

然后，在 _requestHandlers.js_ 中，我们将数据包含在对 _upload_ 请求的响应中：  
<pre><code>function start(response, postData) {
  console.log("Request handler 'start' was called.");

  var body = '&lt;html&gt;'+
    '&lt;head&gt;'+
    '&lt;meta http-equiv="Content-Type" content="text/html; '+
    'charset=UTF-8" /&gt;'+
    '&lt;/head&gt;'+
    '&lt;body&gt;'+
    '&lt;form action="/upload" method="post"&gt;'+
    '&lt;textarea name="text" rows="20" cols="60"&gt;&lt;/textarea&gt;'+
    '&lt;input type="submit" value="Submit text" /&gt;'+
    '&lt;/form&gt;'+
    '&lt;/body&gt;'+
    '&lt;/html&gt;';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}

function upload(response, postData) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("You've sent: " + postData);
  response.end();
}

exports.start = start;
exports.upload = upload;</code></pre>

好了，我们现在可以接收POST数据并在请求处理程序中处理该数据了。  

我们最后要做的是： 当前我们是把请求的整个消息体传递给了请求路由和请求处理程序。我们应该只把POST数据中，我们感兴趣的部分传递给请求路由和请求处理程序。
在我们这个例子中，我们感兴趣的其实只是 _text_ 字段。  

我们可以使用此前介绍过的 _querystring_ 模块来实现：  
<pre><code>var querystring = require("querystring");

function start(response, postData) {
  console.log("Request handler 'start' was called.");

  var body = '&lt;html&gt;'+
    '&lt;head&gt;'+
    '&lt;meta http-equiv="Content-Type" content="text/html; '+
    'charset=UTF-8" /&gt;'+
    '&lt;/head&gt;'+
    '&lt;body&gt;'+
    '&lt;form action="/upload" method="post"&gt;'+
    '&lt;textarea name="text" rows="20" cols="60"&gt;&lt;/textarea&gt;'+
    '&lt;input type="submit" value="Submit text" /&gt;'+
    '&lt;/form&gt;'+
    '&lt;/body&gt;'+
    '&lt;/html&gt;';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}

function upload(response, postData) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("You've sent the text: "+
  querystring.parse(postData).text);
  response.end();
}

exports.start = start;
exports.upload = upload;</code></pre>

好了，以上就是关于处理POST数据的全部内容。  


<a name="handling-file-uploads"></a>
#### 处理文件上传  
最后，我们来实现我们最终的用例：允许用户上传图片，并将该图片在浏览器中显示出来。  

回到90年代，这个用例完全可以满足用于IPO的商业模型了，如今，我们通过它能学到这样两件事情： 如何安装外部Node.js模块，以及如何将它们应用到我们的应用中。  

这里我们要用到的外部模块是 Felix Geisendörfer开发的 _node-formidable_ 模块。它对解析上传的文件数据做了很好的抽象。
其实说白了，处理文件上传“就是”处理POST数据 —— 但是，麻烦的是在具体的处理细节，所以，这里采用现成的方案更合适点。  

使用该模块，首先需要安装该模块。Node.js有它自己的包管理器，叫 _NPM_ 。 它可以让安装Node.js的外部模块变得非常方便。通过如下一条命令就可以完成该模块的安装：  
<pre><code>npm install formidable</code></pre>

如果终端输出如下内容：  
<pre><code>npm info build Success: formidable@1.0.2
npm ok</code></pre>

就说明模块已经安装成功了。  

现在我们就可以用 _formidable_ 模块了 —— 使用外部模块与内部模块类似，用require语句将其引入即可：  
<pre><code>var formidable = require("formidable");</code></pre>

这里该模块做的就是将通过HTTP POST请求提交的表单，在Node.js中可以被解析。我们要做的就是创建一个新的 _IncomingForm_ ， 它是对提交表单的抽象表示，之后，就可以用它解析request对象，获取表单中需要的数据字段。  

node-formidable官方的例子展示了这两部分是如何融合在一起工作的：  
<pre><code>var formidable = require('formidable'),
    http = require('http'),
    sys = require('sys');

http.createServer(function(req, res) {
  if (req.url == '/upload' && req.method.toLowerCase() == 'post') {
    // 解析文件上传
    var form = new formidable.IncomingForm();
    form.parse(req, function(err, fields, files) {
      res.writeHead(200, {'content-type': 'text/plain'});
      res.write('received upload:\n\n');
      res.end(sys.inspect({fields: fields, files: files}));
    });
    return;
  }

  // 显示一个文件上传表单
  res.writeHead(200, {'content-type': 'text/html'});
  res.end(
    '&lt;form action="/upload" enctype="multipart/form-data" '+
    'method="post"&gt;'+
    '&lt;input type="text" name="title"&gt;&lt;br&gt;'+
    '&lt;input type="file" name="upload" multiple="multiple"&gt;&lt;br&gt;'+
    '&lt;input type="submit" value="Upload"&gt;'+
    '&lt;/form&gt;'
  );
}).listen(8888);</code></pre>

如果我们将上述代码，保存到一个文件中，并通过node来执行，就可以进行简单的表单提交了，包括文件上传。然后，可以看到通过调用form.parse传递给回调函数的 _files_ 对象的内容，如下所示：  
<pre><code>received upload:

{ fields: { title: 'Hello World' },
  files:
   { upload:
      { size: 1558,
        path: '/tmp/1c747974a27a6292743669e91f29350b',
        name: 'us-flag.png',
        type: 'image/png',
        lastModifiedDate: Tue, 21 Jun 2011 07:02:41 GMT,
        _writeStream: [Object],
        length: [Getter],
        filename: [Getter],
        mime: [Getter] } } }</code></pre>
        
为了实现我们的功能，我们需要将上述代码应用到我们的应用中，另外，我们还要考虑如何将上传文件的内容（保存在/tmp目录中）显示到浏览器中。  

我们先来解决后面那个问题： 对于保存在本地硬盘中的文件，如何才能在浏览器中看到呢？  

显然，我们需要将该文件读取到我们的服务器中，使用一个叫 _fs_ 的模块。  

我们来添加/show URL的请求处理程序，该处理程序直接硬编码将文件 /tmp/test.png内容展示到浏览器中。当然了，首先需要将该图片保存到这个位置才行。  

将 _requestHandlers.js_ 修改为如下形式：  
<pre><code>var querystring = require("querystring"),
    fs = require("fs");

function start(response, postData) {
  console.log("Request handler 'start' was called.");

  var body = '&lt;html&gt;'+
    '&lt;head&gt;'+
    '&lt;meta http-equiv="Content-Type" '+
    'content="text/html; charset=UTF-8" /&gt;'+
    '&lt;/head&gt;'+
    '&lt;body&gt;'+
    '&lt;form action="/upload" method="post"&gt;'+
    '&lt;textarea name="text" rows="20" cols="60"&gt;&lt;/textarea&gt;'+
    '&lt;input type="submit" value="Submit text" /&gt;'+
    '&lt;/form&gt;'+
    '&lt;/body&gt;'+
    '&lt;/html&gt;';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}

function upload(response, postData) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("You've sent the text: "+
  querystring.parse(postData).text);
  response.end();
}

function show(response, postData) {
  console.log("Request handler 'show' was called.");
  fs.readFile("/tmp/test.png", "binary", function(error, file) {
    if(error) {
      response.writeHead(500, {"Content-Type": "text/plain"});
      response.write(error + "\n");
      response.end();
    } else {
      response.writeHead(200, {"Content-Type": "image/png"});
      response.write(file, "binary");
      response.end();
    }
  });
}

exports.start = start;
exports.upload = upload;
exports.show = show;</code></pre>

我们还需要将这新的请求处理程序，添加到index.js中的路由映射表中：  
<pre><code>var server = require("./server");
var router = require("./router");
var requestHandlers = require("./requestHandlers");

var handle = {}
handle["/"] = requestHandlers.start;
handle["/start"] = requestHandlers.start;
handle["/upload"] = requestHandlers.upload;
handle["/show"] = requestHandlers.show;

server.start(router.route, handle);</code></pre>

重启服务器之后，通过访问 http://localhost:8888/show，就可以看到保存在 /tmp/test.png 的图片了。  

好，最后我们要的就是：  

*  在/start 表单中添加一个文件上传元素  
*  将node-formidable整合到我们的 _upload_ 请求处理程序中，用于将上传的图片保存到 /tmp/test.png  
*  将上传的图片内嵌到/upload URL输出的HTML中  

第一项很简单。只需要在HTML表单中，添加一个 _multipart/form-data_ 的编码类型，移除此前的文本区，添加一个文件上传组件，并将提交按钮的文案改为“Upload file”即可。
如下 _requestHandler.js_ 所示：  
<pre><code>var querystring = require("querystring"),
    fs = require("fs");

function start(response, postData) {
  console.log("Request handler 'start' was called.");

  var body = '&lt;html&gt;'+
    '&lt;head&gt;'+
    '&lt;meta http-equiv="Content-Type" '+
    'content="text/html; charset=UTF-8" /&gt;'+
    '&lt;/head&gt;'+
    '&lt;body&gt;'+
    '&lt;form action="/upload" enctype="multipart/form-data" '+
    'method="post"&gt;'+
    '&lt;input type="file" name="upload"&gt;'+
    '&lt;input type="submit" value="Upload file" /&gt;'+
    '&lt;/form&gt;'+
    '&lt;/body&gt;'+
    '&lt;/html&gt;';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}

function upload(response, postData) {
  console.log("Request handler 'upload' was called.");
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("You've sent the text: "+
  querystring.parse(postData).text);
  response.end();
}

function show(response, postData) {
  console.log("Request handler 'show' was called.");
  fs.readFile("/tmp/test.png", "binary", function(error, file) {
    if(error) {
      response.writeHead(500, {"Content-Type": "text/plain"});
      response.write(error + "\n");
      response.end();
    } else {
      response.writeHead(200, {"Content-Type": "image/png"});
      response.write(file, "binary");
      response.end();
    }
  });
}

exports.start = start;
exports.upload = upload;
exports.show = show;</code></pre>

很好。下一步相对比较复杂。这里有这样一个问题： 我们需要在 _upload_ 处理程序中对上传的文件进行处理，这样的话，我们就需要将 _request_ 对象传递给node-formidable的form.parse函数。  

但是，我们有的只是 _response_ 对象和 _postData_ 数组。看样子，我们只能不得不将 _request_ 对象从服务器开始一路通过请求路由，再传递给请求处理程序。
或许还有更好的方案，但是，不管怎么说，目前这样做可以满足我们的需求。  

到这里，我们可以将 _postData_ 从服务器以及请求处理程序中移除了 —— 一方面，对于我们处理文件上传来说已经不需要了，另外一方面，它甚至可能会引发这样一个问题：
我们已经“消耗”了 _request_ 对象中的数据，这意味着，对于 _form.parse_ 来说，当它想要获取数据的时候就什么也获取不到了。（因为Node.js不会对数据做缓存）  

我们从 _server.js_ 开始 —— 移除对postData的处理以及 _request.setEncoding_ （这部分node-formidable自身会处理），转而采用将 _request_ 对象传递给请求路由的方式：  
<pre><code>var http = require("http");
var url = require("url");

function start(route, handle) {
  function onRequest(request, response) {
    var pathname = url.parse(request.url).pathname;
    console.log("Request for " + pathname + " received.");
    route(handle, pathname, response, request);
  }

  http.createServer(onRequest).listen(8888);
  console.log("Server has started.");
}

exports.start = start;
</code></pre>

接下来是 _router.js_ —— 我们不再需要传递 _postData_ 了，这次要传递 _request_ 对象：  
<pre><code>function route(handle, pathname, response, request) {
  console.log("About to route a request for " + pathname);
  if (typeof handle[pathname] === 'function') {
    handle[pathname](response, request);
  } else {
    console.log("No request handler found for " + pathname);
    response.writeHead(404, {"Content-Type": "text/html"});
    response.write("404 Not found");
    response.end();
  }
}

exports.route = route;</code></pre>

现在，_request_ 对象就可以在我们的 _upload_ 请求处理程序中使用了。node-formidable会处理将上传的文件保存到本地 /tmp目录中，而我们需要做的是确保该文件保存成 /tmp/test.png。
没错，我们保持简单，并假设只允许上传PNG图片。  

这里采用 _fs.renameSync(path1,path2)_ 来实现。要注意的是，正如其名，该方法是同步执行的，也就是说，如果该重命名的操作很耗时的话会阻塞。
这块我们先不考虑。  

接下来，我们把处理文件上传以及重命名的操作放到一起，如下 _requestHandlers.js_ 所示：  
<pre><code>var querystring = require("querystring"),
    fs = require("fs"),
    formidable = require("formidable");

function start(response) {
  console.log("Request handler 'start' was called.");

  var body = '&lt;html&gt;'+
    '&lt;head&gt;'+
    '&lt;meta http-equiv="Content-Type" content="text/html; '+
    'charset=UTF-8" /&gt;'+
    '&lt;/head&gt;'+
    '&lt;body&gt;'+
    '&lt;form action="/upload" enctype="multipart/form-data" '+
    'method="post"&gt;'+
    '&lt;input type="file" name="upload" multiple="multiple"&gt;'+
    '&lt;input type="submit" value="Upload file" /&gt;'+
    '&lt;/form&gt;'+
    '&lt;/body&gt;'+
    '&lt;/html&gt;';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
}

function upload(response, request) {
  console.log("Request handler 'upload' was called.");

  var form = new formidable.IncomingForm();
  console.log("about to parse");
  form.parse(request, function(error, fields, files) {
    console.log("parsing done");
    fs.renameSync(files.upload.path, "/tmp/test.png");
    response.writeHead(200, {"Content-Type": "text/html"});
    response.write("received image:&lt;br/&gt;");
    response.write("&lt;img src='/show' /&gt;");
    response.end();
  });
}

function show(response) {
  console.log("Request handler 'show' was called.");
  fs.readFile("/tmp/test.png", "binary", function(error, file) {
    if(error) {
      response.writeHead(500, {"Content-Type": "text/plain"});
      response.write(error + "\n");
      response.end();
    } else {
      response.writeHead(200, {"Content-Type": "image/png"});
      response.write(file, "binary");
      response.end();
    }
  });
}

exports.start = start;
exports.upload = upload;
exports.show = show;</code></pre>

好了，重启服务器，我们应用所有的功能就可以用了。选择一张本地图片，将其上传到服务器，然后浏览器就会显示该图片。  


<a name="conclusion-and-outlokk"></a>
## 总结与展望  
恭喜，我们的任务已经完成了！我们开发完了一个Node.js的web应用，应用虽小，但却“五脏俱全”。 期间，我们介绍了很多技术点：服务端JavaScript、函数式编程、阻塞与非阻塞、回调、事件、内部和外部模块等等。  

当然了，还有许多本书没有介绍到的： 如何操作数据库、如何进行单元测试、如何开发Node.js的外部模块以及一些简单的诸如如何获取GET请求之类的方法。  

但本书毕竟只是一本给初学者的教程 —— 不可能覆盖到所有的内容。  

幸运的是，Node.js社区非常活跃（作个不恰当的比喻就是犹如一群有多动症小孩子在一起，能不活跃吗？），这意味着，有许多关于Node.js的资源，有什么问题都可以向社区寻求解答。
其中[Node.js社区的wiki](https://github.com/joyent/node/wiki)以及[NodeClould](http://www.nodecloud.org/)就是最好的资源。  




