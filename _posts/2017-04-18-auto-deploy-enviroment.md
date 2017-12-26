---
title: 自动化部署环境
categories:
 - 架构设计
tags:
 - 自动化部署
 - 部署环境
---

### 测试环境
提供测试人员使用，代码分支除了可以使用master分支外，其他的分支也是可以的。


### 回归环境
如果同时有好几个人参与同一个项目，**那么基于master分支可能拉出非常多的开发分支**，那么当这些分支合并到master上后，master上的功能可能受到影响，这种情况下，会使用一个回归环境，部署master分支的代码。


### 预发布环境
无论你的WEB开发过程是怎样的,最终的代码和内容还是要通过发布来送达到用户浏览器中,你可以对PK需求,修改BUG,延长加班毫无畏惧,但你不能忽略用户体验.代码一旦部署到正式环境上,对你工作的评判不再是项目组中关心你,体谅你的同事.而是千万对错误零容忍的用户. 在发布前你已经做过周全的测试? 新增的每一项功能已经测试过? 很好.不过是在你的开发环境或某处偏僻的”测试环境”中? **服务器OS不一样,Web Server有差别,缓存服务未启用,APP容器或解释器,数据库版本有差别,没接通第三方API**, 这所有的一切都可能会造成发布后,你自己或用户刷新网站后的那声”What The fuck?”, 我想这应该是较之修改BUG,你更不想面对的情景吧.

**总的说来,”预发布环境”就等于没有真实用户访问的生产环境, 除了让用户不能访问到外,尽一切可能让这个环境和生产环境一致.每次正式发布时以这个环境为目标,测试流程完成后.把发布内容从这个环境”平移”到生产环境.**

从线上负载均衡集群里摘一台机器出来，这台机器就可以作为预发机了。

预发机是跑在正式环境，连接正式数据库的一台机器，但是不在线上负载均衡集群内，也就是说这台机器，可以进行线上环境的测试，但是不影响用户访问。

### 灰度发布
预发布环境过后，就是灰度发布了。由于一个项目，一般会部署到多台机器，所以灰度1台至三台，看看新功能是否ok，如果失败则只需要回滚几台，比较方便，AB test就是一种灰度发布方式。注意，由于是灰度发布几台，所以一般会使用跳板机，然后进行域名绑定，这样才可以保证只访问有最新代码的服务器。

![灰度发布](https://zgjian-pic.oss.cn-beijing.aliyuncs.com/markdown/220350j03i0znju45rlvr9.png)

#### 跳板机
是开发者登录到腾讯分配给应用服务器的唯一途径。开发者必须首先登录跳板机，再通过跳板机登录到应用服务器。

#### [nginx 灰度发布（基于cookies） ](http://blog.chinaunix.net/uid-531464-id-4140473.html)
灰度发布一般有三种方式 nginx+lua，nginx根据cookie分流，nginx 根据权重来分配：
- nginx+lua根据来访者ip地址区分，由于公司出口是一个ip地址，会出现访问网站要么都是老版，要么都是新版，采用这种方式并不适合
- nginx 根据权重来分配，实现很简单，也可以尝试
- nginx根据cookie分流，灰度发布基于用户才更合理

前端nginx服务器监听端口80，需要根据cookie转发，查询的cookie的键（key）为tts_version_id（该键由开发负责增加），如果该cookie值（value）为tts1则转发到tts_V6，为tts2则转发到tts_V7。
```nginx
upstream tts_V6 {
        server 192.168.3.81:5280 max_fails=1 fail_timeout=60;
}
upstream tts_V7 {
       server 192.168.3.81:5380 max_fails=1 fail_timeout=60;
}
upstream default {
        server 192.168.3.81:5280 max_fails=1 fail_timeout=60;
}
server {
        listen 80;
        server_name  test.taotaosou.com;
       access_log  logs/test.taotaosou.com.log  main buffer=32k;
        #match cookie
        set $group "default";
        if ($http_cookie ~* "tts_version_id=tts1"){
                set $group tts_V6;
        }
        if ($http_cookie ~* "tts_version_id=tts2"){
                set $group tts_V7;
        }
        location / {                       
                proxy_pass http://$group;
                proxy_set_header   Host             $host;
                proxy_set_header   X-Real-IP        $remote_addr;
                proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
                index  index.html index.htm;
       }
 }

```

#### [新浪的动态策略灰度发布系统：ABTestingGateway](https://github.com/CNSRE/ABTestingGateway)
![分流过程流程图](https://zgjian-pic.oss.cn-beijing.aliyuncs.com/markdown/div_flowchart.png)

![架构简图](https://zgjian-pic.oss.cn-beijing.aliyuncs.com/markdown/20150818/20150818171120_268.png)

### 生产发布
所有服务器上的代码都已经是最新的了。
