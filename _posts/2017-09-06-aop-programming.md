---
title: AOP-面向切面编程
categories:
 - 编程模式
tags:
 - aop
 - 面向切面编程
---

**在企业级开发中，AOP被广泛使用。**

在应用开发中，我们经常发现需要很多功能，这些功能需要经常被分散在代码中的多个点上，但是这些点事实上跟实际业务没有任何关联。比如，**在执行一些特殊任务之前需要确保用户是在登陆状态中**，我们把这些特殊人物就叫做"cross-cutting concerns"，让我们通过Wikipedia来了解一下"cross-cutting concerns"（横向关系）的定义。

在计算机科学中，**"cross-cutting concerns"指的是“切面（或方向）编程”。这些关系不能从其他系统（框架设计或者某些实现）中很好的分解出来，以至于出现代码重复，在系统中存在有意义的依赖关系，或者两者兼有之。**

```php
<?php  
class BlogPost extends CI_Controller  
{  
    public function createPost() {  
        if (!Authentication::checkAuthentication()) {  
            // redirect to login  
        }  
        else {  
            // proceed  
            Messages::notifyAdmin();  
        }  
    }  
 
    public function approvePost() {  
        if (!Authentication::checkAuthentication()) {  
            // redirect to login  
        }  
        else {  
            // proceed  
        }  
    }  
 
    public function editPost() {  
        if (!Authentication::checkAuthentication()) {  
            // redirect to login  
        }  
        else {  
            // proceed  
        }  
    }  
 
    public function viewPost() {  
        // ...  
    }  
} 
```

看上面的代码，你会发现在**每个方法之前都调用了checkAuthentication()**，因为这些行为需要用户登陆之后才能进行。还有就是notifyAdmin()来辨别是否是管理员帐号，以便创建新贴。看见没有，有**很多“重复的代码”**，而且BlogPost类，应该仅负责管理帖子。**验证和辨别身份应当是分离的。我们违反了“单一职责原则”。**

Spring实现的AOP是代理模式，给调用者使用的实际是已经过加工的对象，你编程时方法体里只写了A，但调用者拿到的对象的方法体却是xAy。
