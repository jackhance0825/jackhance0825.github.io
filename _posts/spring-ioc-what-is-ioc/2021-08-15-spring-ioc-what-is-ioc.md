---
title: Spring 卷 - 手撕 IOC 篇 - IOC 的前世今生
date: 2021-08-15 09:37:47 +07:00
modified: 2021-08-15 09:37:47 +07:00
tags: [java, spring, ioc]
description: 重新踏上 IOC 的旅途，回顾 TA 的前世今生
---


#### IOC 到底是什么

In software engineering, inversion of control (IoC) is a programming principle. IoC inverts the flow of control as compared to traditional control flow. In IoC, custom-written portions of a computer program receive the flow of control from a generic framework. A software architecture with this design inverts control as compared to traditional procedural programming: in traditional programming, the custom code that expresses the purpose of the program calls into reusable libraries to take care of generic tasks, but with inversion of control, it is the framework that calls into the custom, or task-specific, code.

———— <a href="https://en.wikipedia.org/wiki/Inversion_of_control" target="_blank" >维基百科</a>


IOC（inversion of control） 翻译过来是，控制反转。

在传统的编程中，业务逻辑流是被对象确定的静态绑定到彼此的，从而缺乏了灵活性。

通过控制反转，流程取决于在程序执行期间简历的对象图。通过抽象定于的对象交互使动态的业务流成为可能。运行时绑定可通过依赖注入（dependence injection）或服务定位器（service locator）等机制实现。代码可在编译期间静态链接，是通过从外部配置读取其描述，而不是直接引用代码本身来找到要执行的代码。

控制反转用于增加程序的模块化并使其方便扩展，并在面向对象编程和其他编程范式中得到应用。




<hr>

#### IOC 的诞生


<hr>

#### IOC 的实现策略


<hr>

#### IOC 容器到底是什么



<hr>

#### IOC 容器职责



<hr>

#### IOC 容器实现



