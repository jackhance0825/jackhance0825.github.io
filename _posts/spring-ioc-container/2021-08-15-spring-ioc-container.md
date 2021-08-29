---
title: Spring 卷 - IOC 篇 - Spring IOC 容器
date: 2021-08-15 15:37:47 +07:00
modified: 2021-08-15 15:37:47 +07:00
tags: [java, spring, ioc]
description: Spring IOC 容器概述
---


#### Spring IOC 依赖查找

主动或手动的依赖查找方式，需要依赖容器或标准API实现。

1. 根据 Bean 名称查找
    1. 实时查找
    ```xml
        <bean id="worker" class="com.jackhance.spring.ioc.container.model.Worker">
            <property name="id" value="9527"/>
            <property name="name" value="jackhance"/>
            <property name="age" value="25"/>
        </bean>
    ```
    ```java
        Worker worker = (Worker) beanFactory.getBean("worker");
        System.out.println("实时依赖查找：" + worker);
    ```
    实时查找，IOC 容器会返回托管在容器内部的实例对象。
    2. 延迟查找
    ```xml
        <!-- FactoryBean 的方式定义bean -->
        <bean id="myServiceFactory" class="org.springframework.beans.factory.config.ObjectFactoryCreatingFactoryBean">
            <property name="targetBeanName" value="worker"/>
        </bean>
    ```
    ```java
        ObjectFactory<Worker> myServiceFactory = (ObjectFactory<Worker>) beanFactory.getBean("myServiceFactory");
        worker = myServiceFactory.getObject();
        System.out.println("ObjectFactory 延时依赖查找：" + worker);
    ```
    延迟查找，可以通过对象工厂 FactoryBean 的方式来创建 Bean ,在调用 `ObjectFactory # getObject` 时，才进行对象创建。
2. 根据 Bean 类型查找
    1. 单个 Bean 对象
    ```java
        Worker worker = beanFactory.getBean(Worker.class);
        System.out.println("根据类型查找单个：" + worker);
    ```
    2. 集合 Bean 对象
    ```java
        if (beanFactory instanceof ListableBeanFactory) {
            ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
            Map<String, Worker> workers = listableBeanFactory.getBeansOfType(Worker.class);
            System.out.println("根据类型查找多个Bean对象：" + workers);
        }
    ```
3. 根据 Bean 名称 + 类型查找
    1. 根据 Java 注解查找
    ```java
            /**
            * 标注高级
            *
            * @author jackhance
            * @mail jackhance0825@163.com
            */
            @Target({ElementType.TYPE})
            @Retention(RetentionPolicy.RUNTIME)
            public @interface Advanced {
            }

            /**
            * 新生代工人
            *
            * @author jackhance
            * @mail jackhance0825@163.com
            */
            @Advanced
            public class AdvancedWorker extends Worker{
                private String hobby;

                public String getHobby() {
                    return hobby;
                }

                public void setHobby(String hobby) {
                    this.hobby = hobby;
                }

                @Override
                public String toString() {
                    return "AdvancedWorker{" +
                            "hobby='" + hobby + '\'' +
                            "} " + super.toString();
                }
            }
    ```
    ```java
        if (beanFactory instanceof ListableBeanFactory) {
            ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
            Map<String, Worker> workers = (Map) listableBeanFactory.getBeansWithAnnotation(Advanced.class);
            System.out.println("根据注解查找多个Bean对象：" + workers);
        }
    ```
    2. 单个 Bean 对象
    ```java
        Worker worker = beanFactory.getBean(Worker.class);
        System.out.println("根据类型查找单个：" + worker);
    ```
    3. 集合 Bean 对象
    ```java
        if (beanFactory instanceof ListableBeanFactory) {
            ListableBeanFactory listableBeanFactory = (ListableBeanFactory) beanFactory;
            Map<String, Worker> workers = listableBeanFactory.getBeansOfType(Worker.class);
            System.out.println("根据类型查找多个Bean对象：" + workers);
        }
    ```

<hr>

#### Spring IOC 依赖注入

手动或自动依赖绑定的方式，无需依赖特定的容器和API。
```java
        /**
        * 工人组
        *
        * @author jackhance
        * @mail jackhance0825@163.com
        */
        public class WorkerGroup {

                private Worker worker;

                private Worker[] workers;

                private Collection<Worker> workerCollection;

                private Environment environment;

                private ObjectFactory<BeanFactory> beanFactoryObjectFactory;

                private BeanFactory beanFactory;

                // ... getter setter
        }
```

1. 根据 Bean 名称注入
```xml
    <bean id="workerGroup" class="com.jackhance.spring.ioc.container.model.WorkerGroup" autowire="no">

       <property name="workers">
           <util:list>
                <ref bean="worker"/>
                <ref bean="advancedWorker"/>
           </util:list>
       </property>
    </bean>
```
2. 根据 Bean 类型注入
```xml
    <bean id="workerGroup" class="com.jackhance.spring.ioc.container.model.WorkerGroup" autowire="byType"> <!-- 通过类型依赖注入 -->
    </bean>
```
单个 Bean 对象 : `WorkerGroup # worker`
集合 Bean 对象 : `WorkerGroup # workerCollection`
3. 注入容器內建 Bean 对象
`WorkerGroup # environment`
`WorkerGroup # beanFactoryObjectFactory`
4. 注入非 Bean 对象
`WorkerGroup # beanFactory`
5. 注入类型
    1. 实时注入: 在执行依赖注入前，对象已创建完毕
    2. 延迟注入：依赖注入对象工厂，在通过对象工厂获取对象时，才进行对象的实例化
    `WorkerGroup # beanFactoryObjectFactory`

<hr>

#### Spring IOC 依赖来源

1. 自定义 Bean
```java
        WorkerGroup workerGroup = beanFactory.getBean("workerGroup", WorkerGroup.class);
        System.out.println("[依赖来源]自定义 Bean : " + workerGroup);
```
2. 内建依赖 Bean
```java
        ObjectFactory<BeanFactory> beanFactoryObjectFactory = workerGroup.getBeanFactoryObjectFactory();
        BeanFactory beanFactory0 = beanFactoryObjectFactory.getObject();
        System.out.println("[依赖来源]内建依赖 Bean : " + beanFactoryObjectFactory);
```
3. 容器內建 Bean
```java
        Environment env = workerGroup.getEnvironment();
        System.out.println("[依赖来源]容器內建 Bean : " + env);
```

<hr>

#### Spring IOC 配置元信息

1. Bean 定义配置
    1. 基于 XML 文件
    2. 基于 Properties 文件
    3. 基于 Java 注解
    4. 基于 Java API
2. IoC 容器配置
    1. 基于 XML 文件
    2. 基于 Java 注解
    3. 基于 Java API
3. 外部化属性配置
    1. 基于 Java 注解

这里在后续的篇章再进行展开，这里先做大概的了解。
<hr>

#### Spring IOC 容器

BeanFactory 是 Spring 底层 IoC 容器
ApplicationContext 是具备应用特性的 BeanFactory 超集
```java
        BeanFactory beanFactory = new ClassPathXmlApplicationContext("classpath:/META-INF/ioc-container-dependency-injection.xml");

        ObjectFactory<BeanFactory> beanFactoryObjectFactory = workerGroup.getBeanFactoryObjectFactory();
        BeanFactory beanFactory0 = beanFactoryObjectFactory.getObject();

        // false ， BeanFactory 非同一个对象？ why？？
        System.out.println("BeanFactory equals  : " + (beanFactory0 == beanFactory));

        // 依赖查找 BeanFactory 失败 org.springframework.beans.factory.NoSuchBeanDefinitionException  ，why？
        System.out.println(beanFactory.getBean(BeanFactory.class));
```

##### BeanFactory 非同一个对象？

我们可以找到源码 

ClassPathXmlApplicationContext -> 

AbstractXmlApplicationContext -> 

AbstractRefreshableConfigApplicationContext -> 

AbstractRefreshableApplicationContext ->

AbstractApplicationContext ->

ConfigurableApplicationContext ->

ApplicationContext ->

ListableBeanFactory ->

BeanFactory

AbstractRefreshableApplicationContext 的实现：
```java
        /** Bean factory for this context. */
        @Nullable
        private volatile DefaultListableBeanFactory beanFactory;

        @Override
        public final ConfigurableListableBeanFactory getBeanFactory() {
            DefaultListableBeanFactory beanFactory = this.beanFactory;
            if (beanFactory == null) {
                throw new IllegalStateException("BeanFactory not initialized or already closed - " +
                        "call 'refresh' before accessing beans via the ApplicationContext");
            }
            return beanFactory;
        }
```
ClassPathXmlApplicationContext 实现接口 BeanFactory ，具有 BeanFactory 接口的特性，但是并不是 BeanFactory 。ClassPathXmlApplicationContext 与 BeanFactory 的关系是以组合的形式存在。

##### 为什么依赖查找 BeanFactory 会失败？

后续篇章再做阐述。

<hr>

#### Spring 应用上下文

ApplicationContext 除了 IoC 容器角色，还有提供：
- 面向切面（AOP）
- 配置元信息（Configuration Metadata）
- 资源管理（Resources）
- 事件（Events）
- 国际化（i18n）
- 注解（Annotations）
- Environment 抽象（Environment Abstraction）


<hr>

#### Spring IOC 容器生命周期

- 启动
`AbstractApplicationContext # refresh`
- 运行
- 停止
`AbstractApplicationContext # close`

<hr>




