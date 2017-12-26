---
title: ActiveRecord模式
categories:
 - 开发框架
tags:
 - ActiveRecord
 - 数据库模型
---

> Active Record是一种领域模型模式，特点是一个模型类对应关系型数据库中的一个表，而模型类的一个实例对应表中的一行记录。Active Record和Row Gateway十分相似，但前者是领域模型，后者是一种数据源模式。关系型数据库往往通过外键来表述实体关系，Active Record在数据源层面上也将这种关系映射为对象的关联和聚集。

> Active Record适合非常简单的领域需求，尤其在领域模型和数据库模型十分相似的情况下。如果遇到更加复杂的领域模型结构（例如用到继承、策略的领域模型），往往需要使用分离数据源的领域模型，结合Data Mapper（数据映射器）使用。

![YIi2-ActiveRecord](https://zgjian-pic.oss.cn-beijing.aliyuncs.com/post/Analyze_yii2_database_layout.png-waterMark "YIi2-ActiveRecord")

