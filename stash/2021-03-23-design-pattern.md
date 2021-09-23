---
layout: post
title: 设计模式之开篇
date: 2021-3-23 23:49
categories: Design-Pattern
tags: design-Pattern
author: renshuo
mathjax: true
---

* content
{:toc}

本文首先介绍了面向对象的基本原则，然后对所以的设计模式进行了汇总。

<!--more-->

## 面向对象基本原则

### 1 单一职责原则 (SRP)

*Single Responsibility Principle*

一个类应该仅有一个引起其变化的原因。

每个类都应该有一个单一的功能，并且该功能应该由这个类完全封装起来。所有它的（这个类的）服务都应该严密的和该功能平行（功能平行，意味着没有依赖）。

### 2 开闭原则 (OCP)

*Open Close Principle*

软件中的对象（类，模块，函数等等）应该对于扩展是开放的，但是对于修改是封闭的。

提倡通过继承的方式扩展新的功能。

### 3 里氏替换原则 (LSP)

*Liskov Substitution Principle*

所有引用基类（父类）的地方必须能透明地使用其子类的对象。

创建基类的新的子类时，不应该改变基类的行为。

### 4 依赖倒置原则 (DIP)

*Dependence Inversion Principle*

1. 高层次的模块不应该依赖于低层次的模块，两者都应该依赖于抽象接口。
2. 抽象接口不应该依赖于具体实现。而具体实现则应该依赖于抽象接口。

使得高层次的模块不依赖于低层次的模块的实现细节，依赖关系被颠倒（反转），从而使得低层次模块依赖于**高层次模块的需求**抽象。

### 5 接口隔离原则 (ISP)

*Interface Segregation Principle*

不应该强行要求客户端依赖于它们不用的接口；类之间的依赖应该建立在最小的接口上面，这里最小的粒度取决于单一职责原则的划分。

把大的接口应该拆分成小的接口。

### 6 迪米特法则 (LOD)

*Law of Demeter*

 一个对象应当尽可能少的去了解其它对象，尽可能少的与其他对象发生相互作用。

也称为最少知道原则(Least Knowledge Principle, LKP)

### 7 组合/聚合复用原则 (CARP)

*Composite/Aggregate Reuse Principle*

优先使用组合/聚合，而不是继承。

## 设计模式