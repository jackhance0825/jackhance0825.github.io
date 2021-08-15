---
title: Spring 卷 - IOC 篇 - IOC 的前世今生
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

通过控制反转，流程取决于在程序执行期间建立的对象图。通过抽象定义的对象交互使动态的业务流成为可能。运行时绑定可通过依赖注入（dependence injection）或服务定位器（service locator）等机制实现。代码可在编译期间静态链接，是通过从外部配置读取其描述，而不是直接引用代码本身来找到要执行的代码。

控制反转用于增加程序的模块化并使其方便扩展，并在面向对象编程和其他编程范式中得到应用。


<hr>

#### IOC 的足迹

- 1983年，Richard E. Sweet 在《The Mesa Programming Environment》中提出“Hollywood
Principle”（好莱坞原则）Don‘t call us, we’ll call you. <sup id="hollywood-principle">[[1]](#hollywood-principle-ref)</sup>
- 1988年，Ralph E. Johnson & Brian Foote 在《Designing Reusable Classes》中提出“Inversion
of control”（控制反转）<sup id="inversion-of-control">[[2]](#inversion-of-control-ref)</sup>
- 1996年，Michael Mattsson 在《Object-Oriented Frameworks, A survey of methodological
issues》中将“Inversion of control”命名为 “Hollywood principle”
- 2004年，Martin Fowler 在《Inversion of Control Containers and the Dependency Injection
pattern》中提出了自己对 IoC 以及 DI 的理解 <sup id="inversion-of-control-DI">[[3]](#inversion-of-control-DI-ref)</sup>
- 2005年，Martin Fowler 在 《InversionOfControl》对 IoC 做出进一步的说明 <sup id="inversion-of-control-more">[[4]](#inversion-of-control-more-ref)</sup>


<hr>

#### IOC 的实现策略

- 使用服务定位器模式
- 依赖注入
    - 构造函数注入
    - 参数注入
    - setter方法注入
    - 接口注入
- 上下文查找
- 模板方法设计模式
- 策略设计模式

<hr>

#### IOC 容器到底是什么

《Expert One-on-One™ J2EE™ Development without EJB™》提到关于轻量级容器的特征：
- 可以管理应用程序代码的容器。
- 一个快速启动的容器。
- 不需要任何特殊部署步骤即可在其中部署对象的容器。
- 一个具有如此轻量级和最小 API 依赖性的容器，它可以在各种环境中运行。
- 在部署工作量和性能方面为添加托管对象设置了如此低的标准的容器，可以部署和管理细粒度对象以及粗粒度组件的开销。


《Expert One-on-One™ J2EE™ Development without EJB™》提到关于轻量级容器的好处：
- 逃离单体容器的约束
- 最大化代码可重用性
- 更多的面向对象
- 更高的生产力
- 更好的可测试性

因此，我们所期待的 IOC 容器应该具备以上的特征。

<hr>

#### IOC 容器职责

- 依赖处理
    - 依赖查找
    - 依赖注入
- 生命周期管理
    - 容器
    - 托管的资源（如Java Beans）
- 配置
    - 容器
    - 外部化配置
    - 托管的资源（如Java Beans）

<hr>

#### IOC 容器实现

- Java SE
    - Java Beans： JavaBeans<sup id="java-beans">[[5]](#java-beans-ref)</sup> BeanContext<sup id="bean-context">[[6]](#bean-context-ref)</sup>
    - Java ServiceLoader SPI
    - JNDI
- Java EE
    - EJB
    - Servlet
- 开源框架
    - <a href="http://avalon.apache.org/closed.html"  target="_blank" >Apache Avalon</a>
    - <a href="http://picocontainer.com/"  target="_blank" >PicoContainer</a>
    - <a href="https://github.com/google/guice"  target="_blank" >Google Guice</a>
    - <a href="https://spring.io/projects/spring-framework"  target="_blank" >Spring Framework</a>

<hr>

#### 小记

<small id="hollywood-principle-ref"><sup>[[1]](#hollywood-principle)</sup> <a href="https://www.digibarn.com/friends/curbow/star/XDEPaper.pdf" target="_blank" >《The Mesa Programming Environment》</a></small> Don‘t call us, we’ll call you ：  us 指的是所需的资源，you 指的是系统或者模块

<small id="inversion-of-control-ref"><sup>[[2]](#inversion-of-control)</sup> <a href="http://www.laputan.org/drc.html" target="_blank" >《Designing Reusable Classes》</a></small>

<small id="inversion-of-control-DI-ref"><sup>[[3]](#inversion-of-control-DI)</sup> <a href="https://martinfowler.com/articles/injection.html" target="_blank" >《Inversion of Control Containers and the Dependency Injection
pattern》</a></small>

<small id="inversion-of-control-more-ref"><sup>[[4]](#inversion-of-control-more)</sup> <a href="https://martinfowler.com/bliki/InversionOfControl.html" target="_blank" >《InversionOfControl》</a></small>

<small id="java-beans-ref"><sup>[[5]](#java-beans)</sup> <a href="https://www.oracle.com/technetwork/java/javase/tech/index-jsp-138795.html" target="_blank" >JavaBeans 规范</a></small>

<small id="bean-context-ref"><sup>[[6]](#bean-context)</sup> <a href="https://docs.oracle.com/javase/8/docs/technotes/guides/beans/spec/beancontext.html" target="_blank" >BeanContext 规范</a></small>


