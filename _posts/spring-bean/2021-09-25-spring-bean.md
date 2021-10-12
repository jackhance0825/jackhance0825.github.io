---
title: Spring 卷 - IOC 篇 - Spring Bean 初探
date: 2021-09-25 14:59:00 +07:00
modified: 2021-09-25 14:59:00 +07:00
tags: [java, spring, ioc, bean]
description: Spring Bean 初探
---


### Spring Bean 的定义

每个 Spring Ioc 容器管理一个或者多个 Bean。这些 Bean 皆通过补充到 Ioc 容器的配置元信息，从而进行创建。（e.g. xml 配置文件的 `<bean/>` ）。

在容器内部，Bean 的定义以 BeanDefinition 对象存在，其中 BeanDefinition 包含以下元信息: 
- 含包名的类名：Bean 实现类名
- Bean 的行为配置：作用域 scope 、 生命周期回调 lifecycle callback 等等
- Bean 工作时协同工作的协作方(依赖方)
- 其他配置：e.g. 数据库连接池的大小

<br>
<hr>

### BeanDefinition 的元信息

#### BeanDefinition 元信息包含：
- Class：Bean 的类全名（非接口、非抽象类）
- Name：Bean 的名称或者id
- Scope：Bean 的作用域（singleton、prototype）
- Constructor arguments：Bean 的构造器参数（依赖注入）
- Properties：Bean 的属性设置
- Autowiring mode：Bean 的自动绑定模式（byName、byType）
- Lazy initialization mode：Bean 的延迟初始化模式
- Initialization method：Bean 的初始化回调方法名称
- Destruction method：Bean 的销毁回调方法名称`

#### BeanDefinition 的构造

通过 BeanDefinitionBuilder 构建

```java
BeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Worker.class)
        // 属性设置
        .addPropertyValue("id", "worker-1")
        .addPropertyValue("name", "jackhance")
        .addPropertyValue("age", 30)
        // 获取 BeanDefinition 实例
        .getBeanDefinition();
```

通过 AbstractBeanDefinition 以及派生类构建

```java
GenericBeanDefinition genericBeanDefinition = new GenericBeanDefinition();
genericBeanDefinition.setBeanClass(Worker.class);

// 属性设置
MutablePropertyValues mutablePropertyValues = new MutablePropertyValues()
        .add("id", "worker-1")
        .add("name", "jackhance")
        .add("age", 30);
genericBeanDefinition.setPropertyValues(mutablePropertyValues);
```

<br>
<hr>

### Spring Bean 的命名

每个 bean 都有一个或多个标识符。这些标识符在承载 bean 的容器中必须是唯一的。一个 bean 通常只有一个标识符。但是，如果它需要多个，则可以将多余的视为别名(Alias)。

在基于 XML 的配置元数据中，您可以使用id属性、name属性或两者来指定 bean 标识符。该id属性允许您指定一个 ID。

作为历史记录，在 Spring 3.1 之前的版本中，id属性被定义为一种`xsd:ID`<sup id="xsd-id">[[1]](#xsd-id-ref)</sup>类型，它限制了可能的字符。从 3.1 开始，它被定义为一种`xsd:string`<sup id="xsd-string">[[2]](#xsd-string-ref)</sup>类型。请注意，bean 的id唯一性仍然由容器强制执行，但不再由 XML 解析器强制执行。

定义指定 id 的 bean：
```java
BeanDefinitionRegistry # registerBeanDefinition(String , BeanDefinition );
```

倘若你不为 bean 提供一个 `name` 或 一个 `id` ，容器会为该 bean 生成一个唯一的名称。
Spring 2.0.3 引入了接口 `BeanNameGenerator` 来为 bean生成唯一的名称。
```java
/**
 * Strategy interface for generating bean names for bean definitions.
 *
 * @author Juergen Hoeller
 * @since 2.0.3
 */
public interface BeanNameGenerator {

	/**
	 * Generate a bean name for the given bean definition.
	 * @param definition the bean definition to generate a name for
	 * @param registry the bean definition registry that the given definition
	 * is supposed to be registered with
	 * @return the generated bean name
	 */
	String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry);
}
```

`BeanNameGenerator` 有俩个默认的实现：
- `DefaultBeanNameGenerator` 默认的实现
委派给 `BeanDefinitionReaderUtils#generateBeanName(BeanDefinition, BeanDefinitionRegistry)` 命名 bean
- `AnnotationBeanNameGenerator` 基于注解扫描的实现，起始于 Spring 2.5（与 `@Component` 同起始版本）


<br>
<hr>

### Spring Bean 的别名

如果要为 bean 引入其他别名，也可以在name 属性中指定它们，用半角逗号 (,)、分号 (;) 或空格( )分隔。

#### 使用xml定义：
```xml
<alias name="myWorker" alias="systemA-worker"/>
<alias name="myWorker" alias="systemB-worker"/>
```

现在，每个组件和主应用程序都可以通过一个唯一的名称来引用，并且保证不会与任何其他定义发生冲突（有效地创建一个命名空间），但它们引用的是同一个 bean。

#### 使用注解定义：
```java
// 定义一个 id 为 worker 的 bean ，同时拥有别名 jackhance-worker
@Bean(name = {"worker", "jackhance-worker"})
public Worker worker() {
    Worker worker = new Worker();
    worker.setId("worker-02");
    worker.setName("jackhance");
    worker.setAge(30);
    return worker;
}
```

<br>
<hr>

### Spring Bean 的注册


#### xml 配置元信息注册
```xml
<bean id="worker" class="com.jackhance.spring.ioc.container.model.Worker">
    <property name="id" value="9527"/>
    <property name="name" value="jackhance"/>
    <property name="age" value="25"/>
</bean>
```

#### java 注解注册

- `@Component` 及其"派生"注解 (e.g. `@Service`、 `@Controller` 等等)
- `@Bean`
- `@Import`


#### java API 注册

- 命名方式：`BeanDefinitionRegistry # registerBeanDefinition(String, BeanDefinition)`
- 非命名方式：`BeanDefinitionReaderUtils # registerWithGeneratedName(AbstractBeanDefinition, BeanDefinitionRegistry)`
- 配置类方式：`AnnotatedBeanDefinitionReader # register(Class...)`


#### 外部单例对象注册

`SingletonBeanRegistry # registerSingleton`

外部单例对象注册到 beanFactory ，作为 bean 托管到 beanFactory 

可通过依赖注入或者依赖查找获得 bean ，但是 beanFactory 并不管理 bean 的生命周期


<br>
<hr>


### Spring Bean 的实例化

#### 构造器方式实例化

方式1 ： 通过注解构造器的方式，实例化 bean

```java
@Bean(name = "worker-create-by-annotated-constructor")
public Worker workerCreateByAnnotatedConstructor() {
    Worker worker = new Worker();
    worker.setId("pc9528");
    worker.setName("jackhance");
    worker.setAge(30);
    return worker;
}
```

方式2 ： 通过 xml 配置构造器的方式，实例化 bean

```xml
<bean id="worker-create-by-xml-constructor" class="com.jackhance.spring.ioc.container.model.Worker">
    <property name="id" value="pc9527"/>
    <property name="name" value="jackhance"/>
    <property name="age" value="30"/>
</bean>

```

#### 静态方法方式实例化

通过指定静态工厂方法，来实例化 bean ：

```java
public class Worker {
    private String id;
    private String name;
    private int age;

    ... getter or setter

    public static Worker generateWorker() {
        Worker worker = new Worker();
        worker.setId("9527");
        worker.setName("jackhance");
        worker.setAge(30);
        return worker;
    }
}
```

```xml
<bean id="worker-create-by-xml-static-method" class="com.jackhance.spring.ioc.container.model.Worker" factory-method="generateWorker"/>
```


#### 工厂方法方式实例化

通过指定工厂方法的方式进行实例化：

GenericWorkerFactory -> WorkerFactory # createWorker :
```java
public interface WorkerFactory {

    default Worker createWorker() {
        return Worker.generateWorker();
    }

}
```

```xml
<bean id="worker-factory" class="com.jackhance.spring.ioc.bean.model.GenericWorkerFactory"/>

<bean id="worker-create-by-xml-factory-method" factory-bean="worker-factory" factory-method="createWorker"/>
```

#### FactoryBean 方式实例化

```java
/**
 * {@link Worker} bean 的 {@link FactoryBean} 实现
 *
 * @author jackhance
 * @mail jackhance0825@163.com
 * @date 2021/9/29 0:13
 */
public class WorkerFactoryBean implements FactoryBean {
    @Override
    public Object getObject() throws Exception {
        return Worker.generateWorker();
    }

    @Override
    public Class<?> getObjectType() {
        return Worker.class;
    }
}
```

方式1 ： 通过 xml 方式实例化

```xml
<bean id="worker-create-by-xml-factory-bean" class="com.jackhance.spring.ioc.bean.model.WorkerFactoryBean" />
```


方式2 ： 通过注解方式实例化

```java
@Bean(name = "worker-create-by-annotated-factory-bean")
public WorkerFactoryBean workerCreateByAnnotatedFactoryBean() {
    return new WorkerFactoryBean();
}
```

#### ServiceLoader 方式实例化 (特殊)

ServiceLoader 是一种特殊的服务实现方式。在实现服务的 java 平台中，可通过添加对应的服务供应实现类到类路径下，以此实现功能的扩展。

ServiceLoader 是指定配置资源目录为 `META-INF/services` ，目录下配置的文件为服务的类型（实现接口的类全名），文件内容包含服务实现类型的列表，允许空行，一个作为一行，`#` 为注释

e.g. :

创建文件 META-INF/services/com.jackhance.spring.ioc.bean.model.WorkerFactory ，文件内容：
```
com.jackhance.spring.ioc.bean.model.GenericWorkerFactory
```

```xml
<bean id="workerFactoryServiceLoader" class="org.springframework.beans.factory.serviceloader.ServiceLoaderFactoryBean">
    <property name="serviceType" value="com.jackhance.spring.ioc.bean.model.WorkerFactory" />
</bean>
```

```java
System.out.println("================================= ServiceLoaderFactoryBean ==============================");

ServiceLoader serviceLoader = applicationContext.getBean("workerFactoryServiceLoader", ServiceLoader.class);

for (Iterator<WorkerFactory> it = serviceLoader.iterator(); it.hasNext(); ) {
    WorkerFactory workerFactory = it.next();
    System.out.println("ServiceLoader : " + workerFactory.createWorker());
}

System.out.println("====================== ServiceLoader # load =========================================");

serviceLoader = ServiceLoader.load(WorkerFactory.class);

for (Iterator<WorkerFactory> it = serviceLoader.iterator(); it.hasNext(); ) {
    WorkerFactory workerFactory = it.next();
    System.out.println("ServiceLoader : " + workerFactory.createWorker());
}
```

#### AutowireCapableBeanFactory # createBean 方式实例化 (特殊)

```java
/**
 * 通用 {@link com.jackhance.spring.ioc.container.model.Worker} 工厂
 *
 * @author jackhance
 * @mail jackhance0825@163.com
 * @date 2021/9/28 23:47
 */
public class GenericWorkerFactory implements WorkerFactory, InitializingBean, DisposableBean, BeanNameAware {

    private String name;

    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct : " + this.name + " 初始化中...");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean#afterPropertiesSet() : " + this.name + " 初始化中...");
    }

    public void doInit() {
        System.out.println("自定义初始化方法 doInit() : " + this.name + " 初始化中...");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("@PreDestroy : " + this.name + " 销毁中...");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean#destroy() : " + this.name + " 销毁中...");
    }

    public void doDestroy() {
        System.out.println("自定义销毁方法 doDestroy() : " + this.name + " 销毁中...");
    }

    @Override
    public void finalize() throws Throwable {
        System.out.println("当前 GenericWorkerFactory 对象 name = " + this.name + " 正在被垃圾回收...");
    }

    @Override
    public void setBeanName(String name) {
        this.name = name;
    }
}
```

```java
AutowireCapableBeanFactory beanFactory = applicationContext.getAutowireCapableBeanFactory();

WorkerFactory workerFactory = (WorkerFactory) beanFactory.createBean(GenericWorkerFactory.class, AUTOWIRE_BY_TYPE, true);

applicationContext.close();
```

执行输出： 

```cmd
@PostConstruct : com.jackhance.spring.ioc.bean.model.GenericWorkerFactory 初始化中...
InitializingBean#afterPropertiesSet() : com.jackhance.spring.ioc.bean.model.GenericWorkerFactory 初始化中...
```

在此方式下，可通过给定的类创建一个新的 bean ，行使所有初始化行为，包含适用的 BeanPostProcessor 。

在bean 的生命周期内，会触发初始化方法 （e.g. `@PostConstruct`、`InitializingBean # afterPropertiesSet` 等等），但不会触发销毁方法（e.g. `@PreDestroy`、`DisposableBean # destroy` 等等）

<br>
<hr>


### Spring Bean 的初始化


#### `@PostConstruct` 方法

Bean 可在方法标注注解 `@PostConstruct` ，可在 Bean 初始化时触发。

```java
public class GenericWorkerFactory implements WorkerFactory, InitializingBean, DisposableBean, BeanNameAware {

    // ...

    @PostConstruct
    public void init() {
        System.out.println("@PostConstruct : " + this.name + " 初始化中...");
    }

    // ...

}
```

#### `InitializingBean # afterPropertiesSet`

Bean 实现接口 `InitializingBean # afterPropertiesSet` ，可在 Bean 初始化时触发。

```java
public class GenericWorkerFactory implements WorkerFactory, InitializingBean, DisposableBean, BeanNameAware {

    // ...

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean#afterPropertiesSet() : " + this.name + " 初始化中...");
    }

    // ...

}
```


#### 自定义初始化方法

首先定义 Bean 的初始化方法：

```java
public class GenericWorkerFactory implements WorkerFactory, InitializingBean, DisposableBean, BeanNameAware {

    // ...

    public void doInit() {
        System.out.println("自定义初始化方法 doInit() : " + this.name + " 初始化中...");
    }

    // ...

}
```

注册 Bean 的初始化方法，有以下方式：
- XML 配置：`<bean init-method="init" ... />`
- Java 注解：`@Bean(initMethod="init")`
- Java API：`AbstractBeanDefinition # setInitMethodName(String)`


#### 初始化方法的执行顺序

1. `@PostConstruct`
2. `InitializingBean # afterPropertiesSet`
3. 自定义初始化方法


<br>
<hr>


### Spring Bean 的延迟初始化

在 Spring 容器启动时，标注了延迟初始化的 Bean 并不会初始化，初始化会发生在获取时。

#### xml 配置

```xml
<bean id="worker-factory-by-xml-lazy" class="com.jackhance.spring.ioc.bean.model.GenericWorkerFactory"
        init-method="doInit"
        destroy-method="doDestroy"
        lazy-init="true" />
```

#### 注解 `@Lazy`

```java
@Bean(name = "worker-factory-create-by-annotated-lazy", initMethod = "doInit", destroyMethod = "doDestroy")
@Lazy
public WorkerFactory annotatedLazyWorkerFactory() {
    return new GenericWorkerFactory();
}
```

<br>
<hr>


### Spring Bean 的销毁

#### `@PreDestroy`

```java
public class GenericWorkerFactory implements WorkerFactory, InitializingBean, DisposableBean, BeanNameAware {

    // ...

    @PreDestroy
    public void preDestroy() {
        System.out.println("@PreDestroy : " + this.name + " 销毁中...");
    }

    // ...

}
```


#### `DisposableBean # destroy`

```java
public class GenericWorkerFactory implements WorkerFactory, InitializingBean, DisposableBean, BeanNameAware {

    // ...

    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean#destroy() : " + this.name + " 销毁中...");
    }

    // ...

}
```


#### 自定义销毁方法

首先定义 Bean 的销毁方法：

```java
public class GenericWorkerFactory implements WorkerFactory, InitializingBean, DisposableBean, BeanNameAware {

    // ...

    public void doDestroy() {
        System.out.println("自定义销毁方法 doDestroy() : " + this.name + " 销毁中...");
    }

    // ...
}
```

然后注册 Bean 的销毁方法方式：
- XML 配置：`<bean destroy="destroy" ... />`
- Java 注解：`@Bean(destroy="destroy")`
- Java API：`AbstractBeanDefinition # setDestroyMethodName(String)`

#### 销毁方法的执行顺序

1. `@PreDestroy`
2. `DisposableBean # destroy`
3. 自定义销毁方法

<br>
<hr>


### 小记

<small id="xsd-id-ref"><sup>[[1]](#xsd-id)</sup> <a href="https://github.com/spring-projects/spring-framework/blob/3.0.x/org.springframework.beans/src/main/resources/org/springframework/beans/factory/xml/spring-beans-3.0.xsd" target="_blank" > xsd-ID 声明 </a></small> 

<small id="xsd-string-ref"><sup>[[2]](#xsd-string)</sup> <a href="https://github.com/spring-projects/spring-framework/blob/3.1.x/org.springframework.beans/src/main/resources/org/springframework/beans/factory/xml/spring-beans-3.1.xsd" target="_blank" > xsd-string 声明 </a></small> 

### 代码

<a href="https://github.com/jackhance0825/thinking-in-spring-5.2.16/tree/main/bean" target="_blank" > 本篇章代码 </a>
