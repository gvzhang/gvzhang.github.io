---
title: Nginx常用配置以及陷阱
categories:
 - web服务器
tags:
 - nginx
---

#### location匹配规则
##### 语法规则
>`location [=|~|~*|^~] /uri/ { … }`

模式	 | 含义
---- | ---
location = /uri | = 表示精确匹配，只有完全匹配上才能生效
location ^~ /uri | ^~ 开头对URL路径进行前缀匹配，并且在正则之前。
location ~ pattern | 开头表示区分大小写的正则匹配
location ~* pattern | 开头表示不区分大小写的正则匹配
location /uri | 不带任何修饰符，也表示前缀匹配，但是在正则匹配之后
location /	 | 通用匹配，任何未匹配到其它location的请求都会匹配到，相当于switch中的default

前缀匹配时，Nginx 不对 url 做编码，因此请求为 /static/20%/aa，可以被规则 ^~ /static/ /aa 匹配到（注意是空格）

多个 location 配置的情况下匹配顺序为（参考资料而来，还未实际验证，试试就知道了，不必拘泥，仅供参考）:
- 首先精确匹配 `=`
- 其次前缀匹配 `^~`
- 其次是按文件中顺序的正则匹配
- 然后匹配不带任何修饰的前缀匹配。
- 最后是交给 `/` 通用匹配
- 当有匹配成功时候，停止匹配，按当前匹配规则处理请求

注意：前缀匹配，如果有包含关系时，按最大匹配原则进行匹配。比如在前缀匹配：`location /dir01` 与 `location /dir01/dir02`，如有请求 `http://localhost/dir01/dir02/file` 将最终匹配到 `location /dir01/dir02`

#### rewrite 语法
- last – 基本上都用这个 Flag（rewrite匹配后，会再次发起一个请求，只会对location里的规则再次匹配）
- break – 中止 Rewirte，不再继续匹配（rewrite匹配后，不会发起请求，也不会匹配后面的规则）
- redirect – 返回临时重定向的 HTTP 状态 302
- permanent – 返回永久重定向的 HTTP 状态 301

#### if 是邪恶的
当在 location 区块中使用 if 指令的时候会有一些问题, 在某些情况下它并不按照你的预期运行而是做一些完全不同的事情。而在另一些情况下他甚至会出现段错误。一般来说避免使用 if 指令是个好主意。

在 location 区块里 if 指令下唯一 100% 安全的指令应该只有:

> return …; rewrite … last;

除此以外的指令都可能导致不可预期的行为, 包括诡异的发出段错误信号 (SIGSEGV)。

#### Nginx 静态文件服务
我们先来看看最简单的本地静态文件服务配置示例：

```nginx
server {
	listen       80;
	server_name www.test.com;
	charset utf-8;
	root   /data/www.test.com;
	index  index.html index.htm;
}
```

就这些？恩，就这些！如果只是提供简单的对外静态文件，它真的就可以用了。可是他不完美，远远没有发挥 Nginx 的半成功力，为什么这么说呢，看看下面的配置吧，为了大家看着方便，我们把每一项的作用都做了注释。

```nginx
http {
    # 这个将为打开文件指定缓存，默认是没有启用的，max 指定缓存数量，
    # 建议和打开文件数一致，inactive 是指经过多长时间文件没被请求后删除缓存。
    open_file_cache max=204800 inactive=20s;

    # open_file_cache 指令中的inactive 参数时间内文件的最少使用次数，
    # 如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个
    # 文件在inactive 时间内一次没被使用，它将被移除。
    open_file_cache_min_uses 1;

    # 这个是指多长时间检查一次缓存的有效信息
    open_file_cache_valid 30s;

    # 默认情况下，Nginx的gzip压缩是关闭的， gzip压缩功能就是可以让你节省不
    # 少带宽，但是会增加服务器CPU的开销哦，Nginx默认只对text/html进行压缩 ，
    # 如果要对html之外的内容进行压缩传输，我们需要手动来设置。
    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml;

    server {
       listen       80;
       server_name www.test.com;
       charset utf-8;
       root   /data/www.test.com;
       index  index.html index.htm;
    }
}
```

我们都知道，应用程序和网站一样，其性能关乎生存。但如何使你的应用程序或者网站性能更好，并没有一个明确的答案。代码质量和架构是其中的一个原因，但是在很多例子中我们看到，你可以通过关注一些十分基础的应用内容分发技术（basic application delivery techniques），来提高终端用户的体验。其中一个例子就是实现和调整应用栈（application stack）的缓存。

#### Nginx 陷阱和常见错误
##### chmod 777
永远不要 使用 777，这可能是一个漂亮的数字，有时候可以懒惰的解决权限问题， 但是它同样也表示你没有线索去解决权限问题，你只是在碰运气。 你应该检查整个路径的权限，并思考发生了什么事情。
要轻松的显示一个路径的所有权限，你可以使用：
```powershell
namei -om /path/to/check
```

##### 用 if 判断 Server Name
糟糕的配置：
```nginx
server {
    server_name example.com *.example.com;
        if ($host ~* ^www\.(.+)) {
            set $raw_domain $1;
            rewrite ^/(.*)$ $raw_domain/$1 permanent;
        }
        # [...]
    }
}
```

这个配置有三个问题。首先是 if 的使用, 为啥它这么糟糕呢? 你有阅读邪恶的 if 指令吗? 当 Nginx 收到无论来自哪个子域名的何种请求, 不管域名是 www.example.com 还是 example.com，这个 if 指令 总是 会被执行。 因此 Nginx 会检查 每个请求 的 Host header，这是十分低效的。 你应该避免这种情况，而是使用下面配置里面的两个 server 指令。

推荐的配置：
```nginx
server {
    server_name www.example.com;
    return 301 $scheme://example.com$request_uri;
}
server {
    server_name example.com;
    # [...]
}
```

除了增强了配置的可读性，这种方法还降低了 Nginx 的处理要求；我们摆脱了不必要的 if 指令； 我们用了 $scheme 来表示 URI 中是 http 还是 https 协议，避免了硬编码。

##### 用 if 检查文件是否存在
使用 if 指令来判断文件是否存在是很可怕的，如果你在使用新版本的 Nginx， 你应该看看 try_files，这会让你的生活变得更轻松。
糟糕的配置：
```nginx
server {
    root /var/www/example.com;
    location / {
        if (!-f $request_filename) {
            break;
        }
    }
}
```

推荐的配置：
```nginx
server {
    root /var/www/example.com;
    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

我们不再尝试使用 if 来判断 `$uri` 是否存在，用 `try_files` 意味着你可以测试一个序列。 如果 `$uri` 不存在，就会尝试 `$uri/`，还不存在的话，在尝试一个回调 location。

##### 把不可控制的请求发给 PHP
很多网络上面推荐的和 PHP 相关的 Nginx 配置，都是把每一个 .php 结尾的 URI 传递给 PHP 解释器。 请注意，大部分这样的 PHP 设置都有严重的安全问题，因为它可能允许执行任意第三方代码。

有问题的配置通常如下：
```nginx
location ~* \.php$ {
    fastcgi_pass backend;
    # [...]
}
```

在这里，每一个 .php 结尾的请求，都会传递给 FastCGI 的后台处理程序。 这样做的问题是，当完整的路径未能指向文件系统里面一个确切的文件时， 默认的 PHP 配置试图是猜测你想执行的是哪个文件。

举个例子，如果一个请求中的 /forum/avatar/1232.jpg/file.php 文件不存在， 但是 /forum/avatar/1232.jpg 存在，那么 PHP 解释器就会取而代之， 使用 /forum/avatar/1232.jpg 来解释。如果这里面嵌入了 PHP 代码， 这段代码就会被执行起来。

有几个避免这种情况的选择：
- 在 php.ini 中设置 cgi.fix_pathinfo=0。 这会让 PHP 解释器只尝试给定的文件路径，如果没有找到这个文件就停止处理。
- 确保 Nginx 只传递指定的 PHP 文件去执行
```nginx
location ~* (file_a|file_b|file_c)\.php$ {
    fastcgi_pass backend;
    # [...]
}
```
- 对于任何用户可以上传的目录，特别的关闭 PHP 文件的执行权限
```nginx
location /uploaddir {
    location ~ \.php$ {return 403;}
    # [...]
}
```
- 使用 try_files 指令过滤出文件不存在的情况
```nginx
location ~* \.php$ {
    try_files $uri =404;
    fastcgi_pass backend;
    # [...]
}
```
- 使用嵌套的 location 过滤出文件不存在的情况
```nginx
location ~* \.php$ {
    location ~ \..*/.*\.php$ {return 404;}
    fastcgi_pass backend;
    # [...]
}
```

##### 丢失（消失）的 HTTP 头
如果你没有明确的设置 underscores_in_headers on; , Nginx 将会自动丢弃带有下划线的 HTTP 头(根据 HTTP 标准，这样做是完全正当的). 这样做是为了防止头信息映射到 CGI 变量时产生歧义，因为破折号和下划线都会被映射为下划线。

##### 使用主机名来解析地址
糟糕的配置：
```nginx
upstream {
    server http://someserver;
}

server {
    listen myhostname:80;
    # [...]
}
```

你不应该在 listen 指令里面使用主机名。 虽然这样可能是有效的，但它会带来层出不穷的问题。 其中一个问题是，这个主机名在启动时或者服务重启中不能解析。 这会导致 Nginx 不能绑定所需的 TCP socket 而启动失败。

一个更安全的做法是使用主机名对应 IP 地址，而不是主机名。 这可以防止 Nginx 去查找 IP 地址，也去掉了去内部、外部解析程序的依赖。
