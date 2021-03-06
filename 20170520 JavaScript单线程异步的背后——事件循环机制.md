
感觉这篇文章拖了很久，好尴尬的拖延症 

正文从这里开始～～～

对JavaScript有个很模糊的印象，它是单线程异步的。本文主要来说说JavaScript到底是怎么运行的。但在这之前，让我们先理一下这些概念（现学现卖）。

## 基本概念

### 线程与进程

进程(Process)是系统资源分配和调度的单元。一个运行着的程序就对应了一个进程。一个进程包括了运行中的程序和程序所使用到的内存和系统资源。如果是单核CPU的话，在同一时间内，有且只有一个进程在运行。但是，单核CPU也能实现多任务同时运行，比如你边听网易云音乐的每日推荐歌曲，边在网易有道云笔记上写博文。这算开了两个进程（多进程），那运行的机制就是一会儿播放一下歌，一会儿响应一下你的打字，但由于CPU切换的速度很快，你根本感觉不到，以至于你认为这两个进程是在同时运行的。进程之间是资源隔离的。

那线程(Thread)是什么？线程是进程下的执行者，一个进程至少会开启一个线程（主线程），也可以开启多个线程。比如网易云音乐一边播放音频，一边显示歌词。多进程的运行其实也就是通过进程中的线程来执行的。一个进程下的线程是共享资源的。当多个线程同时操作同一个资源的时候，就出现资源争抢的问题。这又是另外一个问题了。

### 并行与并发 

并行(Parallelism)是指程序的运行状态，在同一个时间内有几件事情并行在处理。由于一个线程在同一时间只能处理一件事情，所以并行需要多个线程在同一时间执行多件事情。

而并发(Concurrency)是指程序的设计结构，在同一时间内多件事情能被交替地处理。重点是，在某个时间内只有一件事情在执行。比如单核CPU能实现多任务运行的过程就是并发的。

### 同步与异步

同步异步是指程序的行为。同步(Synchronous)是程序发出调用的时候，一直等待直到返回结果，没有结果之前不会返回。也就是说，同步是调用者主动等待调用过程。

异步(Asynchronous)是发出调用之后，马上返回，但是不会马上返回结果。调用者不必主动等待，当被调用者得到结果之后会主动通知调用者。

举个例子，去奶茶店买饮料。同步就是，一个顾客说出需求（请求），然后一直等着服务员做好饮料，顾客拿到自己点的饮料之后才离开；然后下一个顾客继续重复上述过程。异步就是，顾客先排队点单，点完之后拿着单子在一边，等服务员做好之后会叫号，叫到你了你去拿就好。

所以线程跟同步异步没有直接的关系，单线程也是可以实现异步的。至于实现的方式，下面会具体说到。

### 阻塞与非阻塞

阻塞与非阻塞是指等待状态。阻塞(Blocking)是指调用在等待的过程中线程被“挂起”（CPU资源被分配到其他地方去了）。

非阻塞(Non-blocking)是指等待的过程CPU资源还在该线程中，线程还能做其他的事情。

以刚才排队买饮料的例子，阻塞就是你在等待的时候什么事情也做不了，而非阻塞是你在等待的时候可以管自己先做其他的事情。

所以，同步可以阻塞也可以非阻塞，异步可以阻塞也可以非阻塞。


## 单线程的JS

大概理清楚上述概念之后呢，就知道单线程和异步是没有矛盾的。那JS是如何执行的呢？JS其实就是一门语言，说是单线程还是多线程得结合具体运行环境。JS的运行通常是在浏览器中进行的，具体由JS引擎去解析和运行。下面我们来具体了解一下浏览器。

### 浏览器

目前最为流行的浏览器为：Chrome，IE，Safari，FireFox，Opera。浏览器的内核是多线程的。一个浏览器通常由以下几个常驻的线程：

+ 渲染引擎线程：顾名思义，该线程负责页面的渲染
+ JS引擎线程：负责JS的解析和执行
+ 定时触发器线程：处理定时事件，比如setTimeout, setInterval
+ 事件触发线程：处理DOM事件
+ 异步http请求线程：处理http请求

需要注意的是，渲染线程和JS引擎线程是不能同时进行的。渲染线程在执行任务的时候，JS引擎线程会被挂起。因为JS可以操作DOM，若在渲染中JS处理了DOM，浏览器可能就不知所措了。


### JS引擎

通常讲到浏览器的时候，我们会说到两个引擎：渲染引擎和JS引擎。渲染引擎就是如何渲染页面，Chrome／Safari／Opera用的是Webkit引擎，IE用的是Trident引擎，FireFox用的是Gecko引擎。不同的引擎对同一个样式的实现不一致，就导致了经常被人诟病的浏览器样式兼容性问题。这里我们不做具体讨论。

JS引擎可以说是JS虚拟机，负责JS代码的解析和执行。通常包括以下几个步骤：

+ 词法分析：将源代码分解为有意义的分词
+ 语法分析：用语法分析器将分词解析成语法树
+ 代码生成：生成机器能运行的代码
+ 代码执行

不同浏览器的JS引擎也各不相同，Chrome用的是V8，FireFox用的是SpiderMonkey，Safari用的是JavaScriptCore，IE用的是Chakra。

回到JS是单线程这句话，本质上来说，是浏览器在运行时只开启了一个JS引擎线程来解析和执行JS。那为什么只有一个引擎呢？如果同时有两个线程去操作DOM，浏览器是不是又要不知所措了？！

## JS运行机制

说了这么多之后，终于我们要说到JS的整个运行过程了。

### 同步执行

我们先看下JS同步执行过程是怎么做到的？这就涉及到了一个很重要的概念——执行上下文。我的一篇译文[深入学习JavaScript闭包](http://mp.weixin.qq.com/s?__biz=MjM5MTA1MjAxMQ==&mid=2651226038&idx=1&sn=636aa6089bb12c9431262129c42ea958&chksm=bd49a6328a3e2f24a872fad076cf22ed39a2f11a64864659eaacede0714025724e97af32e789&mpshare=1&scene=1&srcid=0303IevkR8NiPmcgvN69bLsq#rd)很详细地说明了这个概念。 

执行上下文记录了代码运行时的环境，当前运行状态下有且有一个执行上下文在起作用。那执行上下文到底记录了什么呢？大概有词法环境，变量环境等。举个简单的例子：

    var x = 10;
    
    function foo(){
        var y=20;
        function bar(){
            var z=15; 
        }
        bar();
    }
    
    foo();

代码运行，首先进入全局上下文。然后执行`foo()`的时候，就进入了foo上下文，当然此时全局上下文还在。当执行 `bar()`的时候，又进入了bar上下文。执行完毕`bar()`，回到foo上下文。执行完`foo()`，又回到全局上下文。所以，执行过程执行上下文会形成一个调用栈(Call stack)，先进后出。
    
    // 进栈
                                                              3 bar Context
        =>                      =>   2 foo Context     =>     2 foo Context
            1 global Context         1 global Context         1 global Context
    
    // 出栈
    3 bar Context
    2 foo Context     =>   2 foo Context     =>                        =>
    1 global Context       1 global Context         1 global Context

在JS执行过程中，有且仅有一个执行上下文在起作用。因为JS是单线程的，一次只能做一件事。

以上的过程都是同步执行的。

### 异步执行——事件循环

我们回顾一下JS中自带了哪些原生的异步事件：

+ setTimeout
+ setInterval
+ 事件监听
+ Ajax请求
+ etc..

JS异步的效果得益于浏览器的执行环境。实际上，浏览器又开了线程来处理这些BOM事件。举例：

    function foo(){
        console.log(1);
    }
    
    function bar(){
        console.log(2);
    }
    
    foo();
    
    setTimeout(function cb(){
        console.log(3);
    });
    
    bar();
    
    

按照上一节的分析，首先进入全局上下文，运行至`foo()`，进入了foo上下文环境；执行`console.log(1)`，控制台输出1；foo上下文环境出栈，运行至`setTimeout`，交给**浏览器的定时处理线程**；运行至`bar()`，进入了bar上下文环境；执行`console.log(2)`，控制台输出2；foo上下文环境出栈；等到**浏览器线程**执行完`setTimeout`，返回`cb()`回调函数至当前**任务队列**；当发现执行栈为空时，**浏览器的JS引擎**会执行一次循环，将事件队列的队首出队至JS执行栈中；执行`cb()`，进入cb上下文环境；执行`console.log(3)`，控制台输出3；事件队列为空，全局上下文出栈。

以上就是JS引擎的事件循环机制，是实现异步的一种机制。主要涉及到**浏览器线程**，**任务队列**以及**JS引擎**。所以，我们可以看出，JS的异步请求，借助了而它所在的运行环境浏览器来处理并且返回结果。而且，这也解释了为什么那些回调函数的`this`指向`window`，因为这些异步的代码都是在全局上下文环境下执行的。
 
 后话：不知道自己理解对了没有，也不知道说得清楚不清楚。不当之处，欢迎指出。


参考资料：

+ [JavaScript：彻底理解同步、异步和事件循环(Event Loop)](https://segmentfault.com/a/1190000004322358)
+ [还在疑惑并发和并行？](https://laike9m.com/blog/huan-zai-yi-huo-bing-fa-he-bing-xing,61/)
+ [IMWeb社区 浏览器进程？线程？傻傻分不清楚！](http://imweb.io/topic/58e3bfa845e5c13468f567d5)
+ [Javascript是单线程的深入分析](http://www.cnblogs.com/Mainz/p/3552717.html)
+ [JavaScript单线程和浏览器事件循环简述](http://www.cnblogs.com/whitewolf/p/javascript-single-thread-and-browser-event-loop.html)
+ [AlloyTeam 【转向Javascript系列】从setTimeout说事件循环模型](http://www.alloyteam.com/2015/10/turning-to-javascript-series-from-settimeout-said-the-event-loop-model/)
+ [小胡子哥 JavaScript异步编程原理](http://www.cnblogs.com/hustskyking/p/javascript-asynchronous-programming.html)
+ [阮一峰 JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
+ [【朴灵评注】JavaScript 运行机制详解：再谈Event Loop](http://blog.csdn.net/lin_credible/article/details/40143961)
+ [Philip Roberts: Help, I’m stuck in an event-loop.](https://vimeo.com/96425312)
+ [Philip Roberts: What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)
+ [jquery中Ajax的异步和同步](http://cnn237111.blog.51cto.com/2359144/1038080)