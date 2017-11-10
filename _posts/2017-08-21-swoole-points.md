---
title: Swoole知识点
categories:
 - PHP
 - 网络编程
tags:
 - Swoole
---

### 运行流程图
#### 主进程
**主进程内有多个Reactor线程**，基于epoll/kqueue进行网络事件轮询。**收到数据后转发到worker进程**去处理

#### Manager进程
**对所有worker进程进行管理**，worker进程生命周期结束或者发生异常时自动回收，并创建新的worker进程

#### worker进程
对收到的数据进行处理，包括**协议解析和响应请求**。

#### Task任务
Swoole的业务逻辑部分是同步阻塞运行的，如果遇到一些耗时较大的操作，例如访问数据库、广播消息等，就会影响服务器的响应速度。因此Swoole提供了Task功能，**将这些耗时操作放到另外的进程去处理，当前进程继续执行后面的逻辑**。

![运行流程图](https://wiki.swoole.com/static/uploads/swoole.jpg)

### 进程/线程结构图
![进程/线程结构图](https://wiki.swoole.com/static/image/process.jpg)

### swoole_server中对象的4层生命周期
#### 程序全局期
**在swoole_server->start之前就创建好的对象**，我们称之为程序全局生命周期。这些变量在程序启动后就会一直存在，**直到整个程序结束运行才会销毁**。

#### 进程全局期
worker进程启动后创建的对象（onWorkerStart中创建的对象），在这个子进程存活周期之内，是常驻内存的。onConnect/onReceive/onClose 中都可以去访问它。

>**进程全局对象所占用的内存是在当前子进程内存堆的，并非共享内存。对此对象的修改仅在当前worker进程中有效**
>进程期include/require的文件，在reload后就会重新加载

#### 会话期
会话期是在**onConnect后创建**，或者在第一次onReceive时创建，**onClose时销毁**。一个客户端连接进入后，创建的对象会常驻内存，直到此客户端离开才会销毁。

swoole中会话期的对象直接是常驻内存，不需要session_start之类操作。可以直接访问对象，并执行对象的方法。

#### 请求期
**请求期就是指一个完整的请求发来**，也就是onReceive收到请求开始处理，直到返回结果发送response。**这个周期所创建的对象，会在请求完成后销毁**。

**swoole中请求期对象与普通PHP程序中的对象就是一样的**。请求到来时创建，请求结束后销毁。

### 事件执行顺序
- 所有事件回调均在$server->start后发生
- 服务器关闭程序终止时最后一次事件是onShutdown
- 服务器启动成功后，onStart/onManagerStart/onWorkerStart会在不同的进程内并发执行。
- onReceive/onConnect/onClose/onTimer在worker进程(包括task进程)中各自触发
- worker/task进程启动/结束时会分别调用onWorkerStart/onWorkerStop
- onTask事件仅在task进程中发生
- onFinish事件仅在worker进程中发生
>onStart/onManagerStart/onWorkerStart 3个事件的执行顺序是不确定的

#### swoole_server->start
启动server，监听所有TCP/UDP端口，函数原型：
```php
bool swoole_server->start()
```

- 启动成功后会创建worker_num+2个进程。Master进程+Manager进程+`serv->worker_num`个Worker进程。
- 启动失败会立即返回false
- 启动成功后将进入事件循环，等待客户端连接请求。**start方法之后的代码不会执行**
- 服务器关闭后，start函数返回true，并继续向下执行

>设置了task_worker_num会增加相应数量的Task进程
>函数列表中start之前的方法仅可在start调用前使用，**在start之后的方法仅可在onWorkerStart、onReceive等事件回调函数中使用**

#### onStart
Server启动在主进程的主线程回调此函数，函数原型
```php
function onStart(swoole_server $server);
```
在此事件之前Swoole Server已进行了如下操作
- 已创建了manager进程
- 已创建了worker子进程
- 已监听所有TCP/UDP端口
- 已监听了定时器

接下来要执行
- 主Reactor开始接收事件，客户端可以connect到Server

**onStart回调中，仅允许echo、打印Log、修改进程名称。不得执行其他操作。onWorkerStart和onStart回调是在不同进程中并行执行的，不存在先后顺序。**

可以在onStart回调中，将`$serv->master_pid`和`$serv->manager_pid`的值保存到一个文件中。这样可以编写脚本，向这两个PID发送信号来实现关闭和重启的操作。

>- **在onStart中创建的全局资源对象不能在worker进程中被使用，因为发生onStart调用时，worker进程已经创建好了。**
>- 新创建的对象在主进程内，worker进程无法访问到此内存区域。
>- 因此全局对象创建的代码需要放置在swoole_server_start之前。

### 疑问
#### 进程、线程资源生命周期混乱，什么时候开启什么时候销毁？

通过设置Swoole的max_request参数，**worker进程的生命周期是可以控制的，生命周期结束后会自动回收所有内存**，所以轻微的内存泄露问题也不大。

##### 局部变量
在事件回调函数返回后，所有局部对象和变量会全部回收，不需要unset。如果变量是一个资源类型，那么对应的资源也会被PHP底层释放。

##### 全局变量
在PHP中，有3类全局变量。
- 使用global关键词声明的变量
- 使用static关键词声明的类静态变量、函数静态变量
- PHP的超全局变量，包括`$_GET`、`$_POST`、`$GLOBALS`等

**全局变量和对象，类静态变量，保存在swoole_server对象上的变量不会被释放。**需要程序员自行处理这些变量和对象的销毁工作。

- 在事件回调函数中需要特别注意非局部变量的array类型值，某些操作如 **`TestClass::$array[] = "string"` 可能会造成内存泄漏**，严重时可能发生爆内存，必要时应当注意清理大数组。
- 在事件回调函数中，非局部变量的字符串进行拼接操作是必须小心内存泄漏，如 **`TestClass::$string .= $data`，可能会有内存泄漏**，严重时可能发生爆内存。

##### 解决方法
1. **同步阻塞并且请求响应式无状态**的Server程序可以设置max_request，当Worker进程/Task进程结束运行时或达到任务上限后进程自动退出。该进程的所有变量/对象/资源均会被释放回收。（变成了跟php-fpm一样了）
2. 程序内在onClose或设置**定时器及时使用unset清理变量，回收资源**

##### 异步客户端
```php
function test()
{
    $client = new swoole_client(SWOOLE_TCP | SWOOLE_ASYNC);
    $client->on("connect", function($cli) {
        $cli->send("hello world\n");
    });
    $client->on("receive", function($cli, $data){
        echo "Received: ".$data."\n";
        $cli->close();
    });
    $client->on("error", function($cli){
        echo "Connect failed\n";
    });
    $client->on("close", function($cli){
        echo "Connection close\n";
    });
    $client->connect('127.0.0.1', 9501);
    return;
}
```
- `$client`是局部变量，常规情况下return时会销毁。
- 但这个`$client`是异步客户端在执行connect时swoole引擎底层会增加一次引用计数，因此return时并不会销毁。
- 该客户端执行onReceive回调函数时进行了close或者服务器端主动关闭连接触发onClose，这时底层会减少引用计数，`$client`才会被销毁。

#### 子进程、子线程资源共享细节？
参见《swoole_server中对象的4层生命周期》

#### 是否可以共用1个redis或mysql连接
绝对不可以。必须每个进程单独创建Redis、MySQL、PDO连接，其他的存储客户端同样也是如此。原因是如果共用1个连接，那么返回的结果无法保证被哪个进程处理。**持有连接的进程理论上都可以对这个连接进行读写，这样数据就发生错乱了。**

**所以在多个进程之间，一定不能共用连接**
- 在swoole_server中，应当在onWorkerStart中创建连接对象
- 在swoole_process中，应当在swoole_process->start后，子进程的回调函数中创建连接对象
- 本页面所述信息对使用pcntl_fork的程序同样有效

##### 示例
```php
$serv = new swoole_server("0.0.0.0", 9502);

//必须在onWorkerStart回调中创建redis/mysql连接
$serv->on('workerstart', function($serv, $id) {
    $redis = new redis;
    $redis->connect('127.0.0.1', 6379);
    $serv->redis = $redis;
});

$serv->on('receive', function (swoole_server $serv, $fd, $from_id, $data) { 
    $value = $serv->redis->get("key");
    $serv->send($fd, "Swoole: ".$value);
});

$serv->start();
```

### 其他注意事项
#### swoole_timer_tick
设置一个间隔时钟定时器，与after定时器不同的是tick定时器会持续触发，直到调用swoole_timer_clear清除。**定时器仅在当前进程空间内有效**
```php
int swoole_timer_tick(int $ms, callable $callback, mixed $user_param);
```

#### worker_num
设置启动的worker进程数。
- 业务代码是全异步非阻塞的，这里设置为CPU的1-4倍最合理
- 业务代码为同步阻塞，需要根据请求响应时间和系统负载来调整

比如**1**个请求耗时**100ms**，要提供**1000QPS**的处理能力，那必须配置**100**个进程或更多。但开的进程越多，占用的内存就会大大增加，而且进程间切换的开销就会越来越大。所以这里适当即可。不要配置过大。
- 每个进程占用**40M**内存，那**100**个进程就需要占用**4G**内存

#### 类/函数重复定义
新手非常容易犯这个错误，由于swoole是常驻内存的，所以加载类/函数定义的文件后不会释放。**因此引入类/函数的php文件时必须要使用`include_once`或`require_once`**，否会发生`cannot redeclare function/class`的致命错误。

#### 进程隔离
**进程隔离也是很多新手经常遇到的问题。修改了全局变量的值，为什么不生效，原因就是全局变量在不同的进程，内存空间是隔离的，所以无效。**所以使用swoole开发Server程序需要了解进程隔离问题。

**如果需要在不同的Worker进程内共享数据，可以用Redis、MySQL、文件、Swoole\Table、APCu、shmget等工具实现**
**不同进程的文件句柄是隔离的，所以在A进程创建的Socket连接或打开的文件，在B进程内是无效，即使是将它的fd发送到B进程也是不可用的**

正确的做法是使用Swoole提供的`Swoole\Atomic`或`Swoole\Table`数据结构来保存数据。如上述代码可以使用`Swoole\Atomic`实现。
```php
$server = new Swoole\Http\Server('127.0.0.1', 9500);

$atomic = new Swoole\Atomic(1);

$server->on('Request', function ($request, $response) use ($atomic) {
    $response->end($atomic->add(1));
});

$server->start();
```
`Swoole\Atomic`数据是建立在共享内存之上的，使用add方法加1时，在其他工作进程内也是有效的。
