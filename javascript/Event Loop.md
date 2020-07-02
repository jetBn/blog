# Event Loop

1. 关于javascirpt

    javascript是一门单线程的语言， 虽然HTML5提出了web-work这样多线程解决方案，但是依旧没有改变javascript是单线程的本质。
    <br>
    > 关于HMLT5的web-work
      它其实就是将一些大量的计算代码交给web work去处理运行而不冻结用户界面，就是阻塞UI的渲染，但是子线程是完全受主线程的控制的，而且不能和页面进行交互，如获取元素， alert等，但是线程之间是可以进行传递数据的。
2. javascript的事件循环 

    由于js是单线程的，就是同一个时间只能做同一个事情，那么就引出问题了，我们访问一个页面的时候，比如有很多的图片、视频等等资源，加载的时候我们肯定不是在那等的，所以就引入两种任务 同步任务与异步任务。
    
    ![Image text](https://raw.githubusercontent.com/jetBn/blog/master/assets/md_images/event_loop1.png)
    1. 同步和异步任务分别进入不同的场所分支。执行同步的任务都在主线程上执行，形成一个执行栈，而异步任务进入Event Table中注册并且返回回调函数
    2. 当这个异步任务有了运行的结果，Event Table会将这个回调函数移入Event Queue中进入等待的一种状态。
    3. 当主线程同步的任务执行完毕， 会去Event Queue中读取对应的函数，并且结束它的等待状态，让它进入主线程   行执行。
    4. 主线程不断重复着上面的3个步骤，也就是常说的Event loop事件循环。
    <br>
        > 那怎么知道主线程执行栈为空的？
        其实js引擎在monitoring process 监控进程，不会不断的检查主线程执行栈是否为空，一旦为空，就会去Event Queue那里检查是否有等待被调用的函数。

3. 任务队列解释
    <br>
    > 1. "任务队列"是一个事件的队列（也可以理解成消息队列）， IO设备完成一项任务，就在"任务队列"中添加一个事件，表示相关的异步任务可以进入"执行栈"了。主线程读取"任务队列"，就是读取里面有哪些事件 。
    >2. "任务队列"中的事件，除了IO设备的事件以为，还包括一些用户产生的事件（比如鼠标点击，页面滚动等等）。只要指定过回调函数，这些事件发生时就会进入"任务队列"，等待主线程的读取。
    >3. 所谓"回调函数"(callback), 就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。
    > 4. "任务队列"是一个先进先出的数据结构，排在前面的事件，优先被主线程读取。主线程的读取过程基本上自动的， 只要执行栈一清空，"任务队列"上的第一位事件就自动进入主线程。但是，由于存在后文提到的"定时器"功能，主线程首先要检查一下执行时间，某些时间只有到了规定的时间，才能返回主线程。
    > 5. 读取到一个异步任务，首先是将异步任务放进事件表格（Event table）中，当放进事件表格中的异步任务完成某种事情或者达成某些条件（如setTimeout事件到了，鼠标点击了， 数据文件获取到了）之后，才将这些异步任务推入事件队列（Event Queue）中，这时候的异步任务才是执行栈中空闲的时候才能读取到的异步任务。

4. 宏任务和微任务
    宏任务(macro-task)包括：setTimeout, setInterval, setImmediate, I/O, UI交互事件，可以理解是每次执行栈执行的代码就是一个宏任务
    微任务(micro-task)包括： Promises, Object.observe, MutationObserver, process.nextTick, 且process.nextTick优先级大于promise.then。可以理解是在当前 task 执行结束后立即执行的任务；
    <br>
5. 宏任务、微任务的执行顺序
    主要的来讲就是先执行同步代码，遇到异步宏任务则将异步宏任务放入宏任务队列中，遇到异步微任务则将异步微任务放入微任务的队列中，当所有的同步代码执行完毕后，再将异步微任务从队列中调出来进入主线程执行，微任务执行完毕后再将异步宏任务从队列中调取出来到主线程中执行，一直循环直至所有任务执行完毕。
    <br>

    一道面试题代码分析

    ```
    console.log('1');
    async function async1() {
        console.log('2');
        await async2();
        console.log('3');
    }
    async function async2() {
        console.log('4');
    }

    process.nextTick(function() {
        console.log('5');
    })

    setTimeout(function() {
        console.log('6');
        process.nextTick(function() {
            console.log('7');
        })
        new Promise(function(resolve) {
            console.log('8');
            resolve();
        }).then(function() {
            console.log('9')
        })
    })

    async1();

    new Promise(function(resolve) {
        console.log('10');
        resolve();
    }).then(function() {
        console.log('11');
    });
    console.log('12');
    ```
    第一轮时间循环流程： 
    * 整体script代码作为一个宏任务进入主线，遇到console.log，输出1
    * 遇到async1、async2函数声明， 声明暂时不用管
    * 遇到process.nextTick()，其回调函数被分发到微任务Event Queue中，我们记为process1
    * 遇到setTimeout， 它的回调被分发到宏任务Event Queue中。我们暂记为setTimeout1
    * 执行async1，遇到console.log，输出2
    <br>
    
    当使用async定义的函数，当它被调用时，它返回的是一个Promise对象
    而当await后面的表达式是一个Promise时，它的返回值实际上是Promise的回调reslove的参数
    <br>

    * 遇到await async2()调用，发现async2也是一个定义async的函数，所以直接执行返回4，同时返回了一个Promise。（这里返回的Promise被分配到微任务Event Queue中， 我们记为await1。await会让出线程，接下来就会跳出async1函数继续往下执行）
    * 遇到Promise, new Promise直接执行，输出10。then被分发到微任务Event Queue中。记为then1
    * 遇到console.log 输出12

    |宏任务Event Queue | 微任务Event Queue|
    |-|-|
    |setTimeout1 | process1|
    |&nbsp; | await1|
    |&nbsp; | then1|

    这是第一轮事件循环结束时候各个Event Queue的情况，此时已经输出 1 2 4 10 12
    <br>
    这时候存在process1，await1，then1三个微任务，进行依次执行
    * 执行process1输出5
    * 取到await1， 就是async1放进去的Promise，执行Promise时候发现了遇到了他的reslove函数，（这里这个reslove函数又会被放入微任务Event Queue中，记为await2，然后再次跳出async1函数继续执行下一个微任务）
    * 执行then1输出11

    |宏任务Event Queue | 微任务Event Queue|
    | - | - |
    |setTimeout1 | await2|

    到这里输出1 2 4 10 12 5 11，然后我们的第一次微任务执行完毕了 新加入了一个await2的微任务在微任务Event Queue中。
    <br>
    此时我们进行执行await2微任务，<strong>它是async1放进去Promise的reslove回调</strong>，执行它（因为async2并没有返回值， 所以这个reslove的参数是undefined），此时await定义的这个Promise已经执行完毕并且返回了结果， 所以可以继续往下执行async1函数后面的任务了，那就是console.log(3)， 输出 3
    <br>
    现在到这里我们第一轮事件循环结束了，此时，输出为 1 2 4 10 12 5 11 3
    <br>
    现在微任务执行完毕，进行第二轮的事件循环从setTimeout1宏任务开始

    * 遇到console.log，输出6
    * 遇到process.nextTick()，被分发到微任务Event Queue中去，记为process2
    * 遇到new Promise立即执行后输出8，然后then被分发到微任务Event Queue，记为then2

    |宏任务Event Queue | 微任务Event Queue|
    | - | -|
    |&nbsp; | process2|
    |&nbsp; | then2|

    上述表格为二轮事件循环结束各个Event Queue的情况，这时候的情况是存在两个微任务process2和then2

    * 执行process2， 输出7
    * 执行then2，输出9

    第二轮事件循环结束，第二轮结果为 6 8 7 9
    <br>
    所以最后的输出结果为1 2 4 10 12 5 11 3 6 8 7 9
    <br>
6. 最后
    通过题目与理论的结合更深刻的理解javascirpt中Event loop



