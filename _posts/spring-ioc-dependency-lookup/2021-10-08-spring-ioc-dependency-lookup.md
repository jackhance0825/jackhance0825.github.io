---
title: Spring 卷 - IOC 篇 - Spring Ioc 依赖查找
date: 2021-10-08 21:13:00 +07:00
modified: 2021-10-08 21:13:00 +07:00
tags: [java, spring, ioc, dependency, lookup]
description: Spring Ioc 依赖查找详解
---


### 依赖查找的起源

Spring 的依赖查找，可以从 JavaBeans 看到身影。Spring 的依赖查找，很大程度地借鉴了 JavaBeans 的实现。

#### 单一类型查找

`java.beans.beancontext.BeanContext` 通过 JavaBean 的名称来实例化：

```java
public interface BeanContext extends BeanContextChild, Collection, DesignMode, Visibility {

    /**
     * Instantiate the javaBean named as a
     * child of this <code>BeanContext</code>.
     * The implementation of the JavaBean is
     * derived from the value of the beanName parameter,
     * and is defined by the
     * <code>java.beans.Beans.instantiate()</code> method.
     *
     * @return a javaBean named as a child of this
     * <code>BeanContext</code>
     * @param beanName The name of the JavaBean to instantiate
     * as a child of this <code>BeanContext</code>
     * @throws IOException if an IO problem occurs
     * @throws ClassNotFoundException if the class identified
     * by the beanName parameter is not found
     */
    Object instantiateChild(String beanName) throws IOException, ClassNotFoundException;

    //...
}
```

#### 集合类型查找

`java.beans.beancontext.BeanContextServices` 通过指定类型来查找所有服务：

```java
public interface BeanContextServices extends BeanContext, BeanContextServicesListener {

    //...

    /**
     * Gets the list of service dependent service parameters
     * (Service Selectors) for the specified service, by
     * calling getCurrentServiceSelectors() on the
     * underlying BeanContextServiceProvider.
     * @param serviceClass the specified service
     * @return the currently available service selectors
     * for the named serviceClass
     */
    Iterator getCurrentServiceSelectors(Class serviceClass);

    //...
}
```

#### 层次性查找

`java.beans.beancontext.BeanContextChild` 通过 `setBeanContext` 实现了层次性的实现，为第三方提供了扩展，可添加一个 `BeanContextChild` 到目标的 `BeanContext`。

详见官方文档<sup id="bean-context-hierarchical">[[1]](#bean-context-hierarchical-ref)</sup>

<figure>
<img src="/spring-ioc-dependency-lookup/beancontext.gif" alt="beanContext">
<figcaption>Fig 1. beanContext.</figcaption>
</figure>

<br>


```java
public interface BeanContextChild {

    /**
     * <p>
     * Objects that implement this interface,
     * shall fire a java.beans.PropertyChangeEvent, with parameters:
     *
     * propertyName "beanContext", oldValue (the previous nesting
     * <code>BeanContext</code> instance, or <code>null</code>),
     * newValue (the current nesting
     * <code>BeanContext</code> instance, or <code>null</code>).
     * <p>
     * A change in the value of the nesting BeanContext property of this
     * BeanContextChild may be vetoed by throwing the appropriate exception.
     * </p>
     * @param bc The <code>BeanContext</code> with which
     * to associate this <code>BeanContextChild</code>.
     * @throws PropertyVetoException if the
     * addition of the specified <code>BeanContext</code> is refused.
     */
    void setBeanContext(BeanContext bc) throws PropertyVetoException;

    /**
     * Gets the <code>BeanContext</code> associated
     * with this <code>BeanContextChild</code>.
     * @return the <code>BeanContext</code> associated
     * with this <code>BeanContextChild</code>.
     */
    BeanContext getBeanContext();

    // ...
}
```

<br>
<hr>

### 单一类型依赖查找

Spring 提供单一类型依赖查找，实现接口: `BeanFactory`

#### 根据 Bean 名称查找

`BeanFactory # getBean(String)`

```java
// Bean 的名称为 worker1
Worker worker1 = (Worker) beanFactory.getBean("worker1");
```

<br>


`BeanFactory # getBean(String, Object...) `

Spring 2.5 覆盖默认参数，参数 `Object...` 用于使用工厂方法或者构造方法构造 Bean 实例，若 Bean 是 singleton 并已提前构造，此参数会被忽略，直接返回已构造的实例 Bean。

```java
// Bean 的名称为 worker2
Worker worker2 = (Worker) beanFactory.getBean("worker2", 3, "jay");
```

<br>

#### 根据 Bean 类型查找

`BeanFactory # getBean(Class)` 实时查找

```java
Worker workerByType = beanFactory.getBean(Worker.class);
```

<br>

`BeanFactory # getBean(Class, Object...)` 实时查找

Spring 4.1 覆盖默认参数，参数 `Object...` 用于使用工厂方法或者构造方法构造 Bean 实例，若 Bean 是 singleton 并已提前构造，此参数会被忽略，直接返回已构造的实例 Bean。

```java
Worker workerArgsByType = (Worker) beanFactory.getBean(Worker.class, 3, "jay");
```

<br>

`BeanFactory # getBeanProvider(Class)` Spring 5.1 Bean 延迟查找

Bean 的延迟查找并不会发生实例化，方法返回 ObjectProvider 对象，Bean 的实例化会发生在首次获取。

```java
ObjectProvider<Worker> beanProvider = beanFactory.getBeanProvider(Worker.class);
```

<br>

`BeanFactory # getBeanProvider(ResolvableType)` Spring 5.1 Bean 延迟查找

Bean 的延迟查找并不会发生实例化，方法返回 ObjectProvider 对象，Bean 的实例化会发生在首次获取。

```java
ResolvableType resolvableType = ResolvableType.forClass(Worker.class);
ObjectProvider<Worker> beanProvider = beanFactory.getBeanProvider(resolvableType);
```

<br>

#### 根据 Bean 名称 + 类型查找

`BeanFactory # getBean(String, Class)`

```java
Worker worker1 = beanFactory.getBean("worker1", Worker.class);
```

<br>
<hr>

### 集合类型依赖查找

Spring 提供集合类型依赖查找，实现接口 - `ListableBeanFactory`

#### 根据 Bean 类型查找

获取同类型 Bean 名称列表

`ListableBeanFactory # getBeanNamesForType(Class)`

```java
String[] beanNamesForType = beanFactory.getBeanNamesForType(Worker.class);
```

<br>

`ListableBeanFactory # getBeanNamesForType(ResolvableType)`

```java
ResolvableType resolvableType = ResolvableType.forClass(Worker.class);
beanNamesForType = beanFactory.getBeanNamesForType(resolvableType);
```

<br>

获取同类型 Bean 实例列表
`ListableBeanFactory # getBeansOfType(Class)` 以及重载方法

```java
Map<String, Worker> beansOfType = beanFactory.getBeansOfType(Worker.class);
```

<br>

#### 通过注解类型查找

Spring 3.0 获取标注类型 Bean 名称列表
`ListableBeanFactory # getBeanNamesForAnnotation(Class<? extends Annotation>)`

```java
String[] beanNamesForAnnotation = beanFactory.getBeanNamesForAnnotation(IBean.class);
```

<br>

Spring 3.0 获取标注类型 Bean 实例列表
`ListableBeanFactory # getBeansWithAnnotation(Class<? extends Annotation>)`

```java
Map<String, Object> beansWithAnnotation = beanFactory.getBeansWithAnnotation(IBean.class);
```

<br>

Spring 3.0 获取指定名称 + 标注类型 Bean 实例
`ListableBeanFactory # findAnnotationOnBean(String, Class<? extends Annotation>)`

```java
IBean annotationOnBean = beanFactory.findAnnotationOnBean("worker3", IBean.class);
```

<br>
<hr>

### 层次性类型依赖查找

Spring 提供层次性类型依赖查找，实现接口 - `HierarchicalBeanFactory`

双亲委派模型 `HierarchicalBeanFactory # getParentBeanFactory`，与 `ClassLoader` 的双亲委派模型相似


DefaultListableBeanFactory ->

AbstractAutowireCapableBeanFactory ->

AbstractBeanFactory # containsBean

```java
@Override
public boolean containsBean(String name) {
    String beanName = transformedBeanName(name);
    if (containsSingleton(beanName) || containsBeanDefinition(beanName)) {
        return (!BeanFactoryUtils.isFactoryDereference(name) || isFactoryBean(name));
    }
    // Not found -> check parent.
    BeanFactory parentBeanFactory = getParentBeanFactory();
    return (parentBeanFactory != null && parentBeanFactory.containsBean(originalBeanName(name)));
}
```

从上面的逻辑，我们可以看到，BeanFactory 查找 Bean 时，优先查找本地是否包含此 Bean，若没有，则委派给父 BeanFactory 去查找 Bean，递归，以此类推，递归到根 BeanFactory ，返回是否包含 Bean。


根据 Bean 名称查找 : 
`BeanFactory # containsBean(String)`

基于 containsLocalBean 方法实现根据 Bean 名称查找 : 
`BeanFactory # containsLocalBean(String)`


单一类型，根据 Bean 类型查找实例列表：
`<T> T BeanFactoryUtils # beanOfType(ListableBeanFactory, Class<T>)`

集合类型，根据 Bean 类型查找实例列表：
`<T> Map<String, T> BeanFactoryUtils # beansOfTypeIncludingAncestors(ListableBeanFactory, Class<T>)`

根据 Bean 类型查找名称列表
`String[] BeanFactoryUtils # beanNamesForTypeIncludingAncestors(ListableBeanFactory, Class<?>)`

<br>
<hr>

### 延迟依赖查找

Spring 通过 `org.springframework.beans.factory.ObjectProvider` 提供了延迟查找的功能，只有通过 `ObjectProvider` 获取 Bean 实例时，Bean 才可以进行实例初始化。

一般来说，在应用启动时，应用的所有 Bean 进行实例化初始化，以便于及早发现问题，而不用等待几个小时、几天或者半个月下来，才发现问题。假若应用的启动很慢，造成开发与维护的极大不便，延迟查找是一种不错的选择。

```java
/**
* 通过 {@link ObjectProvider} 延迟查找
*/
private static void lookupByObjectProvider(AnnotationConfigApplicationContext applicationContext) {
    ObjectProvider objectProvider = applicationContext.getBeanProvider(String.class);
    System.out.println(objectProvider.getObject());
}
```

同时，Spring 5 对 Java 8 特性扩展

函数式接口：
`ObjectProvider # getIfAvailable(Supplier)`
`ObjectProvider # ifAvailable(Consumer)`

```java
/**
* 通过 {@link ObjectProvider#getIfAvailable(Supplier)} 延迟查找
*/
private static void lookupByObjectProviderIfAvailable(AnnotationConfigApplicationContext applicationContext) {
    ObjectProvider objectProvider = applicationContext.getBeanProvider(Worker.class);
    System.out.println(objectProvider.getIfAvailable(() -> new Worker(1, "jackhance")));

    objectProvider.ifAvailable(System.out::println);
}
```

Stream 扩展：
`ObjectProvider # stream()`

```java
private static void lookupByObjectProviderToStream(AnnotationConfigApplicationContext applicationContext) {
    ObjectProvider objectProvider = applicationContext.getBeanProvider(String.class);
    objectProvider.stream().forEach(System.out::println);
}
```

<br>
<hr>

### 安全依赖查找

| 依赖查找类型             | 实现                                |   安全与否            |
| :---------------------- | :--------------------------------- | :-------------------: |
| 单一类型查找             | `BeanFactory # getBean`            | no                    |
|                         | `ObjectFactory # getObject`        | no                    |
|                         | `ObjectProvider # getIfAvailable`  | yes                   |
| 集合类型查找             | `ListableBeanFactory # getBeansOfType`| yes                   |
|                         | `ObjectProvider # stream`          | yes                   |


层次性依赖查找的安全性取决于其扩展的单一或集合类型的 BeanFactory 接口

<br>
<hr>

### 依赖查找的常规异常

| 异常类型             | 触发条件                                |   场景举例            |
| :---------------------- | :--------------------------------- | :-------------------: |
| NoSuchBeanDefinitionException |当查找 Bean 不存在于 IoC 容器时   | BeanFactory#getBean ObjectFactory#getObject   |
| NoUniqueBeanDefinitionException  | 类型依赖查找时，IoC 容器存在多个 Bean 实例  |BeanFactory#getBean(Class)   |
| BeanInstantiationException | 当 Bean 所对应的类型非具体类时  |BeanFactory#getBean |
| BeanCreationException | 当 Bean 初始化过程中 | Bean 初始化方法执行异常时 |
| BeanDefinitionStoreException | 当 BeanDefinition 配置元信息非法时 | XML 配置资源无法打开时 |

<br>
<hr>



### 小记

<small id="bean-context-hierarchical-ref"><sup>[[1]](#bean-context-hierarchical)</sup> <a href="https://docs.oracle.com/javase/8/docs/technotes/guides/beans/spec/beancontext.html" target="_blank" > Extensible Runtime Containment and Services Protocol for JavaBeans Version 1.0 </a></small> 


### 代码

<a href="https://github.com/jackhance0825/thinking-in-spring-5.2.16/tree/main/ioc-dependency-lookup" target="_blank" > 本篇章代码 </a>
