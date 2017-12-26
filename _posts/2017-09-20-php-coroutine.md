---
title: PHP协程
categories:
 - PHP
 - 网络编程
tags:
 - 协程
---

### [在PHP中使用协程实现多任务调度](http://www.laruence.com/2015/05/28/3038.html)

生成器提供了一种更容易的方法来实现简单的对象迭代，相比较定义类实现 Iterator 接口的方式，**性能开销和复杂性大大降低。**

生成器允许你在 foreach 代码块中写代码来迭代一组数据而不需要在内存中创建一个数组, 那会使你的内存达到上限，或者会占据可观的处理时间。相反，你可以写一个生成器函数，**就像一个普通的自定义函数一样, 和普通函数只返回一次不同的是, 生成器可以根据需要 yield 多次，以便生成需要迭代的值。**

比如，调用 range(0, 1000000) 将导致内存占用超过 100 MB。

#### 生成器为可中断的函数
要从生成器认识协程, 理解它内部是如何工作是非常重要的: 生成器是一种可中断的函数, 在它里面的yield构成了中断点.

还是看上面的例子, 调用xrange(1,1000000)的时候, xrange()函数里代码其实并没有真正地运行. 它只是返回了一个迭代器：

```php
<?php
$range = xrange(1, 1000000);
var_dump($range); // object(Generator)#1
var_dump($range instanceof Iterator); // bool(true)
```

这也解释了为什么xrange叫做迭代生成器, 因为它返回一个迭代器, 而这个迭代器实现了Iterator接口.

调用迭代器的方法一次, 其中的代码运行一次.例如, 如果你调用`$range->rewind()`, 那么`xrange()`里的代码就会运行到控制流第一次出现yield的地方. 而函数内传递给yield语句的返回值可以通过`$range->current()`获取.

为了继续执行生成器中yield后的代码, 你就需要调用`$range->next()`方法. 这将再次启动生成器, 直到下一次yield语句出现. 因此,连续调用`next()`和`current()`方法, 你就能从生成器里获得所有的值, 直到再没有yield语句出现.

对`xrange()`来说, 这种情形出现在`$i`超过`$end`时. 在这中情况下, 控制流将到达函数的终点,因此将不执行任何代码.一旦这种情况发生,`vaild()`方法将返回假, 这时迭代结束.

#### 协程
协程的支持是在迭代生成器的基础上, **增加了可以回送数据给生成器的功能**(调用者发送数据给被调用的生成器函数). 这就把生成器到调用者的单向通信转变为两者之间的双向通信.

上面的例子里演示了yield作为接受者, 接下来我们看如何同时进行接收和发送的例子：
```php
<?php
function gen() {
    $ret = (yield 'yield1');
    var_dump($ret);
    $ret = (yield 'yield2');
    var_dump($ret);
}
 
$gen = gen();
var_dump($gen->current());    // string(6) "yield1"
var_dump($gen->send('ret1')); // string(4) "ret1"   (the first var_dump in gen)
                              // string(6) "yield2" (the var_dump of the ->send() return value)
var_dump($gen->send('ret2')); // string(4) "ret2"   (again from within gen)
                              // NULL               (the return value of ->send())
```

#### 多任务协作
**多任务协作这个术语中的“协作”很好的说明了如何进行这种切换的：它要求当前正在运行的任务自动把控制传回给调度器,这样就可以运行其他任务了. 这与“抢占”多任务相反, 抢占多任务是这样的：调度器可以中断运行了一段时间的任务, 不管它喜欢还是不喜欢.** 协作多任务在Windows的早期版本(windows95)和Mac OS中有使用, 不过它们后来都切换到使用抢先多任务了. 理由相当明确：如果你依靠程序自动交出控制的话, 那么一些恶意的程序将很容易占用整个CPU, 不与其他任务共享.

现在你应当明白协程和任务调度之间的关系：yield指令提供了任务中断自身的一种方法, **然后把控制交回给任务调度器**. 因此协程可以运行多个其他任务. 更进一步来说, **yield还可以用来在任务和调度器之间进行通信**.

为了实现我们的多任务调度, 首先实现“任务” — 一个用轻量级的包装的协程函数:
```php
<?php
class Task {
    protected $taskId;
    protected $coroutine;
    protected $sendValue = null;
    protected $beforeFirstYield = true;
 
    public function __construct($taskId, Generator $coroutine) {
        $this->taskId = $taskId;
        $this->coroutine = $coroutine;
    }
 
    public function getTaskId() {
        return $this->taskId;
    }
 
    public function setSendValue($sendValue) {
        $this->sendValue = $sendValue;
    }
 
    public function run() {
        if ($this->beforeFirstYield) {
            $this->beforeFirstYield = false;
            return $this->coroutine->current();
        } else {
            $retval = $this->coroutine->send($this->sendValue);
            $this->sendValue = null;
            return $retval;
        }
    }
 
    public function isFinished() {
        return !$this->coroutine->valid();
    }
}
```

通过添加 beforeFirstYieldcondition 我们可以确定第一个yield的值能被正确返回.

```php
<?php
function gen() {
    yield 'foo';
    yield 'bar';
}
 
$gen = gen();
var_dump($gen->send('something'));
 
// 如之前提到的在send之前, 当$gen迭代器被创建的时候一个renwind()方法已经被隐式调用
// 所以实际上发生的应该类似:
//$gen->rewind();
//var_dump($gen->send('something'));
 
//这样renwind的执行将会导致第一个yield被执行, 并且忽略了他的返回值.
//真正当我们调用yield的时候, 我们得到的是第二个yield的值! 导致第一个yield的值被忽略.
//string(3) "bar"
```

调度器现在不得不比多任务循环要做稍微多点了, 然后才运行多任务：

```php
<?php
class Scheduler {
    protected $maxTaskId = 0;
    protected $taskMap = []; // taskId => task
    protected $taskQueue;
 
    public function __construct() {
        $this->taskQueue = new SplQueue();
    }
 
    public function newTask(Generator $coroutine) {
        $tid = ++$this->maxTaskId;
        $task = new Task($tid, $coroutine);
        $this->taskMap[$tid] = $task;
        $this->schedule($task);
        return $tid;
    }
 
    public function schedule(Task $task) {
        $this->taskQueue->enqueue($task);
    }
 
    public function run() {
        while (!$this->taskQueue->isEmpty()) {
            $task = $this->taskQueue->dequeue();
            $task->run();
 
            if ($task->isFinished()) {
                unset($this->taskMap[$task->getTaskId()]);
            } else {
                $this->schedule($task);
            }
        }
    }
}
```

newTask()方法（使用下一个空闲的任务id）创建一个新任务,然后把这个任务放入任务map数组里. 接着它通过把任务放入任务队列里来实现对任务的调度. 接着run()方法扫描任务队列, 运行任务.如果一个任务结束了, 那么它将从队列里删除, 否则它将在队列的末尾再次被调度.

让我们看看下面具有两个简单（没有什么意义）任务的调度器：

```php
<?php
function task1() {
    for ($i = 1; $i <= 10; ++$i) {
        echo "This is task 1 iteration $i.\n";
        yield;
    }
}
 
function task2() {
    for ($i = 1; $i <= 5; ++$i) {
        echo "This is task 2 iteration $i.\n";
        yield;
    }
}
 
$scheduler = new Scheduler;
 
$scheduler->newTask(task1());
$scheduler->newTask(task2());
 
$scheduler->run();
```

两个任务都仅仅回显一条信息,然后使用yield把控制回传给调度器.输出结果如下：

```php
This is task 1 iteration 1.
This is task 2 iteration 1.
This is task 1 iteration 2.
This is task 2 iteration 2.
This is task 1 iteration 3.
This is task 2 iteration 3.
This is task 1 iteration 4.
This is task 2 iteration 4.
This is task 1 iteration 5.
This is task 2 iteration 5.
This is task 1 iteration 6.
This is task 1 iteration 7.
This is task 1 iteration 8.
This is task 1 iteration 9.
This is task 1 iteration 10.
```

输出确实如我们所期望的：对前五个迭代来说,两个任务是交替运行的, 而在第二个任务结束后, 只有第一个任务继续运行.

![多任务调度](http://zgjian-pic.oss-cn-beijing.aliyuncs.com/coroutines_flow_think.png)

![基础版调度器](http://zgjian-pic.oss-cn-beijing.aliyuncs.com/markdown/201603200102.png)

#### 与调度器之间通信
既然调度器已经运行了, 那么我们来看下一个问题：**任务和调度器之间的通信.**

**<u>我们将使用进程用来和操作系统会话的同样的方式来通信</u>：系统调用.**

**统一控制，调度器只做调度器的工作.**

**我们需要系统调用的理由是操作系统与进程相比它处在不同的权限级别上. 因此为了执行特权级别的操作（<u>如杀死另一个进程</u>), 就不得不以某种方式把控制传回给内核, 这样内核就可以执行所说的操作了**. 再说一遍, 这种行为在内部是通过使用中断指令来实现的. 过去使用的是通用的int指令, 如今使用的是更特殊并且更快速的syscall/sysenter指令.

[SystemCall代码](http://zgjian-pic.oss-cn-beijing.aliyuncs.com/markdown/system-call.zip)

![调度通讯以及任务管理](http://zgjian-pic.oss-cn-beijing.aliyuncs.com/coroutines_flow_think2.png)

#### 非阻塞IO
Web服务器最难的部分通常是像读数据这样的套接字操作是阻塞的. 例如PHP将等待到客户端完成发送为止. 对一个Web服务器来说, 这有点不太高效. 因为服务器在一个时间点上只能处理一个连接.

**解决方案是确保在真正对套接字读写之前该套接字已经“准备就绪”.** 为了查找哪个套接字已经准备好读或者写了, 可以使用 流选择函数.

协程编写WEB服务器：[SystemCallServer代码](http://www.zgjian.cc/markdown/coroutines-server.zip)

![协程服务器](http://zgjian-pic.oss-cn-beijing.aliyuncs.com/markdown/coroutine-server.png)

![新版调度器](http://zgjian-pic.oss-cn-beijing.aliyuncs.com/markdown/201603200103.png)

#### 协程栈（代码写得更简洁）
如果你试图用我们的调度系统建立更大的系统的话, 你将很快遇到问题：我们习惯了把代码分解为更小的函数, 然后调用它们. 然而, 如果使用了协程的话, 就不能这么做了. 例如,看下面代码：

```php
<?php
function echoTimes($msg, $max) {
    for ($i = 1; $i <= $max; ++$i) {
        echo "$msg iteration $i\n";
        yield;
    }
}
 
function task() {
    echoTimes('foo', 10); // print foo ten times
    echo "---\n";
    echoTimes('bar', 5); // print bar five times
    yield; // force it to be a coroutine
}
 
$scheduler = new Scheduler;
$scheduler->newTask(task());
$scheduler->run();
```

这段代码试图把重复循环“输出n次“的代码嵌入到一个独立的协程里,然后从主任务里调用它. 然而它无法运行. 正如在这篇文章的开始所提到的, 调用生成器（或者协程）将没有真正地做任何事情, 它仅仅返回一个对象.这 也出现在上面的例子里:echoTimes调用除了放回一个（无用的）协程对象外不做任何事情.

#### 错误处理
throw() 方法接受一个 Exception, 并将其抛出到协程的当前悬挂点, 看看下面代码：

```php
<?php
function gen() {
    echo "Foo\n";
    try {
        yield;
    } catch (Exception $e) {
        echo "Exception: {$e->getMessage()}\n";
    }
    echo "Bar\n";
}
 
$gen = gen();
$gen->rewind();                     // echos "Foo"
$gen->throw(new Exception('Test')); // echos "Exception: Test"
                                    // and "Bar"
```

这非常好, 有没有? 因为我们现在可以使用系统调用以及**子协程调用异常抛出**.

子协程抛出异常，父协程捕异常进行处理，避免PHP抛出Fatal Exception导致整个线程挂掉。

#### 结语
在这篇文章里,我使用多任务协作构建了一个任务调度器, 其中包括执行“系统调用”, **做非阻塞操作和处理错误. 所有这些里真正很酷的事情是任务的结果代码看起来完全同步**, 甚至任务正在执行大量的异步操作的时候也是这样.

如果你打算从套接口读取数据的话, **你将不需要传递某个回调函数或者注册一个事件侦听器. 相反, 你只要书写yield $socket->read(). 这儿大部分都是你常常也要编写的,只 在它的前面增加yield.**

当我第一次听到协程的时候, 我发现这个概念完全令人折服, 正是因为这个激励我在PHP中实现了它. **同时我发现协程真正非常的令人惊叹:在令人敬畏的代码和一大堆乱代码之间只有一线之隔**, 我认为协程恰好处在这条线上, 不多不少. **不过, 要说使用上面所述的方法书写异步代码是否真的有益, 这个就见仁见智了.**

#### 使用场景
- IO操作密集型，配合非阻塞调用、非阻塞函数使用，最好有SELECT监听函数，主要在于节省IO操作过程的时间，用来进行其他操作。
- C语言层面使用了Coroutine协程做并发Http请求的优化（多线程下的所有Http请求调度）
- PHP层面只能在单线程的情况下，对多任务（配合非阻塞调用）操作进行单线程执行效率的优化

#### Swoole协程运行过程（C语言层面的）
- 调用`onRequest`事件回调函数时，底层会调用C函数`coro_create`创建一个协程（#1位置），同时保存这个时间点的CPU寄存器状态和ZendVM stack信息。
- **调用`mysql->connect`时发生IO操作，底层会调用C函数`coro_save`保存当前协程的状态**，包括Zend VM上下文以及协程描述信息，并调用**coro_yield**让出程序控制权，当前的请求会挂起（#2位置）
- **协程让出程序控制权后，会继续进入EventLoop处理其他事件，这时Swoole会继续去处理其他客户端发来的Request**
- IO事件完成后，MySQL连接成功或失败，底层调用C函数`core_resume`恢复对应的协程，恢复ZendVM上下文，继续向下执行PHP代码（#3位置）
- `mysql->query`的执行过程与`mysql->connect`一致，也会进行一次协程切换调度
- 所有操作完成后，调用end方法返回结果，并销毁此协程

参考文章：
- [在PHP中使用协程实现多任务调度](http://www.laruence.com/2015/05/28/3038.html/comment-page-1#comment-220491)
- [PHP 协程](http://yangxikun.com/php/2016/03/20/php-generators.html)
