---
title: Spring 卷 - IOC 篇 - Spring Ioc 依赖注入
date: 2021-10-14 21:11:00 +07:00
modified: 2021-10-14 21:11:00 +07:00
tags: [java, spring, ioc, dependency, injection]
description: Spring Ioc 依赖注入详解
---


### 依赖注入的模式和类型

#### 模式

- 手动模式：通过配置或者API的方式，确定注入规则。
    - xml 资源配置元信息，实现依赖注入
    - Java 注解配置元信息，实现依赖注入
    - API 配置元信息，实现依赖注入
- 自定模式：实现方提供依赖自动关联的方式，BeanFactory 容器按照内建的注入规则，实现依赖注入。
    - Autowiring 自动绑定

#### 类型

1. Setter 方法：`<property name="worker" ref="workerBean"/>`
2. 构造器: `<constructor-arg name="id"  value="123" />`
3. 字段: `@Resource Worker worker;`
4. 方法: `@Autowire public void doWorker(Worker worker) {...}`
5. 接口回调 Aware: `class Demo implements ApplicationContextAware {...}`

<br>
<hr>

### 自动绑定（Autowiring）

什么是自动绑定呢？我从官网摘录了部分：<sup id="autowiring-collaborators">[[1]](#autowiring-collaborators-ref)</sup>

```
The Spring container can autowire relationships between collaborating beans. You can let Spring resolve collaborators (other beans) automatically for your bean by inspecting the contents of the ApplicationContext. Autowiring has the following advantages:

Autowiring can significantly reduce the need to specify properties or constructor arguments. (Other mechanisms such as a bean template discussed elsewhere in this chapter are also valuable in this regard.)

Autowiring can update a configuration as your objects evolve. For example, if you need to add a dependency to a class, that dependency can be satisfied automatically without you needing to modify the configuration. Thus autowiring can be especially useful during development, without negating the option of switching to explicit wiring when the code base becomes more stable.
```

Spring 容器可通过自动绑定 Bean 协作者之间的关系。当然这里的 Bean 并不只是我们注册到 Spring 容器内部的，还包含了内建的 Bean ，外部的配置资源等等。因此我觉得这里用协作者是比较恰当的。

Spring 的自动绑定包含了俩个优点：
1. 自动绑定显著地减少了指定属性或者构造器参数的需求。
2. 自动绑定会自适应配置。如`<property name="name" ref="jackhance"/>` ,名称为 jackhance 的 Bean 的配置变化时，这里绑定的属性会同时变化，而不像`<property name="name" value="jackhance"/>` 固化为 jackhance，一成不变。


#### 自动绑定模式

- no ： 默认值，Autowiring 未激活，需要手动绑定
- byName ：依据属性的名称作为 Bean 名称进行依赖查找，并将查找到的 Bean 进行绑定
- byType ： 依据属性的类型进行依赖查找，并将查找到的 Bean 进行绑定
- constructor ：特殊 byType 类型，用于构造器参数注入

在 Spring 中有对应的枚举，`org.springframework.beans.factory.annotation.Autowire`: 

```java
public enum Autowire {

	/**
	 * Constant that indicates no autowiring at all.
	 */
	NO(AutowireCapableBeanFactory.AUTOWIRE_NO),

	/**
	 * Constant that indicates autowiring bean properties by name.
	 */
	BY_NAME(AutowireCapableBeanFactory.AUTOWIRE_BY_NAME),

	/**
	 * Constant that indicates autowiring bean properties by type.
	 */
	BY_TYPE(AutowireCapableBeanFactory.AUTOWIRE_BY_TYPE);

    //...
}

```


#### 自动绑定的限制

我从官网摘录了部分：<sup id="limitations-and-disadvantages-of-autowiring">[[2]](#limitations-and-disadvantages-of-autowiring-ref)</sup>
- 显式指定依赖会覆盖自动绑定。
- 无法自动绑定原始类型（e.g. int、short、long）、Strings、Classes 。Spring故意这么设计的。（自动绑定本身伴随着不确定性，绑定的范围进行限定，应该是降低不确定性造成的影响）
- 相比明确的绑定，自动绑定是缺乏精确性的。使用自动绑定时，Spring 需要猜测绑定的 Bean ，如果出现歧义，会产生不想期待的结果。
- 若通过 setter 方法或者构造器注入时，匹配结果为多个 Bean ，但是方法期待的结果为单个 Bean ，这时会抛错。需要指定 `@Primary` 的 Bean。


<br>
<hr>

### Setter 方法依赖注入

Setter 方法依赖注入实现方式分为手动模式和自动模式，通过 Bean 的 setter 方法实现依赖注入。

先定义 Bean 类 WorkerHolder，提供 setter 方法，通过以下方式依赖注入 Worker：
```java
/**
 * {@link Worker} Holder 类
 *
 * @author jackhance
 * @mail jackhance0825@163.com
 */
public class WorkerHolder {

    private Worker worker;

    public void setWorker(Worker worker) {
        this.worker = worker;
    }
	// 省略其他方法、字段
}
```

#### 手动模式

##### xml 资源配置元信息方式

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="simpleWorker" class="com.jackhance.spring.ioc.dependency.injection.model.Worker">
        <property name="id" value="1"/>
        <property name="name" value="jackhance-01"/>
        <property name="age" value="25"/>
    </bean>

	<!-- 通过 setter 方法注入 worker -->
    <bean id="workerHolder" class="com.jackhance.spring.ioc.dependency.injection.model.WorkerHolder">
        <property name="worker" ref="simpleWorker"/>
    </bean>

</beans>
```

##### Java 注解配置元信息方式

```java
    /**
     * 依赖注入 {@link Worker} 对象
     */
    @Bean
    public WorkerHolder annotatedWorkerHolder(Worker worker) {
        WorkerHolder holder = new WorkerHolder();
        holder.setWorker(worker);
        return holder;
    }

```


##### API 配置元信息方式

```java
/**
 * 通过 API 配置元信息，实现 setter 方法依赖注入示例
 *
 * @author jackhance
 * @mail jackhance0825@163.com
 */
public class APIDependencySetterInjectDemo {

    public static void main(String[] args) {
        // 创建 IoC 容器
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 读取 xml 配置到 Spring 容器，加载、解析、生成 BeanDefinition
        XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

        String location = "classpath:/META-INF/ioc-dependency-injection-setter.xml";
        xmlBeanDefinitionReader.loadBeanDefinitions(location);

        // 通过 API 创建 BeanDefinition
        AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(WorkerHolder.class)
                .addPropertyReference("worker", "simpleWorker")
                .getBeanDefinition();

        // 注册 BeanDefinition 到 IoC 容器
        beanFactory.registerBeanDefinition("workerHolderFromAPI", beanDefinition);

        // 依赖查找
        WorkerHolder holder = beanFactory.getBean("workerHolderFromAPI", WorkerHolder.class);
        System.out.println(holder);
    }
}
```

#### 自动模式

##### byName

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

	<!-- 通过 setter 方法 byName 方式自动注入 -->
    <bean id="workerHolderByName" class="com.jackhance.spring.ioc.dependency.injection.model.WorkerHolder" autowire="byName"/>

	<!-- 省略其他 bean 定义 -->
</beans>
```

##### byType

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

	<!-- 通过 setter 方法 byType 方式自动注入 -->
    <bean id="workerHolderByType" class="com.jackhance.spring.ioc.dependency.injection.model.WorkerHolder" autowire="byType"/>

	<!-- 省略其他 bean 定义 -->
</beans>
```


<br>
<hr>

### 构造器依赖注入

构造器依赖注入实现方式分为手动模式和自动模式，通过 Bean 的构造方法实现依赖注入。

#### 手动模式

##### xml 资源配置元信息方式

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

	<!-- 通过 constructor 方法注入 worker -->
    <bean id="workerHolder" class="com.jackhance.spring.ioc.dependency.injection.model.WorkerHolder">
        <constructor-arg name="worker" ref="simpleWorker"/>
    </bean>

	<!-- 省略其他 bean 定义 -->
</beans>
```


##### Java 注解配置元信息方式

```java

/**
* 依赖注入 {@link Worker} 对象
*/
@Bean
public WorkerHolder annotatedWorkerHolder(Worker worker) {
	return new WorkerHolder(worker);
}
```

##### API 配置元信息方式

```java
/**
 * 通过 API 配置元信息，实现 constructor 方法依赖注入示例
 *
 * @author jackhance
 * @mail jackhance0825@163.com
 */
public class APIDependencyConstructorInjectDemo {

    public static void main(String[] args) {
        // 创建 IoC 容器
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 读取 xml 配置到 Spring 容器，加载、解析、生成 BeanDefinition
        XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

        String location = "classpath:/META-INF/ioc-dependency-injection-constructor.xml";
        xmlBeanDefinitionReader.loadBeanDefinitions(location);

        // 通过 API 创建 BeanDefinition
        AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(WorkerHolder.class)
                // 按照构造器参数顺序添加参数
                .addConstructorArgReference("primaryWorker")
                .getBeanDefinition();

        // 注册 BeanDefinition 到 IoC 容器
        beanFactory.registerBeanDefinition("workerHolderFromAPI", beanDefinition);

        // 依赖查找
        WorkerHolder holder = beanFactory.getBean("workerHolderFromAPI", WorkerHolder.class);
        System.out.println(holder);
    }

}
```

#### 自动模式

##### constructor

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

	<!-- 通过 constructor 方法 constructor 方式自动注入, 存在多个时，注入标注 primary 的 Bean -->
    <bean id="workerHolderByConstructor" class="com.jackhance.spring.ioc.dependency.injection.model.WorkerHolder" autowire="constructor"/>

	<!-- 省略其他 bean 定义 -->
</beans>
```


<br>
<hr>

### 字段依赖注入

字段依赖注入，只支持手动模式，通过 Java 注解配置元信息实现。

#### `@Autowired`

```java
	/**
     * {@link Autowired} 方式依赖注入
     */
    @Autowired
    private WorkerHolder workerHolder1;

```


#### `@Resource`

```java
    /**
     * {@link Resource} 方式依赖注入
     */
    @Resource
    private WorkerHolder workerHolder2;

```

#### `@inject`

```java
    /**
     * {@link Inject} 方式依赖注入
     */
    @Inject
    private WorkerHolder workerHolder3;
```

<br>
<hr>

### 方法依赖注入

方法依赖注入，只支持手动模式，通过 Java 注解配置元信息实现。

#### `@Autowired`

```java
    /**
     * {@link Autowired} 方式依赖注入
     */
    @Autowired
    public void init1(Worker worker) {
        this.workerHolder1 = new WorkerHolder(worker);
    }
```


#### `@Resource`

```java
    /**
     * {@link Resource} 方式依赖注入
     */
    @Resource
    public void init2(Worker worker) {
        this.workerHolder2 = new WorkerHolder(worker);
    }
```

#### `@inject`

```java
    /**
     * {@link Inject} 方式依赖注入
     */
    @Inject
    public void init3(Worker worker) {
        this.workerHolder3 = new WorkerHolder(worker);
    }
```

#### `@Bean`

```java
    /**
     * {@link Bean} 方式依赖注入，并注册 workerHolder 实例到 Spring 容器
     */
    @Bean
    public WorkerHolder workerHolder(Worker worker) {
        return new WorkerHolder(worker);
    }
```

<br>
<hr>

### 回调依赖注入

Spring 接口回调，通过 `Aware` 接口实现，组件类可以通过实现接口，实现注入指定资源。

|       `Aware` 接口      |      获取的资源       |
| :---------------------- | :-------------------: |
| BeanFactoryAware | 获取 IoC 容器 BeanFactory |
| ApplicationContextAware | 获取 Spring 应用上下文 ApplicationContext 对象 |
| EnvironmentAware | 获取 Environment 对象 |
| ResourceLoaderAware | 获取资源加载器对象 ResourceLoader |
| BeanClassLoaderAware | 获取加载当前 Bean Class 的 ClassLoader |
| BeanNameAware | 获取当前 Bean 的名称 |
| MessageSourceAware | 获取 MessageSource 对象，用于 Spring 国际化 |
| ApplicationEventPublisherAware | 获取 ApplicationEventPublisher 对象，用于 Spring 事件 |
| EmbeddedValueResolverAware | 获取 StringValueResolve 对象，用于占位符处理 |

```java
/**
 * 基于 {@link Aware} 接口回调的依赖注入示例
 *
 * @author jackhance
 * @mail jackhance0825@163.com
 */
@Configuration
public class AwareInterfaceInjectDemo implements BeanFactoryAware, ApplicationContextAware, ResourceLoaderAware {

    private BeanFactory beanFactory;

    private ApplicationContext applicationContext;

    private ResourceLoader resourceLoader;

    public static void main(String[] args) {
        // 创建应用上下文
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();

        // 注册注册类
        applicationContext.register(AwareInterfaceInjectDemo.class);

        // 启动应用上下文
        applicationContext.refresh();

        AwareInterfaceInjectDemo demo = applicationContext.getBean(AwareInterfaceInjectDemo.class);

        System.out.printf("beanFactory equals : %s, beanFactory = %s%n", (demo.beanFactory == applicationContext.getBeanFactory()), demo.beanFactory);

        System.out.printf("applicationContext equals : %s, applicationContext = %s%n", (demo.applicationContext == applicationContext), demo.applicationContext);

        System.out.println("resourceLoader = " + demo.resourceLoader);

        // 关闭应用上下文
        applicationContext.close();
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }
}

```


<br>
<hr>

### 依赖注入类型选择

低依赖：构造器注入

多依赖：Setter 方法注入

便利性：字段注入

声明类：方法注入



<br>
<hr>

### 基础类型注入







<br>
<hr>

### 集合类型注入







<br>
<hr>

### 限定注入

```java
/**
 * 组注解，基于 {@link Qualifier}
 *
 * @author jackhance
 * @mail jackhance0825@163.com
 */
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
@Qualifier
public @interface Group {
}
```

```java
    @Autowired
    public Worker worker;

    @Autowired
    @Qualifier("worker4")
    public Worker qualifierWorker;

    @Autowired
    public Collection<Worker> workers;

    @Autowired
    @Qualifier
    public Collection<Worker> qualifierWorkers;

    @Autowired
    @Group
    public Collection<Worker> groupWorkers;

    /**
     * worker1、worker2
     * 逻辑分组 @Qualifier ： worker3、worker4
     * 自定义逻辑分组 @Group ： worker5 、 worker6
     */

    @Bean
    public Worker worker1() {
        return buildWorker("1", "jackhance", 30);
    }

    @Bean
    @Primary
    public Worker worker2() {
        return buildWorker("2", "jackhance", 30);
    }

    /**
     * 基于 {@link Qualifier} 逻辑分组
     */
    @Bean
    @Qualifier
    public Worker worker3() {
        return buildWorker("3", "jackhance", 30);
    }

    /**
     * 基于 {@link Qualifier} 逻辑分组
     */
    @Bean
    @Qualifier
    public Worker worker4() {
        return buildWorker("4", "jackhance", 30);
    }

    /**
     * 自定义逻辑分组
     */
    @Bean
    @Group
    public Worker worker5() {
        return buildWorker("5", "jackhance", 30);
    }

    /**
     * 自定义逻辑分组
     */
    @Bean
    @Group
    public Worker worker6() {
        return buildWorker("6", "jackhance", 30);
    }
```


<br>
<hr>

### 延迟依赖注入

```java
    /**
     * {@link Autowired} {@link Lazy}延迟注入
     */
    @Autowired
    @Lazy
    private Worker lazyWorker1;

    /**
     * {@link Inject} {@link Lazy}延迟注入
     */
    @Inject
    @Lazy
    private Worker lazyWorker2;

    /**
     * {@link Resource} {@link Lazy}延迟注入
     */
    @Resource
    @Lazy
    private Worker lazyWorker3;

    /**
     * {@link ObjectProvider} 延迟注入
     */
    @Autowired
    private ObjectProvider<Worker> objectProvider;

    /**
     * {@link ObjectFactory} 延迟注入
     */
    @Autowired
    private ObjectFactory<Worker> objectFactory;
```

<br>
<hr>

### `@Autowired` 注入原理



<br>
<hr>

### Java 通用注解注入原理

CommonAnnotationBeanPostProcessor


<br>
<hr>

### `@Inject` 注入原理



<br>
<hr>

### 自定义依赖注入



<br>
<hr>

### 小记

<small id="autowiring-collaborators-ref"><sup>[[1]](#autowiring-collaborators)</sup> <a href="https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire" target="_blank" > Autowiring Collaborators </a></small> 

<small id="limitations-and-disadvantages-of-autowiring-ref"><sup>[[1]](#limitations-and-disadvantages-of-autowiring)</sup> <a href="https://docs.spring.io/spring-framework/docs/5.2.16.RELEASE/spring-framework-reference/core.html#beans-autowired-exceptions" target="_blank" > limitations and disadvantages of autowiring </a></small> 

<br>
<hr>

### 代码

<a href="https://github.com/jackhance0825/thinking-in-spring-5.2.16/tree/main/ioc-dependency-injection" target="_blank" > 本篇章代码 </a>
