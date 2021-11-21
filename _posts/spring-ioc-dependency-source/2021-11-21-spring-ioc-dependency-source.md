---
title: Spring 卷 - IOC 篇 - Spring Ioc 依赖来源
date: 2021-11-21 14:53:00 +07:00
modified: 2021-11-21 14:53:00 +07:00
tags: [java, spring, ioc, dependency, source]
description: Spring Ioc 依赖来源详解
---


### 依赖来源的分类

| 依赖来源类型                  | 是否为 Spring Bean                  |   生命周期是否由 Spring 管理    |  是否存在配置元信息   | 使用场景              |
| :--------------------------- | :--------------------------------- | :----------------------------- | :------------------ |:-------------------: |
|  BeanDefinition              |    是                              |    是                           |    是               | 依赖查找、依赖注入    |
| 单体对象（Singleton）         |    是                              |    否                           |    否               | 依赖查找、依赖注入    |
| Resolvable Dependency        |    否                              |    否                           |    否               | 依赖注入             |
| 外部化配置（e.g. properties） |    否                              |    否                           |    否               | e.g. `@Value` `@PropertySource` `@PropertySources`|


<hr/>
<br/>


### BeanDefinition 依赖来源

#### BeanDefinition 依赖来源要数

注册方法： `BeanDefinitionRegistry#registerBeanDefinition(String beanName, BeanDefinition beanDefinition)`

元数据： BeanDefinition

延迟：支持


#### BeanDefinition 依赖来源配置元信息

方式1：

```xml
<bean id="worker1" class="com.jackhance.spring.ioc.dependency.source.model.Worker">
    <constructor-arg name="id"  value="1" />
    <constructor-arg name="name" value="jackhance"/>
</bean>
```


方式2：

```java

@Bean
private Worker worker2() {
    return new Worker(2, "jackhance");
}

```


方式3：
```java
private static void registerBeanDefinition(BeanDefinitionRegistry registry) {
    BeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Worker.class)
            .addPropertyValue("id", 3)
            .addPropertyValue("name", "jackhance")
            .getBeanDefinition();

    registry.registerBeanDefinition("worker3", beanDefinition);
}
```

#### Spring 内建 BeanDefinition 依赖来源

| Bean 名称                  | Bean 实例对象                  |   使用场景              |
| :--------------------------- | :-------------------------- |:-------------------: |
| AnnotationConfigUtils.CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME  | ConfigurationClassPostProcessor | 处理配置类  @Configuration  |
| AnnotationConfigUtils.AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME      | AutowiredAnnotationBeanPostProcessor | 处理 @Autowire @Value @Inject |
| AnnotationConfigUtils.COMMON_ANNOTATION_PROCESSOR_BEAN_NAME         | CommonAnnotationBeanPostProcessor    | 处理 @Resource @WebServiceRef @EJB @PostConstruct @PreDestroy |
| AnnotationConfigUtils.EVENT_LISTENER_PROCESSOR_BEAN_NAME            | EventListenerMethodProcessor         | 处理 EventListener |
| AnnotationConfigUtils.EVENT_LISTENER_FACTORY_BEAN_NAME              | DefaultEventListenerFactory          | 支持 @EventListener |


<hr/>
<br/>

### 单体对象（Singleton） 依赖来源

#### 单体对象（Singleton）要素

注册方式： `SingletonBeanRegistry#registerSingleton(String beanName, Object singletonObject)`

无 Spring 生命周期管理

无法支持延迟初始化

#### Spring 内建单例对象

| Bean 名称                  | Bean 实例对象                  |   使用场景              |
| :--------------------------- | :-------------------------- |:-------------------: |
| ConfigurableApplicationContext.ENVIRONMENT_BEAN_NAME          |   ConfigurableEnvironment         |  e.g. 外部化配置  profiles  |
| ConfigurableApplicationContext.SYSTEM_PROPERTIES_BEAN_NAME    |   java.util.Map<String, Object>   |  系统属性  |
| ConfigurableApplicationContext.SYSTEM_ENVIRONMENT_BEAN_NAME   |    java.util.Map<String, Object>  |  os 环境变量 |
| AbstractApplicationContext.MESSAGE_SOURCE_BEAN_NAME           |   MessageSource                   |    i18n 国际化    |
| AbstractApplicationContext.APPLICATION_EVENT_MULTICASTER_BEAN_NAME | ApplicationEventMulticaste    |  Spring 事件广播器 |
| AbstractApplicationContext.LIFECYCLE_PROCESSOR_BEAN_NAME      |   LifecycleProcessor               | Lifecycle beans 处理 |

<hr/>
<br/>

### Resolvable Dependency 依赖来源


#### Resolvable Dependency 要素

注册方式： `ConfigurableListableBeanFactory#registerResolvableDependency(Class<?> dependencyType, @Nullable Object autowiredValue)`

无 Spring 生命周期管理

无法支持延迟初始化

无法依赖查找

#### Spring 内建 Resolvable Dependency 

| Bean 实例对象                  |   使用场景              |
| :--------------------------- |:-------------------: |
| BeanFactory                  |  IoC 容器            |
| ResourceLoader               |  资源加载处理器       |
| ApplicationEventPublisher    |  Spring 事件发布器    |
| ApplicationContext           |  应用上下文           |


<hr/>
<br/>

### 外部化配置依赖来源

#### 外部化配置要素

非常规 Spring 依赖来源

无 Spring 生命周期管理

无法支持延迟初始化

无法依赖查找

#### 外部化配置示例

```java
/**
 * 外部化配置依赖来源示例
 *
 * @author jackhance
 * @mail jackhance0825@163.com
 */
@Configuration
@PropertySource(value = "META-INF/spring-ioc-source-external.properties", encoding = "UTF-8")
public class ExternalSourceDemo {

    @Value("${usr.name}")
    String name;

    @Value("${usr.age:1}")
    private int age;

    @Value("${usr.resource}")
    private Resource resource;

    /**
     * 没有配置，默认:广州
     */
    @Value("${usr.location:广州}")
    private String location;

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();

        // 注册配置类
        applicationContext.register(ExternalSourceDemo.class);

        // 启动应用上下文
        applicationContext.refresh();

        ExternalSourceDemo demo = applicationContext.getBean(ExternalSourceDemo.class);

        System.out.println("name : " + demo.name);
        System.out.println("age : " + demo.age);
        System.out.println("resource : " + demo.resource);
        System.out.println("location : " + demo.location);

        // 关闭应用上下文
        applicationContext.close();
    }
}
```

<hr/>
<br/>

### 代码

<a href="https://github.com/jackhance0825/thinking-in-spring-5.2.16/tree/main/ioc-dependency-source" target="_blank" > 本篇章代码 </a>