---
title: PHP框架压测
categories:
 -开发框架
tags:
 - 测试
 - 压力测试
 - 框架
---

### 系统配置
- 虚拟机 2核 4G
- 主机ab压测虚拟机

### OpenResty
#### 测试代码
```nginx
server {
   #监听端口，若你的6699端口已经被占用，则需要修改
    listen 80;
    server_name openresty.test;
    location / {
        default_type text/html;

        content_by_lua '
            ngx.say("<p>hello, world</p>")
        ';
    }
}
```

#### 压测结果
```powershell
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking openresty.test (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        openresty/1.11.2.4
Server Hostname:        openresty.test
Server Port:            80

Document Path:          /
Document Length:        20 bytes

Concurrency Level:      100
Time taken for tests:   1.620 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1680000 bytes
HTML transferred:       200000 bytes
Requests per second:    6171.67 [#/sec] (mean)
Time per request:       16.203 [ms] (mean)
Time per request:       0.162 [ms] (mean, across all concurrent requests)
Transfer rate:          1012.54 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.2      0       1
Processing:     5   16   1.6     16      23
Waiting:        2   12   3.1     13      23
Total:          5   16   1.6     16      24

Percentage of the requests served within a certain time (ms)
  50%     16
  66%     17
  75%     17
  80%     17
  90%     18
  95%     19
  98%     20
  99%     21
 100%     24 (longest request)
```

### PHP源码输出
#### 测试代码
```php
echo "test";
```

#### 压测结果
```powershell
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking php5310.debug (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        nginx/1.13.3
Server Hostname:        php5310.debug
Server Port:            80

Document Path:          /sample.php
Document Length:        4 bytes

Concurrency Level:      100
Time taken for tests:   2.651 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1660000 bytes
HTML transferred:       40000 bytes
Requests per second:    3771.84 [#/sec] (mean)
Time per request:       26.512 [ms] (mean)
Time per request:       0.265 [ms] (mean, across all concurrent requests)
Transfer rate:          611.45 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.3      0       1
Processing:    15   26   1.8     26      37
Waiting:       13   26   1.9     26      37
Total:         15   26   1.8     26      37

Percentage of the requests served within a certain time (ms)
  50%     26
  66%     27
  75%     27
  80%     27
  90%     28
  95%     29
  98%     30
  99%     30
 100%     37 (longest request)
```

### PHP协程Server
#### 测试代码
```php
 $msg = <<<RES
Received following request:

$data

Mysql query result:$queryResult
RES;
    $msgLength = strlen($msg);

    $response = <<<RES
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: $msgLength
Connection: close

$msg
RES;
```

#### 压测结果
```powershell
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.168.10.10 (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:
Server Hostname:        192.168.10.10
Server Port:            8110

Document Path:          /
Document Length:        140 bytes

Concurrency Level:      100
Time taken for tests:   0.263 seconds
Complete requests:      1000
Failed requests:        0
Total transferred:      225000 bytes
HTML transferred:       140000 bytes
Requests per second:    3799.39 [#/sec] (mean)
Time per request:       26.320 [ms] (mean)
Time per request:       0.263 [ms] (mean, across all concurrent requests)
Transfer rate:          834.83 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.3      0       2
Processing:     2   25   6.3     24      38
Waiting:        1   14   7.6     14      35
Total:          2   25   6.4     25      38

Percentage of the requests served within a certain time (ms)
  50%     25
  66%     26
  75%     29
  80%     30
  90%     34
  95%     37
  98%     37
  99%     37
 100%     38 (longest request)
```

#### 注意
- 单一逻辑抛出异常，整个调度会 
- 连续多次的`ab -n 10000 -c 100 192.168.10.10:8110/`情况下，并发效率会变得很慢。为什么呢？？？

### Swoole原生
#### 测试代码
```php
$http = new swoole_http_server('0.0.0.0', 9501);

$http->on('request', function ($request, $response) {
    $resultEnd = "hello swoole";
    $response->header('Content-Type', 'text/html; charset=utf-8');
    $response->end($resultEnd);
});

$http->start();
```

#### 压测结果
```powershell
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking swoole.test (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        nginx/1.13.3
Server Hostname:        swoole.test
Server Port:            80

Document Path:          /
Document Length:        12 bytes

Concurrency Level:      100
Time taken for tests:   2.556 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1690000 bytes
HTML transferred:       120000 bytes
Requests per second:    3912.34 [#/sec] (mean)
Time per request:       25.560 [ms] (mean)
Time per request:       0.256 [ms] (mean, across all concurrent requests)
Transfer rate:          645.69 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.3      0       2
Processing:     2   25   6.6     28      47
Waiting:        1   24   8.4     27      44
Total:          2   25   6.6     28      47

Percentage of the requests served within a certain time (ms)
  50%     28
  66%     30
  75%     30
  80%     31
  90%     32
  95%     33
  98%     36
  99%     38
 100%     47 (longest request)
```

### Framework(Swoole)
#### 测试代码
```php
function index()
{
    echo __METHOD__;
}
```

#### 压测结果
```powershell
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking swoole5310.fw (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        openresty/1.11.2.4
Server Hostname:        swoole5310.fw
Server Port:            80

Document Path:          /
Document Length:        880 bytes

Concurrency Level:      100
Time taken for tests:   3.136 seconds
Complete requests:      10000
Failed requests:        2
   (Connect: 0, Receive: 0, Length: 2, Exceptions: 0)
Total transferred:      10599964 bytes
HTML transferred:       8799964 bytes
Requests per second:    3189.07 [#/sec] (mean)
Time per request:       31.357 [ms] (mean)
Time per request:       0.314 [ms] (mean, across all concurrent requests)
Transfer rate:          3301.18 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.2      0       2
Processing:    18   31   5.0     31      47
Waiting:        9   31   5.2     31      46
Total:         18   31   5.0     32      47

Percentage of the requests served within a certain time (ms)
  50%     32
  66%     33
  75%     34
  80%     35
  90%     37
  95%     40
  98%     42
  99%     43
 100%     47 (longest request)
```


### php-msf
#### 测试代码
```php
public function actionIndex()
{
    $this->output("hello world!");
}
```

#### 压测结果
```powershell
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.168.53.10 (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        swoole-http-server
Server Hostname:        192.168.53.10
Server Port:            8000

Document Path:          /
Document Length:        345 bytes

Concurrency Level:      100
Time taken for tests:   3.304 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      5330000 bytes
HTML transferred:       3450000 bytes
Requests per second:    3026.38 [#/sec] (mean)
Time per request:       33.043 [ms] (mean)
Time per request:       0.330 [ms] (mean, across all concurrent requests)
Transfer rate:          1575.25 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.2      0       2
Processing:    11   33   5.3     34      52
Waiting:        7   30   7.6     33      47
Total:         11   33   5.4     34      52

Percentage of the requests served within a certain time (ms)
  50%     34
  66%     35
  75%     36
  80%     37
  90%     38
  95%     40
  98%     42
  99%     43
 100%     52 (longest request)
```

### Swoft(Swoole)
#### 测试代码
```php
public function actionIndex()
{
    $data = $this->logic->getUser();
    $data['properties'] = App::$properties['env'];
    $this->outputJson($data, 'suc');
}
```

#### 压测结果
```powershell
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking swoft.test (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        nginx/1.13.3
Server Hostname:        swoft.test
Server Port:            80

Document Path:          /
Document Length:        149 bytes

Concurrency Level:      100
Time taken for tests:   3.332 seconds
Complete requests:      10000
Failed requests:        988
   (Connect: 0, Receive: 0, Length: 988, Exceptions: 0)
Total transferred:      3128919 bytes
HTML transferred:       1488919 bytes
Requests per second:    3000.87 [#/sec] (mean)
Time per request:       33.324 [ms] (mean)
Time per request:       0.333 [ms] (mean, across all concurrent requests)
Transfer rate:          916.94 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.2      0       1
Processing:    11   33   6.6     33     100
Waiting:       10   33   6.6     33     100
Total:         11   33   6.6     33     100

Percentage of the requests served within a certain time (ms)
  50%     33
  66%     34
  75%     35
  80%     36
  90%     38
  95%     42
  98%     52
  99%     58
 100%    100 (longest request)
```

### Lumen Framework
#### 测试代码
```php
$app->get('/', function () use ($app) {
    return $app->version();
});
```

#### 压测结果
```powershell
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking lumen5310.fw (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        nginx/1.13.3
Server Hostname:        lumen5310.fw
Server Port:            80

Document Path:          /
Document Length:        40 bytes

Concurrency Level:      100
Time taken for tests:   47.909 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      2360000 bytes
HTML transferred:       400000 bytes
Requests per second:    208.73 [#/sec] (mean)
Time per request:       479.087 [ms] (mean)
Time per request:       4.791 [ms] (mean, across all concurrent requests)
Transfer rate:          48.11 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.4      0       2
Processing:    30  476  49.5    465     740
Waiting:       30  476  49.5    465     740
Total:         31  477  49.5    465     740

Percentage of the requests served within a certain time (ms)
  50%    465
  66%    473
  75%    480
  80%    489
  90%    550
  95%    576
  98%    607
  99%    626
 100%    740 (longest request)
```

### Slim Framework
#### 测试代码
```php
$app->get('/', function ($request, $response, $args) {
    $response->write("Welcome to Slim!");
    return $response;
});
```

#### 压测结果
```powershell
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking slim5310.fw (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        nginx/1.13.3
Server Hostname:        slim5310.fw
Server Port:            80

Document Path:          /
Document Length:        16 bytes

Concurrency Level:      100
Time taken for tests:   35.723 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1980000 bytes
HTML transferred:       160000 bytes
Requests per second:    279.93 [#/sec] (mean)
Time per request:       357.228 [ms] (mean)
Time per request:       3.572 [ms] (mean, across all concurrent requests)
Transfer rate:          54.13 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.4      0       3
Processing:    32  355  29.5    352     592
Waiting:       31  355  29.5    352     591
Total:         32  355  29.5    353     592

Percentage of the requests served within a certain time (ms)
  50%    353
  66%    363
  75%    369
  80%    372
  90%    383
  95%    393
  98%    414
  99%    432
 100%    592 (longest request)
```

### Silex Framework
#### 测试代码
```php
$app->get('/', function () use ($app) {
    return $app->version();
});
```

#### 压测结果
```powershell
This is ApacheBench, Version 2.3 <$Revision: 1796539 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking silex5310.fw (be patient)
Completed 1000 requests
Completed 2000 requests
Completed 3000 requests
Completed 4000 requests
Completed 5000 requests
Completed 6000 requests
Completed 7000 requests
Completed 8000 requests
Completed 9000 requests
Completed 10000 requests
Finished 10000 requests


Server Software:        nginx/1.13.3
Server Hostname:        silex5310.fw
Server Port:            80

Document Path:          /hello
Document Length:        6 bytes

Concurrency Level:      100
Time taken for tests:   52.293 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      2400000 bytes
HTML transferred:       60000 bytes
Requests per second:    191.23 [#/sec] (mean)
Time per request:       522.925 [ms] (mean)
Time per request:       5.229 [ms] (mean, across all concurrent requests)
Transfer rate:          44.82 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.4      0       3
Processing:    60  520  56.5    503     897
Waiting:       60  520  56.5    503     897
Total:         61  520  56.5    503     898

Percentage of the requests served within a certain time (ms)
  50%    503
  66%    518
  75%    554
  80%    577
  90%    603
  95%    617
  98%    634
  99%    645
 100%    898 (longest request)
```
