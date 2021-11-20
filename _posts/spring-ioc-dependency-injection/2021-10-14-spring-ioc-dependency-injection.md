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

Setter 方法依赖注入，实现方式分为**手动模式**和**自动模式**，通过 Bean 的 setter 方法实现依赖注入。

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

通过 xml 配置，将 id 为 simpleWorker 的 Bean，通过 WorkerHolder#setWorker 方法，依赖注入到 id 为workerHolder 的 Bean。

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

通过注解 `@Bean` 依赖注入类型为 Worker 的Bean（存在多个时，选择 primary），方法体内通过 setter 方法构建 annotatedWorkerHolder ，并注册 id 为 annotatedWorkerHolder 的 Bean 到 BeanFactory。

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

按属性名称自动装配。 Spring 查找与需要自动装配的属性同名的 bean。例如，如果一个 bean 定义被设置为按名称自动装配并且它包含一个主属性（即它有一个 setWorker(..) 方法），Spring 会查找一个名为 worker 的 bean 定义并使用它来设置属性。

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

如果容器中只存在一个属性类型的 bean，则让属性自动装配。如果存在多个（不存在 primary ），则会引发致命异常，这表明您不能为该 bean 使用 byType 自动装配。如果没有匹配的 bean，则不会发生任何事情（未设置属性）。

<br>
<hr>

### 构造器依赖注入

构造器依赖注入实现方式分为**手动模式**和**自动模式**，通过 Bean 的构造方法实现依赖注入。

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

通过 xml 配置，将 id 为 simpleWorker 的 Bean，通过 WorkerHolder 的构造方法，依赖注入到构造参数 worker。

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

类似于 byType 但适用于构造函数参数。如果容器中没有一个构造函数参数类型的 bean，则会引发致命错误。

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

`@Autowired` 的依赖注入，是通过 `AutowiredAnnotationBeanPostProcessor` 实现依赖注入。

#### `@Resource`

```java
    /**
     * {@link Resource} 方式依赖注入
     */
    @Resource
    private WorkerHolder workerHolder2;

```

`@Resource` 的依赖注入，是通过 `CommonAnnotationBeanPostProcessor` 实现依赖注入。


#### `@Inject`

```java
    /**
     * {@link Inject} 方式依赖注入
     */
    @Inject
    private WorkerHolder workerHolder3;
```

JSR 330 `@Inject` 的依赖注入，是通过 `AutowiredAnnotationBeanPostProcessor` 实现依赖注入。

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

#### `@Inject`

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

<br>

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

        // 省略业务逻辑...

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

#### 构造器注入使用场景

Spring 团队通常**提倡**构造函数注入，因为它可以让您将应用程序组件实现为不可变对象，并确保所需的依赖项不是null。此外，构造函数注入的组件总是以完全初始化的状态返回给客户端（调用）代码。

但是，大量的构造参数会降低代码的可读性，同时，也代表此类的职责不够单一，应该考虑重构。

因此，`在强依赖、低依赖场景，选构造器注入。`

#### Setter 方法注入使用场景

对于强制依赖项的构造器注入，可选依赖项使用 Setter 方法注入是一种比较优雅的方式。可以在类中配置合理的默认值的可选依赖项，通过 Setter 方法注入有选择性注入依赖。

但是，依赖注入的顺序，完全依赖于用户的操作顺序，如果依赖项存在前后依赖关系，使用此方式，会存在隐患。

因此，`在多依赖场景，选 Setter 方法注入。`

#### 字段注入使用场景

字段注入方式，通过 `@Autowired` 、`@Resource`、`@Inject` 或者自定义注解，标注到字段上，就可以实现依赖注入，这种方式对于我们写程序是很便利的。

但是，这种方式无论在 Spring 或者 Spring Boot 官网上，作者偏好于构造器注入，字段注入是处于准备淘汰的状态。

因此，`字段注入方式，简单便利。`

#### 方法注入使用场景

使用方法注入时，更建议使用 `@Bean` 注解注入参数，Spring 会依赖注入参数到方法，在 Bean 构造的情况下，这其实是一种组合方式，先通过注解依赖注入，在通过手动 API 注入参数，构造 Bean。

因此，`声明类，选方法注入。`


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
<br>

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

    public Worker buildWorker(String id, String name, int age) {
        Worker worker = new Worker();
        worker.setId(id);
        worker.setName(name);
        worker.setAge(age);
        return worker;
    }

    public static void main(String[] args) {
        // 创建应用上下文
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();

        // 注册注册类
        applicationContext.register(QualifierInjectDemo.class);

        // 启动应用上下文
        applicationContext.refresh();

        QualifierInjectDemo demo = applicationContext.getBean(QualifierInjectDemo.class);

        // 期待输出 worker2 Bean
        System.out.println("demo.worker = " + demo.worker);
        // 期待输出 worker4 Bean
        System.out.println("demo.qualifierWorker = " + demo.qualifierWorker);
        // 期待输出 worker1-6
        System.out.println("demo.workers = " + demo.workers);
        // 期待输出 worker3 worker4 worker5 worker6
        System.out.println("demo.qualifiedUsers = " + demo.qualifierWorkers);
        // 期待输出 worker5 worker6
        System.out.println("demo.groupWorkers = " + demo.groupWorkers);

        // 关闭应用上下文
        applicationContext.close();
    }
```

<br>

output:
```text
demo.worker = Worker{id='2', name='jackhance', age=30, hash=945591847}
demo.qualifierWorker = Worker{id='4', name='jackhance', age=30, hash=328827614}
demo.workers = [Worker{id='1', name='jackhance', age=30, hash=109228794}, Worker{id='2', name='jackhance', age=30, hash=945591847}, Worker{id='3', name='jackhance', age=30, hash=561959774}, Worker{id='4', name='jackhance', age=30, hash=328827614}, Worker{id='5', name='jackhance', age=30, hash=2110756088}, Worker{id='6', name='jackhance', age=30, hash=580871917}]
demo.qualifiedUsers = [Worker{id='3', name='jackhance', age=30, hash=561959774}, Worker{id='4', name='jackhance', age=30, hash=328827614}, Worker{id='5', name='jackhance', age=30, hash=2110756088}, Worker{id='6', name='jackhance', age=30, hash=580871917}]
demo.groupWorkers = [Worker{id='5', name='jackhance', age=30, hash=2110756088}, Worker{id='6', name='jackhance', age=30, hash=580871917}]
```

<br>

通过控制台打印输出，我们可以看出：
- `qualifierWorker` 注入指定名字为 worker4 的 Bean 实例；
- `workers` 注入了所有类型的 `Worker` Bean 实例；
- `qualifiedUsers` 注入了标注了 `@Qualifier` 和 `@Group` 所有类型的 `Worker` Bean 实例；
- `groupWorkers` 注入了标注了 `@Group` 所有类型的 `Worker` Bean 实例；


#### `@Qualifier` 的作用

1. 指定注入特定 id 的 Bean（ e.g. `@Qualifier("worker4")` ）;
2. `@Qualifier` 及其"派生注解"（ e.g. `@Group` ） 可以起到分组作用；
3. `@Qualifier`以及"派生注解"标注的分组（ e.g. Spring Cloud 的 `@LoadBalanced` ），父元标注包括子元标注；


#### `@Qualifier` 是如何指定 Bean 注入

`@Qualifier` 的如何匹配 Bean 实现精确依赖注入的？

详见 `QualifierAnnotationAutowireCandidateResolver # checkQualifier` 源码 ：
```java
    /**
	 * Match the given qualifier annotations against the candidate bean definition.
     *
     * @param bdHolder 进行匹配的 {@link BeanDefinitionHolder}
     * @param annotationsToSearch 被依赖注入的 Bean 的注解
     * @return 是否匹配
	 */
	protected boolean checkQualifiers(BeanDefinitionHolder bdHolder, Annotation[] annotationsToSearch) {
		if (ObjectUtils.isEmpty(annotationsToSearch)) {
			return true;
		}
		SimpleTypeConverter typeConverter = new SimpleTypeConverter();
		for (Annotation annotation : annotationsToSearch) {// 被依赖注入的 Bean 的注解
			Class<? extends Annotation> type = annotation.annotationType();
			boolean checkMeta = true;
			boolean fallbackToMeta = false;
			if (isQualifier(type)) {// 被依赖注入的 Bean 的注解是 @Qualifier 或者 元标注@Qualifier的注解 或者 自定义Qualifier注解
				if (!checkQualifier(bdHolder, annotation, typeConverter)) {// 是否匹配
					fallbackToMeta = true;// 不满足匹配，检查注解的元标注
				}
				else {
					checkMeta = false;// 已经匹配，不再需要检查注解的元标注
				}
			}
            // 检查注解的元标注是否满足匹配
			if (checkMeta) {
				boolean foundMeta = false;
				for (Annotation metaAnn : type.getAnnotations()) {
					Class<? extends Annotation> metaType = metaAnn.annotationType();
					if (isQualifier(metaType)) {// 注解的元标注是 @Qualifier 或者 元标注@Qualifier的注解 或者 自定义Qualifier注解
						foundMeta = true;
						if ((fallbackToMeta && StringUtils.isEmpty(AnnotationUtils.getValue(metaAnn))) ||// 检查注解的元标注，拥有 value 值的 @Qualifier 的匹配
								!checkQualifier(bdHolder, metaAnn, typeConverter)) {// 是否匹配 value 值
							return false;
						}
					}
				}
				if (fallbackToMeta && !foundMeta) {// 注解的元标注找不到 Qualifier，不满足匹配
					return false;
				}
			}
		}
		return true;// 匹配
	}

    /**
	 * Match the given qualifier annotation against the candidate bean definition.
	 */
	protected boolean checkQualifier(
			BeanDefinitionHolder bdHolder, Annotation annotation, TypeConverter typeConverter) {

		Class<? extends Annotation> type = annotation.annotationType();
		RootBeanDefinition bd = (RootBeanDefinition) bdHolder.getBeanDefinition();

        // BeanDefinition 是否有指定的依赖注入候选Qualifier，可通过 {@link AbstractBeanDefinition#addQualifier} 注册
		AutowireCandidateQualifier qualifier = bd.getQualifier(type.getName());
		if (qualifier == null) {
			qualifier = bd.getQualifier(ClassUtils.getShortName(type));
		}
		if (qualifier == null) {
			// 首先，获取 BeanDefinition 此类型的注解
			Annotation targetAnnotation = getQualifiedElementAnnotation(bd, type);
			// 找不到，然后找工厂方法此类型的注解
			if (targetAnnotation == null) {
				targetAnnotation = getFactoryMethodAnnotation(bd, type);
			}
            // 找不到，然后找 RootBeanDefinition 此类型的注解
			if (targetAnnotation == null) {
				RootBeanDefinition dbd = getResolvedDecoratedDefinition(bd);
				if (dbd != null) {
					targetAnnotation = getFactoryMethodAnnotation(dbd, type);
				}
			}
            // 找不到，找目标类此类型的注解
			if (targetAnnotation == null) {
				// Look for matching annotation on the target class
				if (getBeanFactory() != null) {
					try {
						Class<?> beanType = getBeanFactory().getType(bdHolder.getBeanName());
						if (beanType != null) {
							targetAnnotation = AnnotationUtils.getAnnotation(ClassUtils.getUserClass(beanType), type);
						}
					}
					catch (NoSuchBeanDefinitionException ex) {
						// Not the usual case - simply forget about the type check...
					}
				}
				if (targetAnnotation == null && bd.hasBeanClass()) {
					targetAnnotation = AnnotationUtils.getAnnotation(ClassUtils.getUserClass(bd.getBeanClass()), type);
				}
			}
            // 若 bdHolder 持有的 BeanDefinition 的注解一样，返回匹配
			if (targetAnnotation != null && targetAnnotation.equals(annotation)) {
				return true;
			}
		}

        // 注解不一致，则获取注解的属性，属性是否匹配
		Map<String, Object> attributes = AnnotationUtils.getAnnotationAttributes(annotation);
		if (attributes.isEmpty() && qualifier == null) {
			// If no attributes, the qualifier must be present
			return false;
		}
		for (Map.Entry<String, Object> entry : attributes.entrySet()) {
			String attributeName = entry.getKey();// 被依赖注入的 Bean 的属性键值
			Object expectedValue = entry.getValue();// 被依赖注入的 Bean 的期待值
			Object actualValue = null;// 进行匹配 Bean 对应属性键的值
			// 检查依赖注入候选Qualifier的期待值
			if (qualifier != null) {
				actualValue = qualifier.getAttribute(attributeName);
			}
			if (actualValue == null) {
				// Fall back on bean definition attribute
				actualValue = bd.getAttribute(attributeName);
			}
            // 特殊处理值为 #{value} 的 attributeName ，Bean 的名称是否匹配 @Qualifier("beanName") 里面的 #{beanName}
			if (actualValue == null && attributeName.equals(AutowireCandidateQualifier.VALUE_KEY) &&
					expectedValue instanceof String && bdHolder.matchesName((String) expectedValue)) {
				// Fall back on bean name (or alias) match
				continue;
			}
			if (actualValue == null && qualifier != null) {
				// Fall back on default, but only if the qualifier is present
				actualValue = AnnotationUtils.getDefaultValue(annotation, attributeName);
			}
            // actualValue 类型转换
			if (actualValue != null) {
				actualValue = typeConverter.convertIfNecessary(actualValue, expectedValue.getClass());
			}
            // actualValue 是否匹配 被依赖注入的 Bean 的期待值
			if (!expectedValue.equals(actualValue)) {
				return false;
			}
		}
		return true;
	}

/*
* <pre>
*    逻辑分析：
*
*    约定：
*        Q：@Qualifier 或者 元标注@Qualifier的注解 或者 自定义Qualifier注解
*        match：注解equals 或者 注解的全部 attribute 值都匹配
*    
*    若被依赖的 Bean 不存在注解 Q ？ 若被依赖的 Bean 注解的元标注不存在注解 Q ？ BeanDefinition 匹配
*                                                                        ： 若被依赖的 Bean 注解的元标注 match ？ BeanDefinition 匹配 ： BeanDefinition 不匹配
*                                : 若被依赖的 Bean 注解 match ? BeanDefinition 匹配
*                                                        : 若被依赖的 Bean 注解的元标注不存在注解 Q ？ BeanDefinition 不匹配
*                                                                                                ： 若被依赖的 Bean 注解的元标注 match ？ BeanDefinition 匹配 ： BeanDefinition 不匹配
* </pre>
*/
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

### 依赖注入过程

处理入口：`DefaultListableBeanFactory#resolveDependency`


### `@Autowired` 注入原理

- 处理器：`AutowiredAnnotationBeanPostProcessor`
    - 通过 `AnnotationConfigUtils#registerAnnotationConfigProcessors` 注册
- 注入类型
    - 非静态字段
    - 非静态方法
    - 构造器

<br>
<hr>

### Java 通用注解注入原理（JSR250）

- 处理器：`CommonAnnotationBeanPostProcessor`
    - 通过 `AnnotationConfigUtils#registerAnnotationConfigProcessors` 注册
- 注入注解：
    - javax.xml.ws.WebServiceRef
    - javax.ejb.EJB
    - javax.annotation.Resource
- 生命周期注解：
    - javax.annotation.PostConstruct
    - javax.annotation.PreDestroy

<br>
<hr>

### `@Inject` 注入原理（JSR330）

- 处理器：`AutowiredAnnotationBeanPostProcessor`
    - 通过 `AnnotationConfigUtils#registerAnnotationConfigProcessors` 注册

<br>
<hr>

### 自定义注解驱动的依赖注入

可基于 `AutowiredAnnotationBeanPostProcessor` 实现

```java
/**
     * 自定义方式一：
     * 覆盖 {@link AutowiredAnnotationBeanPostProcessor} 注册
     *
     * @see AnnotationConfigUtils#registerAnnotationConfigProcessors
     */
    @Bean(name = AnnotationConfigUtils.AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)
    public static AutowiredAnnotationBeanPostProcessor beanPostProcessor() {
        AutowiredAnnotationBeanPostProcessor beanPostProcessor = new AutowiredAnnotationBeanPostProcessor();

        // @Autowired + @Inject +  新注解 @CustomInject
        Set<Class<? extends Annotation>> autowiredAnnotationTypes =
                new LinkedHashSet<>(Arrays.asList(Autowired.class, Inject.class, CustomInject.class));
        
        beanPostProcessor.setAutowiredAnnotationTypes(autowiredAnnotationTypes);

        return beanPostProcessor;
    }
```

<br>

```java
    /**
     * 自定义方式二：
     *
     * 自定义注解驱动
     */
    @Bean
    @Order(Ordered.LOWEST_PRECEDENCE - 3)
    @Scope
    public static AutowiredAnnotationBeanPostProcessor beanPostProcessor() {
        AutowiredAnnotationBeanPostProcessor beanPostProcessor = new AutowiredAnnotationBeanPostProcessor();
        beanPostProcessor.setAutowiredAnnotationType(CustomInject.class);
        return beanPostProcessor;
    }

```

<br>
<hr>

### 小记

<small id="autowiring-collaborators-ref"><sup>[[1]](#autowiring-collaborators)</sup> <a href="https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire" target="_blank" > Autowiring Collaborators </a></small> 

<small id="limitations-and-disadvantages-of-autowiring-ref"><sup>[[1]](#limitations-and-disadvantages-of-autowiring)</sup> <a href="https://docs.spring.io/spring-framework/docs/5.2.16.RELEASE/spring-framework-reference/core.html#beans-autowired-exceptions" target="_blank" > limitations and disadvantages of autowiring </a></small> 

<br>
<hr>

### 代码

<a href="https://github.com/jackhance0825/thinking-in-spring-5.2.16/tree/main/ioc-dependency-injection" target="_blank" > 本篇章代码 </a>
