---
layout: post
title: UML类图总结
date: 2021-3-20 22:00
categories: UML
tags: UML
author: renshuo
mathjax: true
---

* content
{:toc}

关于UML中类图的总结。

<!--more-->

![summary](/pictures/uml/summary.png)

## 实现 (Realization)

- 是一种类与接口的关系

- 带三角箭头的虚线，箭头指向接口

![realization](/pictures/uml/realization.png)

## 泛化 (Generalization)

- 子类和父类之间的继承关系，`"is a"` 的关系
- 用空心三角和实线组成的箭头表示，从子类指向父类

![generalization](/pictures/uml/generalization.png)

## 关联 (Association)

- 对象和对象之间的联系，使一个对象可以调用另一个对象的公共属性和方法
- 以类成员变量作为表现形式
- 单向关联关系，用一个带箭头的实线表示，箭头指向被关联的(成员变量)对象
- 双向关联关系，用双箭头实线或者无箭头实线表示

对象A可以持有对象B的构成的数组或者集合，在关联线的末端，可以用以下表达式进行说明：

- 数字：精确的数量
- `*`或者`0..*`：表示0到多个
- `0..1`：表示0或者1个
- `1..*`：表示1到多个

![association](/pictures/uml/association.png)

![association2](/pictures/uml/association2.png)

## 依赖 (Dependency)

- 弱关联关系，`"use a"` 的关系
- 用一个带虚线的箭头表示，由使用方指向被使用方，表示使用方对象持有被使用方对象的引用

具体表现形式：

- 构造器或者方法中的局部变量
- 方法参数或者返回值
- 调用静态方法

![dependency](/pictures/uml/dependency.png)

## 聚合 (Aggregation)

- 整体与部分的拥有关系，`"has a"` 的关系
- 整体与部分之间是可分离的，可以具有各自的生命周期，部分可以属于多个整体对象，也可以为多个整体对象共享
- 用空心菱形加实线箭头表示，空心菱形在整体一方，箭头指向部分一方

![aggregation](/pictures/uml/aggregation.png)

## 组合 (Composition)

- 整体与部分的包含关系，`"contains a"` 的关系，比聚合关系更强
- 整体与部分是不可分的，部分也不能给其它整体共享，作为整体的对象负责部分的对象的生命周期
- 用实心菱形加实线箭头表示，实心菱形在整体一方，箭头指向部分一方

![composition](/pictures/uml/composition.png)

