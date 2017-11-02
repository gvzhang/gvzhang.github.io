---
title: Yii2架构探究
categories:
 - 开发框架
tags:
 - yii2
 - 架构
---

基于Yii2框架的架构分析图，整理了请求的生命周期以及主要功能模块的关系。

![Yii2架构分析图](http://static.zgjian.cc/post/581fd4d43e76f.png-waterMark?v=11 "Yii2架构分析图")

通过DI（依赖注入）载入各模块

重要模块说明：
- 路由、请求模块
- Module模块，将应用了多块模块，便于开发，管理
- Controller解析Action参数，调度执行Action，根据结果渲染Views
- Action，实现业务逻辑；主要关联模块就是数据库操作的Model
- Model，AR模型，数据库封装
- Views，模板引擎

服务说明：
- 会话、Cookie
- Exception、Logger
- 认证授权
- 缓存
- 加密（TOKEN）
- 国际化
- 验证类
- Restful
- 邮箱
- 任务调度
- 队列
- 等等。。。


其他：
- Codeception测试框架
