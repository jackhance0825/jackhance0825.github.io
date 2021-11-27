---
title: Spring 卷 - IOC 篇 - Spring Bean 作用域
date: 2021-11-24 20:57:00 +07:00
modified: 2021-11-24 20:57:00 +07:00
tags: [java, spring, ioc, bean, scope]
description: Spring Bean 作用域
---


### Spring Bean 的作用域

| 作用域   |                        描述                        |
| :------ | :------------------------------------------------: |
| singleton |  默认，同一 BeanFactory 仅能存在一个 Bean 实例      |
| prototype |  原型，每次依赖查找和依赖注入都会生成新的 Bean 实例  |
| request   |  将 Bean 实例存储在 ServletRequest 上下文          |
| session   |  将 Bean 实例存储在 HttpSession 上下文            |
| application | 将 Bean 实例存储在 ServletContext 上下文          |
| websocket  |  将 Bean 实例的生命周期伴随整个 WebSocket Session   |

<br>
<hr>

### singleton 作用域

如果你对 Bean 的作用域定义为单例，那么当 Spring IoC 容器创建该 Bean 实例时，只会创建单个实例对象，并管理单例对象的生命周期。

该单例对象存储在此类单例 Bean 的缓存中，并且对该命名 bean 的所有后续请求和引用都返回缓存对象。

这里的单例是区别于 GoF 设计模式的单例模式。GoF 单例设计模式，规定每个类加载器 ClassLoader 中，只会创建该类的一个实例对象。而 Spring singleton 作用域，同一 Spring IoC 容器对该类创建实例有且仅有一个。

`org.springframework.beans.factory.FactoryBean` 存在单例的方法定义：

```java

public interface FactoryBean<T> {
    
    // 省略部分代码...

	/**
	 * Is the object managed by this factory a singleton? That is,
	 * will {@link #getObject()} always return the same object
	 * (a reference that can be cached)?
	 * <p><b>NOTE:</b> If a FactoryBean indicates to hold a singleton object,
	 * the object returned from {@code getObject()} might get cached
	 * by the owning BeanFactory. Hence, do not return {@code true}
	 * unless the FactoryBean always exposes the same reference.
	 * <p>The singleton status of the FactoryBean itself will generally
	 * be provided by the owning BeanFactory; usually, it has to be
	 * defined as singleton there.
	 * <p><b>NOTE:</b> This method returning {@code false} does not
	 * necessarily indicate that returned objects are independent instances.
	 * An implementation of the extended {@link SmartFactoryBean} interface
	 * may explicitly indicate independent instances through its
	 * {@link SmartFactoryBean#isPrototype()} method. Plain {@link FactoryBean}
	 * implementations which do not implement this extended interface are
	 * simply assumed to always return independent instances if the
	 * {@code isSingleton()} implementation returns {@code false}.
	 * <p>The default implementation returns {@code true}, since a
	 * {@code FactoryBean} typically manages a singleton instance.
	 * @return whether the exposed object is a singleton
	 * @see #getObject()
	 * @see SmartFactoryBean#isPrototype()
	 */
	default boolean isSingleton() {
		return true;
	}
}
```

默认是单例对象，当 `FactoryBean` 调用 `getObject()` 获取实例对象时，若为单例，则都会返回相同的实例对象。

Bean 的元信息 `org.springframework.beans.factory.config.BeanDefinition`，存在接口，返回 Bean 是否为单例对象：


```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

    // 省略部分代码...

	/**
	 * Return whether this a <b>Singleton</b>, with a single, shared instance
	 * returned on all calls.
	 * @see #SCOPE_SINGLETON
	 */
	boolean isSingleton();
}
```

#### xml 定义

```xml
    <!-- 默认 scope 为 singleton -->
    <bean id="worker1" class="com.jackhance.spring.ioc.beanscope.model.Worker">
        <constructor-arg name="id"  value="1" />
        <constructor-arg name="name" value="jackhance"/>
    </bean>

    <bean id="worker2" class="com.jackhance.spring.ioc.beanscope.model.Worker" scope="singleton">
        <constructor-arg name="id"  value="2" />
        <constructor-arg name="name" value="jackhance"/>
    </bean>
```

#### java 注解定义

```java

    /**
     * 默认 singleton,
     * {@code @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)} 可不指定
     */
    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
    private static Worker singletonWorker() {
        return new Worker(System.nanoTime(), "jackhance");
    }

```

#### Singleton 作用域 Bean 的特点

1. Singleton Bean 依赖查找与依赖注入，均属同一 Bean
2. 依赖注入到集合类型对象中，Singleton Bean 只会存在一个
3. Spring 管理 Singleton Bean 完整的生命周期


<br>
<hr>

### prototype 作用域

当 Spring Bean 的作用域定义为 prototype 时，依赖注入或者依赖查找该 Bean ，每次都会创建新的 Bean 实例。prototype 作用域与 GoF 原型模式相似。

`prototype 作用域更适合有状态 Bean，而 Singleton 作用域更适合无状态 Bean。`

Spring IoC 容器对 prototype 作用域的 Bean 实例化、配置、组装，生成原型对象，交给请求方，Spring IoC 容器并没有对原型对象做进一步的记录，因此，prototype 作用域的 Bean 并不会触发销毁方法，需要用户自行销毁清理。


Bean 的元信息 `org.springframework.beans.factory.config.BeanDefinition`，存在接口，返回 Bean 是否为原型对象：


```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {

    // 省略部分代码...

	/**
	 * Return whether this a <b>Prototype</b>, with an independent instance
	 * returned for each call.
	 * @since 3.0
	 * @see #SCOPE_PROTOTYPE
	 */
	boolean isPrototype();
}
```

#### xml 定义

```xml
<bean id="worker1" class="com.jackhance.spring.ioc.beanscope.model.Worker" scope="prototype">
        <constructor-arg name="id"  value="1" />
        <constructor-arg name="name" value="jackhance"/>
    </bean>

```


#### java 注解定义

```java
@Bean
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
private static Worker prototypeWorker() {
    return new Worker(System.nanoTime(), "jackhance");
}

```

#### prototype 作用域 Bean 的特点

1. Prototype Bean 依赖查找与依赖注入，均生成新对象
2. 每个依赖注入的 Prototype Bean 都不一样，是新的对象
3. Spring 管理 Prototype Bean 初始化生命周期，prototype 作用域的 Bean 并不会触发销毁方法，需要用户自行销毁清理。


<br>
<hr>

### request 作用域

```java

@Configuration
public class WebConfiguration {

    @Bean
    @RequestScope
    public Worker worker() {
        return new Worker(1, "jackhance");
    }
}

@Controller
public class IndexController {

    /**
     * CGLIB 代理后对象（不变的）
     */
    @Autowired
    private Worker worker;

    @GetMapping("/index")
    public String index(Model model) {
        model.addAttribute("workerObject", worker);
        return "index";
    }
}
```

1. Bean 名称为 worker 标注上 @RequestScope ，在 BeanFactory 里生成 worker 的 BeanDefinition
2. 通过 ScopedProxyFactoryBean 生成生成 CGLIB 提升后的代理对象，依赖注入到 IndexController#worker
3. 当操作对象 IndexController#worker 时，查询 RequestScope 请求上下文 RequestContextHolder.currentRequestAttributes() 是否包含名为 scopedTarget.worker 的作用域对象，有则返回，反之则创建
4. 创建时，执行初始化流程
5. 请求结束时，执行销毁

<br>
<hr>

### session 作用域

```java

@Configuration
public class WebConfiguration {

    @Bean
    @SessionScope
    public Worker worker() {
        return new Worker(1, "jackhance");
    }
}

@Controller
public class IndexController {

    /**
     * CGLIB 代理后对象（不变的）
     */
    @Autowired
    private Worker worker;

    @GetMapping("/index")
    public String index(Model model) {
        model.addAttribute("workerObject", worker);
        return "index";
    }
}
```

1. Bean 名称为 worker 标注上 @SessionScope ，在 BeanFactory 里生成 worker 的 BeanDefinition
2. 通过 ScopedProxyFactoryBean 生成生成 CGLIB 提升后的代理对象，依赖注入到 IndexController#worker
3. 当操作对象 IndexController#worker 时，查询 SessionScope 请求上下文 RequestContextHolder.currentRequestAttributes() 是否包含名为 scopedTarget.worker 的作用域对象，有则返回，反之则创建
4. 创建时，执行初始化流程
5. 会话结束时，执行销毁

<br>
<hr>

### application 作用域


```java

@Configuration
public class WebConfiguration {

    @Bean
    @ApplicationScope
    public Worker worker() {
        return new Worker(1, "jackhance");
    }
}

@Controller
public class IndexController {

    /**
     * CGLIB 代理后对象（不变的）
     */
    @Autowired
    private Worker worker;

    @GetMapping("/index")
    public String index(Model model) {
        model.addAttribute("workerObject", worker);
        return "index";
    }
}
```

1. Bean 名称为 worker 标注上 @ApplicationScope ，在 BeanFactory 里生成 worker 的 BeanDefinition
2. 通过 ScopedProxyFactoryBean 生成生成 CGLIB 提升后的代理对象，依赖注入到 IndexController#worker
3. 当操作对象 IndexController#worker 时，查询 ServletContextScope 请求上下文 RequestContextHolder.currentRequestAttributes() 是否包含名为 scopedTarget.worker 的作用域对象，有则返回，反之则创建
4. 创建时，执行初始化流程
5. 应用生命周期结束时，执行销毁

<br>
<hr>

### 小记

<small id="xsd-id-ref"><sup>[[1]](#xsd-id)</sup> <a href="https://github.com/spring-projects/spring-framework/blob/3.0.x/org.springframework.beans/src/main/resources/org/springframework/beans/factory/xml/spring-beans-3.0.xsd" target="_blank" > xsd-ID 声明 </a></small> 

### 代码

<a href="https://github.com/jackhance0825/thinking-in-spring-5.2.16/tree/main/bean-scope" target="_blank" > 本篇章代码 </a>
