---
title: Spring 卷 - IOC 篇 - Spring Bean 生命周期
date: 2021-11-27 20:57:00 +07:00
modified: 2021-11-27 20:57:00 +07:00
tags: [java, spring, ioc, bean, lifecycle]
description: Spring Bean 生命周期
---


### Spring Bean 元信息配置阶段

#### 基于资源配置

资源配置支持：xml、properties、groovy


#### 基于 java 注解配置

注解支持：e.g. `@Configuration 、@Bean、@Componen、@Controller、@Service...`


#### 基于 API 配置

1. 构建 BeanDefinition
    1. 基于 BeanDefinitionBuilder
    2. 基于 GenericBeanDefinition

<br>
<hr>


### Spring Bean 元信息解析阶段

#### 基于资源的元信息解析

基于资源的元信息解析，是借助 AbstractBeanDefinitionReader 解析。

基于 properties 资源的元信息解析示例：

```properties

myWorker.(class)=com.jackhance.spring.ioc.lifecycle.model.Worker
myWorker.id=825
myWorker.name=灵境
myWorker.location=FOSHAN
myWorker.locations=FOSHAN,GUANGZHOU,BEIJING

```

<br/>

```java
public class CommonBeanDefinitionParsingDemo {

    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        // 基于 properties 资源的 BeanDefinitionReader
        PropertiesBeanDefinitionReader beanDefinitionReader = new PropertiesBeanDefinitionReader(beanFactory);

        String resourceLocation = "META-INF/metadata.properties";

        Resource resource = new ClassPathResource(resourceLocation);
        // 资源指定编码 UTF-8
        EncodedResource encodedResource = new EncodedResource(resource, "UTF-8");

        // 从 properties 资源读取元信息
        int loadBeanDefinitionCount = beanDefinitionReader.loadBeanDefinitions(encodedResource);
        System.out.println("加载 BeanDefinition 数量：" + loadBeanDefinitionCount);

        Worker myWorker = beanFactory.getBean("myWorker", Worker.class);
        System.out.println("myWorker=" + myWorker);
    }
}
```

#### 基于 java 注解的元信息解析

基于 java 注解的元信息解析，是借助 AnnotatedBeanDefinitionReader 解析。

```java
/**
 * 基于 java 注解 {@link org.springframework.beans.factory.config.BeanDefinition} 解析示例
 *
 * @author jackhance
 * @mail jackhance0825@163.com
 * @see AnnotatedBeanDefinitionReader
 * @see ConfigurationClassPostProcessor
 */
public class AnnotatedBeanDefinitionParsingDemo {

    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        // 基于 java 注解的 BeanDefinitionReader
        AnnotatedBeanDefinitionReader beanDefinitionReader = new AnnotatedBeanDefinitionReader(beanFactory);

        int beanDefinitionCountBefore = beanFactory.getBeanDefinitionCount();

        beanDefinitionReader.register(AnnotatedBeanDefinitionParsingDemo.class);

        int beanDefinitionCountAfter = beanFactory.getBeanDefinitionCount();

        System.out.println("加载 BeanDefinition 数量：" + (beanDefinitionCountAfter - beanDefinitionCountBefore));

        AnnotatedBeanDefinitionParsingDemo demo = beanFactory.getBean(AnnotatedBeanDefinitionParsingDemo.class);
        System.out.println("demo=" + demo);
    }

}
```


<br>
<hr>


### Spring BeanDefinition 注册阶段

实现 Spring BeanDefinition 注册 API ：

DefaultListableBeanFactory#registerBeanDefinition(String beanName, BeanDefinition beanDefinition)

通过配置元信息，生成普通 BeanDefinition(GenericBeanDefinition) ，注册到 BeanFactory。

<br>
<hr>


### Spring BeanDefinition 合并阶段

实现 Spring BeanDefinition 合并 API ：

AbstractBeanFactory#getMergedBeanDefinition(String beanName)

```java
public BeanDefinition getMergedBeanDefinition(String name) throws BeansException {
    String beanName = transformedBeanName(name);
    // Efficiently check whether bean definition exists in this factory.
    if (!containsBeanDefinition(beanName) && getParentBeanFactory() instanceof ConfigurableBeanFactory) {
        return ((ConfigurableBeanFactory) getParentBeanFactory()).getMergedBeanDefinition(beanName);
    }
    // Resolve merged bean definition locally.
    return getMergedLocalBeanDefinition(beanName);
}
```

获取 RootBeanDefinition 的方式类似于类加载的双亲委派机制，Spring BeanFactory 是层次性，流程如下：
- 从父 BeanFactory 尝试获取 RootBeanDefinition，若能获取，则返回；
- 尝试从子 BeanFactory 尝试获取 RootBeanDefinition，若能获取，则返回；
- 以此类推

合并过程中，将 GenericBeanDefinition 合并成 RootBeanDefinition（已处理 parent ），子属性可以覆盖 parent 属性。

AbstractBeanFactory#getMergedBeanDefinition 代码片段：
```java
protected RootBeanDefinition getMergedBeanDefinition(
			String beanName, BeanDefinition bd, @Nullable BeanDefinition containingBd)
			throws BeanDefinitionStoreException {

		synchronized (this.mergedBeanDefinitions) {
			RootBeanDefinition mbd = null;
			RootBeanDefinition previous = null;

			// Check with full lock now in order to enforce the same merged instance.
			if (containingBd == null) {
				mbd = this.mergedBeanDefinitions.get(beanName);
			}

			if (mbd == null || mbd.stale) {
				previous = mbd;
				if (bd.getParentName() == null) {// 没有 parent ， 直接封装当前 BeanDefinition 为 RootBeanDefinition
					// Use copy of given root bean definition.
					if (bd instanceof RootBeanDefinition) {
						mbd = ((RootBeanDefinition) bd).cloneBeanDefinition();
					}
					else {
						mbd = new RootBeanDefinition(bd);
					}
				}
				else {// 存在 parent ，查找并合并  parent 的 RootBeanDefinition 生成当前 beanName 的 RootBeanDefinition
					// Child bean definition: needs to be merged with parent.
					BeanDefinition pbd;
					try {
						String parentBeanName = transformedBeanName(bd.getParentName());
						if (!beanName.equals(parentBeanName)) {
							pbd = getMergedBeanDefinition(parentBeanName);
						}
						else {
							BeanFactory parent = getParentBeanFactory();
							if (parent instanceof ConfigurableBeanFactory) {
								pbd = ((ConfigurableBeanFactory) parent).getMergedBeanDefinition(parentBeanName);
							}
							else {
								throw new NoSuchBeanDefinitionException(parentBeanName,
										"Parent name '" + parentBeanName + "' is equal to bean name '" + beanName +
										"': cannot be resolved without a ConfigurableBeanFactory parent");
							}
						}
					}
					catch (NoSuchBeanDefinitionException ex) {
						throw new BeanDefinitionStoreException(bd.getResourceDescription(), beanName,
								"Could not resolve parent bean definition '" + bd.getParentName() + "'", ex);
					}
					// 深拷贝，子属性覆盖父属性
					mbd = new RootBeanDefinition(pbd);
					mbd.overrideFrom(bd);
				}

				// 省略部分代码...
			}
			if (previous != null) {
				copyRelevantMergedBeanDefinitionCaches(previous, mbd);
			}
			return mbd;
		}
	}

```


<br>
<hr>

### Spring Bean 类加载阶段

BeanDefinition 提供 bean 类名称、bean 类处理加载等相关接口。当 Spring 从元信息中读取 BeanDefinition 信息后，将元信息构建成 BeanDefinition ，BeanDefinition#beanClass 保存了类全名。当 BeanDefinition 要实例化时，触发类加载器加载，并将 beanClass 设置成 Class 对象（beanClass 为 Object 类型，构建 BeanDefinition 时，保存的是 String 类型）。

摘录 AbstractBeanDefinition 部分代码片段：
```java
public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor
    implements BeanDefinition, Cloneable {
	/**
	 * Specify the bean class name of this bean definition.
	 */
	@Override
	public void setBeanClassName(@Nullable String beanClassName) {
		this.beanClass = beanClassName;
	}

	/**
	 * Return the current bean class name of this bean definition.
	 */
	@Override
	@Nullable
	public String getBeanClassName() {
		Object beanClassObject = this.beanClass;
		if (beanClassObject instanceof Class) {
			return ((Class<?>) beanClassObject).getName();
		}
		else {
			return (String) beanClassObject;
		}
	}

	/**
	 * Specify the class for this bean.
	 * @see #setBeanClassName(String)
	 */
	public void setBeanClass(@Nullable Class<?> beanClass) {
		this.beanClass = beanClass;
	}

	/**
	 * the resolved bean class (never {@code null})
	 */
	public Class<?> getBeanClass() throws IllegalStateException {
		Object beanClassObject = this.beanClass;
		if (beanClassObject == null) {
			throw new IllegalStateException("No bean class specified on bean definition");
		}
		if (!(beanClassObject instanceof Class)) {
			throw new IllegalStateException(
					"Bean class name [" + beanClassObject + "] has not been resolved into an actual Class");
		}
		return (Class<?>) beanClassObject;
	}

	/**
	 * Return whether this definition specifies a bean class.
	 */
	public boolean hasBeanClass() {
		return (this.beanClass instanceof Class);
	}

	/**
	 * Determine the class of the wrapped bean, resolving it from a
	 * specified class name if necessary. Will also reload a specified
	 * Class from its name when called with the bean class already resolved.
	 * @param classLoader the ClassLoader to use for resolving a (potential) class name
	 * @return the resolved bean class
	 * @throws ClassNotFoundException if the class name could be resolved
	 */
	@Nullable
	public Class<?> resolveBeanClass(@Nullable ClassLoader classLoader) throws ClassNotFoundException {
		String className = getBeanClassName();
		if (className == null) {
			return null;
		}
		Class<?> resolvedClass = ClassUtils.forName(className, classLoader);
		this.beanClass = resolvedClass;
		return resolvedClass;
	}

// 省略部分代码
}


```

<br/>


BeanDefinition 类加载过程，AbstractBeanFactory#doResolveBeanClass 代码：

```java
private Class<?> doResolveBeanClass(RootBeanDefinition mbd, Class<?>... typesToMatch)
			throws ClassNotFoundException {

		ClassLoader beanClassLoader = getBeanClassLoader();
		ClassLoader dynamicLoader = beanClassLoader;
		boolean freshResolve = false;

		if (!ObjectUtils.isEmpty(typesToMatch)) {
			// When just doing type checks (i.e. not creating an actual instance yet),
			// use the specified temporary class loader (e.g. in a weaving scenario).
			ClassLoader tempClassLoader = getTempClassLoader();
			if (tempClassLoader != null) {
				dynamicLoader = tempClassLoader;
				freshResolve = true;
				if (tempClassLoader instanceof DecoratingClassLoader) {
					DecoratingClassLoader dcl = (DecoratingClassLoader) tempClassLoader;
					for (Class<?> typeToMatch : typesToMatch) {
						dcl.excludeClass(typeToMatch.getName());
					}
				}
			}
		}

		String className = mbd.getBeanClassName();
		if (className != null) {
			Object evaluated = evaluateBeanDefinitionString(className, mbd);
			if (!className.equals(evaluated)) {
				// A dynamically resolved expression, supported as of 4.2...
				if (evaluated instanceof Class) {
					return (Class<?>) evaluated;
				}
				else if (evaluated instanceof String) {
					className = (String) evaluated;
					freshResolve = true;
				}
				else {
					throw new IllegalStateException("Invalid class name expression result: " + evaluated);
				}
			}
			if (freshResolve) {
				// When resolving against a temporary class loader, exit early in order
				// to avoid storing the resolved Class in the bean definition.
				if (dynamicLoader != null) {
					try {
						return dynamicLoader.loadClass(className);
					}
					catch (ClassNotFoundException ex) {
						if (logger.isTraceEnabled()) {
							logger.trace("Could not load class [" + className + "] from " + dynamicLoader + ": " + ex);
						}
					}
				}
				return ClassUtils.forName(className, dynamicLoader);
			}
		}

		// Resolve regularly, caching the result in the BeanDefinition...
		return mbd.resolveBeanClass(beanClassLoader);
	}
```


<br>
<hr>

### Spring Bean 实例化阶段


#### Spring Bean 实例化前阶段

Spring Bean 实例化前阶段，可以通过注册实现接口 InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation 定制化，返回非 null 值，可跳过 Bean 实例化中阶段。

InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation 代码片段：
```java
	/**
	 * Apply this BeanPostProcessor <i>before the target bean gets instantiated</i>.
	 * The returned bean object may be a proxy to use instead of the target bean,
	 * effectively suppressing default instantiation of the target bean.
	 * <p>If a non-null object is returned by this method, the bean creation process
	 * will be short-circuited. The only further processing applied is the
	 * {@link #postProcessAfterInitialization} callback from the configured
	 * {@link BeanPostProcessor BeanPostProcessors}.
	 * <p>This callback will be applied to bean definitions with their bean class,
	 * as well as to factory-method definitions in which case the returned bean type
	 * will be passed in here.
	 * <p>Post-processors may implement the extended
	 * {@link SmartInstantiationAwareBeanPostProcessor} interface in order
	 * to predict the type of the bean object that they are going to return here.
	 * <p>The default implementation returns {@code null}.
	 * @param beanClass the class of the bean to be instantiated
	 * @param beanName the name of the bean
	 * @return the bean object to expose instead of a default instance of the target bean,
	 * or {@code null} to proceed with default instantiation
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see #postProcessAfterInstantiation
	 * @see org.springframework.beans.factory.support.AbstractBeanDefinition#getBeanClass()
	 * @see org.springframework.beans.factory.support.AbstractBeanDefinition#getFactoryMethodName()
	 */
	@Nullable
	default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}
```

<br/>

虽然前阶段处理器返回非 null 值，可跳过 Bean 实例化中阶段，但 Bean 实例化后阶段依然会执行。
AbstractAutowireCapableBeanFactory#resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) 代码片段：
```java
	protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
		Object bean = null;
		if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
			// Make sure bean class is actually resolved at this point.
			if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
				Class<?> targetType = determineTargetType(beanName, mbd);
				if (targetType != null) {
					bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
					if (bean != null) {
						bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
					}
				}
			}
			mbd.beforeInstantiationResolved = (bean != null);
		}
		return bean;
	}
```


#### Spring Bean 实例化中阶段

Spring Bean 实例化，存在俩种策略，常规策略 SimpleInstantiationStrategy， 基于 CGlib 的策略 CglibSubclassingInstantiationStrategy。

InstantiationStrategy#instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) 代码片段：
```java
    @Override
	public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
		// Don't override the class with CGLIB if no overrides.
		if (!bd.hasMethodOverrides()) {
			Constructor<?> constructorToUse;
			synchronized (bd.constructorArgumentLock) {
				constructorToUse = (Constructor<?>) bd.resolvedConstructorOrFactoryMethod;
				if (constructorToUse == null) {
					final Class<?> clazz = bd.getBeanClass();
					if (clazz.isInterface()) {
						throw new BeanInstantiationException(clazz, "Specified class is an interface");
					}
					try {
						if (System.getSecurityManager() != null) {
							constructorToUse = AccessController.doPrivileged(
									(PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);
						}
						else {
							constructorToUse = clazz.getDeclaredConstructor();
						}
						bd.resolvedConstructorOrFactoryMethod = constructorToUse;
					}
					catch (Throwable ex) {
						throw new BeanInstantiationException(clazz, "No default constructor found", ex);
					}
				}
			}
			return BeanUtils.instantiateClass(constructorToUse);
		}
		else {
			// Must generate CGLIB subclass.
			return instantiateWithMethodInjection(bd, beanName, owner);
		}
	}
```

#### Spring BeanDefinition 元信息合并后置阶段

MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition 代码片段：

```java
public interface MergedBeanDefinitionPostProcessor extends BeanPostProcessor {

	/**
	 * Post-process the given merged bean definition for the specified bean.
	 * @param beanDefinition the merged bean definition for the bean
	 * @param beanType the actual type of the managed bean instance
	 * @param beanName the name of the bean
	 * @see AbstractAutowireCapableBeanFactory#applyMergedBeanDefinitionPostProcessors
	 */
	void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName);
// 省略部分代码
}
```

#### Spring Bean 实例化后阶段

Spring Bean 实例化后阶段返回值，可控制是否跳过属性填充（populate）阶段。

InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation 代码片段：

```java
	/**
	 * Perform operations after the bean has been instantiated, via a constructor or factory method,
	 * but before Spring property population (from explicit properties or autowiring) occurs.
	 * <p>This is the ideal callback for performing custom field injection on the given bean
	 * instance, right before Spring's autowiring kicks in.
	 * <p>The default implementation returns {@code true}.
	 * @param bean the bean instance created, with properties not having been set yet
	 * @param beanName the name of the bean
	 * @return {@code true} if properties should be set on the bean; {@code false}
	 * if property population should be skipped. Normal implementations should return {@code true}.
	 * Returning {@code false} will also prevent any subsequent InstantiationAwareBeanPostProcessor
	 * instances being invoked on this bean instance.
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see #postProcessBeforeInstantiation
	 */
	default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		return true;
	}
```

<br>
<hr>

### Spring Bean 属性赋值 populate 阶段

populate 阶段，详见 AbstractAutowireCapableBeanFactory#populateBean 代码片段：
```java
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
		// 省略部分代码

        // 依赖查找 BeanDefinition 的属性，填充到 PropertyValues
		PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

		int resolvedAutowireMode = mbd.getResolvedAutowireMode();
		if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
			MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
			// Add property values based on autowire by name if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
				autowireByName(beanName, mbd, bw, newPvs);
			}
			// Add property values based on autowire by type if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
				autowireByType(beanName, mbd, bw, newPvs);
			}
			pvs = newPvs;
		}

		boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
		boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

        // populate 属性填充前置阶段
		PropertyDescriptor[] filteredPds = null;
		if (hasInstAwareBpps) {
			if (pvs == null) {
				pvs = mbd.getPropertyValues();
			}
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
					if (pvsToUse == null) {
						if (filteredPds == null) {
							filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
						}
						pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
						if (pvsToUse == null) {
							return;
						}
					}
					pvs = pvsToUse;
				}
			}
		}
        // 依赖检查
		if (needsDepCheck) {
			if (filteredPds == null) {
				filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
			}
			checkDependencies(beanName, mbd, filteredPds, pvs);
		}

        // pvs 填充属性到 Bean 实例包装类 BeanWrapper
		if (pvs != null) {
			applyPropertyValues(beanName, mbd, bw, pvs);
		}
	}
```

#### Spring Bean 属性赋值 populate 前阶段

Bean 属性赋值前回调：

Spring 1.2 - 5.0： InstantiationAwareBeanPostProcessor#postProcessPropertyValues
Spring 5.1： InstantiationAwareBeanPostProcessor#postProcessProperties

```java

	/**
	 * Post-process the given property values before the factory applies them
	 * to the given bean, without any need for property descriptors.
	 * <p>Implementations should return {@code null} (the default) if they provide a custom
	 * {@link #postProcessPropertyValues} implementation, and {@code pvs} otherwise.
	 * In a future version of this interface (with {@link #postProcessPropertyValues} removed),
	 * the default implementation will return the given {@code pvs} as-is directly.
	 * @param pvs the property values that the factory is about to apply (never {@code null})
	 * @param bean the bean instance created, but whose properties have not yet been set
	 * @param beanName the name of the bean
	 * @return the actual property values to apply to the given bean (can be the passed-in
	 * PropertyValues instance), or {@code null} which proceeds with the existing properties
	 * but specifically continues with a call to {@link #postProcessPropertyValues}
	 * (requiring initialized {@code PropertyDescriptor}s for the current bean class)
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @since 5.1
	 * @see #postProcessPropertyValues
	 */
	@Nullable
	default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
			throws BeansException {

		return null;
	}

	/**
	 * Post-process the given property values before the factory applies them
	 * to the given bean. Allows for checking whether all dependencies have been
	 * satisfied, for example based on a "Required" annotation on bean property setters.
	 * <p>Also allows for replacing the property values to apply, typically through
	 * creating a new MutablePropertyValues instance based on the original PropertyValues,
	 * adding or removing specific values.
	 * <p>The default implementation returns the given {@code pvs} as-is.
	 * @param pvs the property values that the factory is about to apply (never {@code null})
	 * @param pds the relevant property descriptors for the target bean (with ignored
	 * dependency types - which the factory handles specifically - already filtered out)
	 * @param bean the bean instance created, but whose properties have not yet been set
	 * @param beanName the name of the bean
	 * @return the actual property values to apply to the given bean (can be the passed-in
	 * PropertyValues instance), or {@code null} to skip property population
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see #postProcessProperties
	 * @see org.springframework.beans.MutablePropertyValues
	 * @deprecated as of 5.1, in favor of {@link #postProcessProperties(PropertyValues, Object, String)}
	 */
	@Deprecated
	@Nullable
	default PropertyValues postProcessPropertyValues(
			PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {

		return pvs;
	}

```


#### Spring Bean 属性赋值 populate 中阶段

AbstractAutowireCapableBeanFactory#applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs)

PropertyValues 类型转换后，填充到 Bean 实例包装类 BeanWrapper。

<br>
<hr>


### Spring Bean Aware 接口回调阶段

Aware 接口回调，是基于 Aware 接口的一系列接口实现的，可以回调相应 Bean 到该实例化的 Bean 中。


接口回调有俩种回调方式：
- 方式A：AbstractAutowireCapableBeanFactory#invokeAwareMethods(String beanName, Object bean)
- 方式B：ApplicationContextAwareProcessor#postProcessBeforeInitialization，通过 ApplicationContextAwareProcessor 回调


Spring Aware 接口的回调顺序：
- BeanNameAware: 基于方式A，回调 Bean 的 beanName；
- BeanClassLoaderAware: 基于方式A，回调 Bean Class 的类加载器 ClassLoader；
- BeanFactoryAware: : 基于方式A，回调 BeanFactory；
- EnvironmentAware ： 基于方式B，回调 Environment；
- EmbeddedValueResolverAware ： 基于方式B，回调 EmbeddedValueResolver，处理占位符填充以及 spel 表达式；
- ResourceLoaderAware ： 基于方式B，回调 ResourceLoader 资源加载器；
- ApplicationEventPublisherAware ： 基于方式B，回调 ApplicationEventPublisher 应用事件发布器；
- MessageSourceAware ： 基于方式B，回调 MessageSource 国际化相关；
- ApplicationContextAware ： 基于方式B，回调 ApplicationContext 应用上下文；

<br>
<hr>

### Spring Bean 初始化阶段

AbstractAutowireCapableBeanFactory#initializeBean 代码片段：
```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
        // 触发 Aware 回调
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
			invokeAwareMethods(beanName, bean);
		}

        // Bean 初始化前阶段
		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

        // Bean 初始化中阶段
		try {
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}

        // Bean 初始化后置阶段
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}
```

#### Spring Bean 初始化前阶段

Bean 初始化前阶段基于接口 BeanPostProcessor#postProcessBeforeInitialization 实现。

CommonAnnotationBeanPostProcessor 基于此接口，实现了 `@PostConstruct` 注解实例化后置处理。

```java
public class CommonAnnotationBeanPostProcessor extends InitDestroyAnnotationBeanPostProcessor
		implements InstantiationAwareBeanPostProcessor, BeanFactoryAware, Serializable {

        public CommonAnnotationBeanPostProcessor() {
            setOrder(Ordered.LOWEST_PRECEDENCE - 3);
            setInitAnnotationType(PostConstruct.class);// 设置初始化注解，也可自定义
            setDestroyAnnotationType(PreDestroy.class);
            ignoreResourceType("javax.xml.ws.WebServiceContext");
        }
}

public class InitDestroyAnnotationBeanPostProcessor
		implements DestructionAwareBeanPostProcessor, MergedBeanDefinitionPostProcessor, PriorityOrdered, Serializable {

   // 省略部分代码

    // 获取生命周期元信息，执行初始化方法（即标注了 PostConstruct 的方法）
    @Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		LifecycleMetadata metadata = findLifecycleMetadata(bean.getClass());
		try {
			metadata.invokeInitMethods(bean, beanName);
		}
		catch (InvocationTargetException ex) {
			throw new BeanCreationException(beanName, "Invocation of init method failed", ex.getTargetException());
		}
		catch (Throwable ex) {
			throw new BeanCreationException(beanName, "Failed to invoke init method", ex);
		}
		return bean;
	}

    // 省略部分代码

    private class LifecycleMetadata {
        // 触发执行初始化元信息元素
        public void invokeInitMethods(Object target, String beanName) throws Throwable {
            Collection<LifecycleElement> checkedInitMethods = this.checkedInitMethods;
            Collection<LifecycleElement> initMethodsToIterate =
                    (checkedInitMethods != null ? checkedInitMethods : this.initMethods);
            if (!initMethodsToIterate.isEmpty()) {
                for (LifecycleElement element : initMethodsToIterate) {
                    if (logger.isTraceEnabled()) {
                        logger.trace("Invoking init method on bean '" + beanName + "': " + element.getMethod());
                    }
                    element.invoke(target);
                }
            }
        }

        // 省略部分代码
    }
    
    
    private static class LifecycleElement {
        
        // 省略部分代码

        public void invoke(Object target) throws Throwable {
            ReflectionUtils.makeAccessible(this.method);
            this.method.invoke(target, (Object[]) null);
        }
    }
}
    
    
```



#### Spring Bean 初始化中阶段

AbstractAutowireCapableBeanFactory#invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)

```java
protected void invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
			throws Throwable {
        // 若 Bean 实现 InitializingBean#afterPropertiesSet()，则执行
		boolean isInitializingBean = (bean instanceof InitializingBean);
		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
			if (logger.isTraceEnabled()) {
				logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
			}
			if (System.getSecurityManager() != null) {
				try {
					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
						((InitializingBean) bean).afterPropertiesSet();
						return null;
					}, getAccessControlContext());
				}
				catch (PrivilegedActionException pae) {
					throw pae.getException();
				}
			}
			else {
				((InitializingBean) bean).afterPropertiesSet();
			}
		}

        // 执行自定义初始化方法
		if (mbd != null && bean.getClass() != NullBean.class) {
			String initMethodName = mbd.getInitMethodName();
			if (StringUtils.hasLength(initMethodName) &&
					!(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
					!mbd.isExternallyManagedInitMethod(initMethodName)) {
				invokeCustomInitMethod(beanName, bean, mbd);
			}
		}
	}


```


#### Spring Bean 初始化后阶段

BeanPostProcessor#postProcessAfterInitialization

```java

	/**
	 * Apply this {@code BeanPostProcessor} to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other {@code BeanPostProcessor} callbacks.
	 * <p>The default implementation returns the given {@code bean} as-is.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * @see org.springframework.beans.factory.FactoryBean
	 */
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
```


#### Spring Bean 初始化完成阶段

Spring 4.1 +：SmartInitializingSingleton#afterSingletonsInstantiated

在确保单例 Bean 完成整个初始化后调用，此时 Bean 已经完整。


```java
public interface SmartInitializingSingleton {

	/**
	 * Invoked right at the end of the singleton pre-instantiation phase,
	 * with a guarantee that all regular singleton beans have been created
	 * already. {@link ListableBeanFactory#getBeansOfType} calls within
	 * this method won't trigger accidental side effects during bootstrap.
	 * <p><b>NOTE:</b> This callback won't be triggered for singleton beans
	 * lazily initialized on demand after {@link BeanFactory} bootstrap,
	 * and not for any other bean scope either. Carefully use it for beans
	 * with the intended bootstrap semantics only.
	 */
	void afterSingletonsInstantiated();

}
```

```java
/**
* AbstractApplicationContext#refresh =>
* AbstractApplicationContext#finishBeanFactoryInitialization =>
* DefaultListableBeanFactory#preInstantiateSingletons
*/
@Override
	public void preInstantiateSingletons() throws BeansException {
		if (logger.isTraceEnabled()) {
			logger.trace("Pre-instantiating singletons in " + this);
		}

		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

		// 触发所有注册的非延迟（non-lazy）加载的单例 Bean 对象初始化
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
				if (isFactoryBean(beanName)) {
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
					if (bean instanceof FactoryBean) {
						FactoryBean<?> factory = (FactoryBean<?>) bean;
						boolean isEagerInit;
						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
							isEagerInit = AccessController.doPrivileged(
									(PrivilegedAction<Boolean>) ((SmartFactoryBean<?>) factory)::isEagerInit,
									getAccessControlContext());
						}
						else {
							isEagerInit = (factory instanceof SmartFactoryBean &&
									((SmartFactoryBean<?>) factory).isEagerInit());
						}
						if (isEagerInit) {
							getBean(beanName);
						}
					}
				}
				else {
					getBean(beanName);
				}
			}
		}

		// 在所有注册的非延迟（non-lazy）加载的单例 Bean 对象初始化后，触发 afterSingletonsInstantiated 回调
		for (String beanName : beanNames) {
			Object singletonInstance = getSingleton(beanName);
			if (singletonInstance instanceof SmartInitializingSingleton) {
				SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
				if (System.getSecurityManager() != null) {
					AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
						smartSingleton.afterSingletonsInstantiated();
						return null;
					}, getAccessControlContext());
				}
				else {
					smartSingleton.afterSingletonsInstantiated();
				}
			}
		}
	}
```

<br>
<hr>


### Spring Bean 销毁阶段


#### Spring Bean 销毁前阶段

DestructionAwareBeanPostProcessor#postProcessBeforeDestruction

```java
public interface DestructionAwareBeanPostProcessor extends BeanPostProcessor {

	/**
	 * Apply this BeanPostProcessor to the given bean instance before its
	 * destruction, e.g. invoking custom destruction callbacks.
	 * <p>Like DisposableBean's {@code destroy} and a custom destroy method, this
	 * callback will only apply to beans which the container fully manages the
	 * lifecycle for. This is usually the case for singletons and scoped beans.
	 * @param bean the bean instance to be destroyed
	 * @param beanName the name of the bean
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.DisposableBean#destroy()
	 * @see org.springframework.beans.factory.support.AbstractBeanDefinition#setDestroyMethodName(String)
	 */
	void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException;

	/**
	 * Determine whether the given bean instance requires destruction by this
	 * post-processor.
	 * <p>The default implementation returns {@code true}. If a pre-5 implementation
	 * of {@code DestructionAwareBeanPostProcessor} does not provide a concrete
	 * implementation of this method, Spring silently assumes {@code true} as well.
	 * @param bean the bean instance to check
	 * @return {@code true} if {@link #postProcessBeforeDestruction} is supposed to
	 * be called for this bean instance eventually, or {@code false} if not needed
	 * @since 4.3
	 */
	default boolean requiresDestruction(Object bean) {
		return true;
	}

}
```

CommonAnnotationBeanPostProcessor 基于此接口，实现了 `@PreDestroy` 注解的销毁前处理

#### Spring Bean 销毁中阶段

DisposableBeanAdapter#destroy
```java
    @Override
	public void destroy() {
        // 销毁前处理阶段
		if (!CollectionUtils.isEmpty(this.beanPostProcessors)) {
			for (DestructionAwareBeanPostProcessor processor : this.beanPostProcessors) {
				processor.postProcessBeforeDestruction(this.bean, this.beanName);
			}
		}

        // 触发 DisposableBean#destroy
		if (this.invokeDisposableBean) {
			if (logger.isTraceEnabled()) {
				logger.trace("Invoking destroy() on bean with name '" + this.beanName + "'");
			}
			try {
				if (System.getSecurityManager() != null) {
					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
						((DisposableBean) this.bean).destroy();
						return null;
					}, this.acc);
				}
				else {
					((DisposableBean) this.bean).destroy();
				}
			}
			catch (Throwable ex) {
				String msg = "Invocation of destroy method failed on bean with name '" + this.beanName + "'";
				if (logger.isDebugEnabled()) {
					logger.warn(msg, ex);
				}
				else {
					logger.warn(msg + ": " + ex);
				}
			}
		}

        // 触发自定义销毁方法
		if (this.destroyMethod != null) {
			invokeCustomDestroyMethod(this.destroyMethod);
		}
		else if (this.destroyMethodName != null) {
			Method methodToInvoke = determineDestroyMethod(this.destroyMethodName);
			if (methodToInvoke != null) {
				invokeCustomDestroyMethod(ClassUtils.getInterfaceMethodIfPossible(methodToInvoke));
			}
		}
	}

```

<br>
<hr>

### Spring Bean 垃圾回收阶段

JVM GC 垃圾回收 Bean 实例。


<br>
<hr>

### 代码

<a href="https://github.com/jackhance0825/thinking-in-spring-5.2.16/tree/main/bean-lifecycle" target="_blank" > 本篇章代码 </a>
