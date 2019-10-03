---
title: 对象关系教程(Object Relational Tutorial)
date: 2019-09-21 21:06:06
tags:
categories:
- 翻译
- SQLAlchemy官方文档
---

SQLAlchemy Object Relational Mapper 提出了一个将用户自定义Python类和数据库表，类实例和对应的表里的行结合起来-的方法，包含一个同步在对象和相关的行的状态的所有改变的系统,叫做`unit of work`,以及一个通过用户定义的类来表达数据库查询和定义相互的关系的系统.

ORM 对于SQLAlchemy Expression Language 是结构化的.然而SQLAlchemy Expression Language ,在 SQLAlchemy Tutorial 里介绍的,提出了一个直接表达关系型数据库原始结构的系统,ORM 提出了一个高等级的,抽象的使用方式,它自己就是Expression Language的applyied usage的一个例子.

尽管在ROM和Expression Language的用法上有一些重叠,但相似之处比他们一开始显现的要肤浅.

用户自定义的domain model(领域模型)

一个成功的应用可能仅仅使用Object Relation Mapper .在高级的场合,一个由ORM构成的应用偶尔的在指定数据库交互适当的地方使用Expression Language ,