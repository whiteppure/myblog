---
title: "SpringBoot详解"
date: 2021-00-00
draft: true
tags: ["Java", "spring"]
slug: "java-spring"
---
## 注解

### @SpringBootApplication
这个注解在启动类上
```
@SpringBootApplication
public class SpringBootExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootExampleApplication.class, args);
    }
}
```
`@SpringBootApplication`是一个复合注解，由其他注解构成。核心注解是`@SpringBootConfiguration`和`@EnableAutoConfiguration`
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication{
}
```
#### @SpringBootConfiguration
`@SpringBootConfiguration`核心注解是`@Configuration`代表自己是一个`Spring`的配置类
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}
```
`@Configuration`底层实现就是一个`Component`
> 指示带注释的类是一个“组件”。
  在使用基于注释的配置和类路径扫描时，这些类被视为自动检测的候选类。
```
/**
 * Indicates that an annotated class is a "component".
 * Such classes are considered as candidates for auto-detection
 * when using annotation-based configuration and classpath scanning.
 *
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component 
```

#### @EnableAutoConfiguration
核心注解是`@AutoConfigurationPackage`和`@Import({AutoConfigurationImportSelector.class})`
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
}
```
`@AutoConfigurationPackage`注解核心是引入了一个`@Import(AutoConfigurationPackages.Registrar.class)`配置类,该类实现了`ImportBeanDefinitionRegistrar`接口
```
	/**
	 * {@link ImportBeanDefinitionRegistrar} to store the base package from the importing
	 * configuration.
	 */
	static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {
		@Override
		public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
			register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
		}
		@Override
		public Set<Object> determineImports(AnnotationMetadata metadata) {
			return Collections.singleton(new PackageImports(metadata));
		}

	}
```
>这里可以打断点自己看一下

`@AutoConfigurationPackage` 这个注解本身的含义就是将主配置类（`@SpringBootApplication`标注的类）所在的包下面所有的组件都扫描到 `spring` 容器中。

`AutoConfigurationImportSelector`核心代码如下
```
	/**
	 * Return the auto-configuration class names that should be considered. By default
	 * this method will load candidates using {@link SpringFactoriesLoader} with
	 * {@link #getSpringFactoriesLoaderFactoryClass()}.
	 * @param metadata the source metadata
	 * @param attributes the {@link #getAttributes(AnnotationMetadata) annotation
	 * attributes}
	 * @return a list of candidate configurations
	 */
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}

	/**
	 * Return the class used by {@link SpringFactoriesLoader} to load configuration
	 * candidates.
	 * @return the factory class
	 */
	protected Class<?> getSpringFactoriesLoaderFactoryClass() {
		return EnableAutoConfiguration.class;
	}
	protected ClassLoader getBeanClassLoader() {
		return this.beanClassLoader;
	}
```
`getSpringFactoriesLoaderFactoryClass`方法返回`EnableAutoConfiguration.class`目的就是为了将启动类所需的所有资源导入。

在`getCandidateConfigurations`中有如下代码
```
Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
```

大意：在`META-INF/spring.factories`中没有发现自动配置类。如果您使用的是自定义打包，请确保该文件是正确的。
![找到spring.factories](/myblog/posts/images/essays/找到spring.factories.png)

`spring.factories`包含了很多类，但不是全部都加载的，在某些类里面，是有一个条件`@ConditionalOnXXX`注解，只有当这个注解上的条件满足才会加载。

例如：`SpringApplicationAdminJmxAutoConfiguration`
```
@Configuration(proxyBeanMethods = false)
@AutoConfigureAfter(JmxAutoConfiguration.class)
@ConditionalOnProperty(prefix = "spring.application.admin", value = "enabled", havingValue = "true",
		matchIfMissing = false)
public class SpringApplicationAdminJmxAutoConfiguration 
```

#### 总结
![@SpringbootApplication原理](/myblog/posts/images/essays/@SpringbootApplication原理.png)

当Springboot启动的时候，会执行`AutoConfigurationImportSelector`这个类中的`getCandidateConfigurations`方法，这个方法会帮我们加载`META-INF/spring.factories`文件里面的当`@ConditionXXX`注解条件满足的类。

### @Autowired

### @Resource

### @Qualify