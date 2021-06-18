---
title: "SpringBoot详解"
date: 2021-05-13
draft: false
tags: ["Java", "springboot"]
slug: "java-springboot"
---
## 概述
官网地址：[https://spring.io/projects/spring-boot](https://spring.io/projects/spring-boot)

> `SpringBoot`是由`Pivotal`团队提供的全新框架，其设计目的是用来简化新`Spring`应用的初始搭建以及开发过程。
该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。`SpringBoot` 提供了一种新的编程范式，可以更加快速便捷地开发 `Spring` 项目，在开发过程当中可以专注于应用程序本身的功能开发，而无需在 `Spring` 配置上花太大的工夫。

`SpringBoot` 基于 `Sring4` 进行设计，继承了原有 `Spring` 框架的优秀基因。
`SpringBoot` 准确的说并不是一个框架，而是一些类库的集合。
`maven` 或者 `gradle` 项目导入相应依赖即可使用 `SpringBoot`，而无需自行管理这些类库的版本。

特点：
- 独立运行的 `Spring` 项目：
`SpringBoot` 可以以 jar 包的形式独立运行，运行一个 `SpringBoot` 项目只需通过 `java–jar xx.jar` 来运行。
- 内嵌 `Servlet` 容器：
`SpringBoot` 可选择内嵌 `Tomcat`、`Jetty` 或者 `Undertow`，这样我们无须以 `war` 包形式部署项目。
- 提供 `starter` 简化 `Maven` 配置：
`Spring` 提供了一系列的 `starter` pom 来简化 `Maven` 的依赖加载，例如，当你使用了`spring-boot-starter-web` 时，会自动加入依赖包。
- 自动配置 `Spring`：
`SpringBoot` 会根据在类路径中的 jar 包、类，为 jar 包里的类自动配置 Bean，这样会极大地减少我们要使用的配置。当然，`SpringBoot` 只是考虑了大多数的开发场景，并不是所有的场景，若在实际开发中我们需要自动配置 `Bean`，而 `SpringBoot` 没有提供支持，则可以自定义自动配置。
- 准生产的应用监控：
`SpringBoot` 提供基于 `http、ssh、telnet` 对运行时的项目进行监控。
- 无代码生成和 xml 配置：
`SpringBoot` 的神奇的不是借助于代码生成来实现的，而是通过条件注解来实现的，这是 `Spring 4.x` 提供的新特性。`Spring 4.x` 提倡使用 Java 配置和注解配置组合，而 `SpringBoot` 不需要任何 xml 配置即可实现 `Spring` 的所有配置。

## @SpringBootApplication原理
`@SpringBootApplication`这个注解通常标注在启动类上：
```
@SpringBootApplication
public class SpringBootExampleApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootExampleApplication.class, args);
    }
}
```
`@SpringBootApplication`是一个复合注解，即由其他注解构成。核心注解是`@SpringBootConfiguration`和`@EnableAutoConfiguration`
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
### @SpringBootConfiguration
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

### @EnableAutoConfiguration
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

### 总结
![@SpringbootApplication原理](/myblog/posts/images/essays/@SpringbootApplication原理.png)

当 `Springboot` 启动的时候，会执行`AutoConfigurationImportSelector`这个类中的`getCandidateConfigurations`方法，这个方法会帮我们加载`META-INF/spring.factories`文件里面的当`@ConditionXXX`注解条件满足的类。

## Bean的自动装配
`Spring`利用依赖注入（DI），完成对IOC容器中各个组件的依赖关系赋值。

`Spring`提供三种装配方式：
- 基于注解的自动装配
- 基于 XML 配置的显式装配
- 基于 Java 配置的显式装配

本篇博客，详细介绍基于注解的自动装配
| 自动装配   | 来源                          | 支持@Primary | springboot支持属性 |
| ---------- | ----------------------------- | ------------ | ------------------ |
| @Autowired | Springboot原生                | 支持         | boolean required   |
| @Resource  | JSR-250，JDK自带              | 不支持       | String name         |
| @Inject    | JSR-330，需要导入javax.inject | 支持         | 无其他属性         |

### @Autowired
可以放在构造器、参数、方法、属性上

源码如下：
```
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {

	/**
	 * Declares whether the annotated dependency is required.
	 * <p>Defaults to {@code true}.
	 */
	boolean required() default true;

}
```
#### 加在属性上
使用`@Autowired`注解通常将其加载属性上或者构造器上，让其自动注入；默认是按照类型去容器中寻找对应的组件，例如：
```

public class SpringBootExampleApplication {


    public static void main(String[] args) {
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(TestConfig.class);

        TestService testService = annotationConfigApplicationContext.getBean(TestService.class);
        // TestService 实例=====>TestService(testDao=TestDao(name=default))
        System.out.println("TestService 实例=====>" +testService);

    }

}


// 扫描的包名称
@ComponentScan({"com.example.springboot.example.task"})
@Configuration
class TestConfig{

}


@ToString
@Service
class TestService {

    @Autowired
    TestDao testDao;

}

@ToString
@Repository
class TestDao{

    @Getter
    @Setter
    private String name = "default";

}
```
如果容器中有多个组件的名称相同,可以通过`@Qualifier`来进行选择注入；
```
public class SpringBootExampleApplication {


    public static void main(String[] args) {
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(TestConfig.class);

        TestService testService = annotationConfigApplicationContext.getBean(TestService.class);
        // TestService 实例=====>TestService(testDao=TestDao(name=我是默认的TestDao))
        System.out.println("TestService 实例=====>" +testService);

    }

}


@ComponentScan({"com.example.springboot.example.task"})
@Configuration
class TestConfig{

    @Bean(name = "testDao2")
     public TestDao testDao(){
        TestDao testDao = new TestDao();
        testDao.setName("我是testDao2");
        return testDao;
    }
}

@ToString
@Service
class TestService {

    @Autowired
    @Qualifier("testDao")
    TestDao testDao;

}

@ToString
@Repository
class TestDao{

    @Getter
    @Setter
    private String name = "我是默认的TestDao";

}
```
除了使用`@Qualifier`来进行选择注入外，也可以使用`@Primary`来设置 bean 的优先级，默认情况下指定让哪个 bean 优先注入；

`@Primary`注解是在没有明确指定的情况下，默认使用的 bean，如果你明确用`@Qualifier`指定，则会使用`@Qualifier`指定的bean；
确保测试结果准确，在使用`@Primary`时，将`@Qualifier`去掉。
```
public class SpringBootExampleTaskApplication {


    public static void main(String[] args) {
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(TestConfig.class);

        TestService testService = annotationConfigApplicationContext.getBean(TestService.class);
//        TestService 实例=====>TestService(testDao=TestDao(name=我是testDao2))
        System.out.println("TestService 实例=====>" +testService);

    }

}


@ComponentScan({"com.example.springboot.example.task"})
@Configuration
class TestConfig{

    @Primary
    @Bean(name = "testDao2")
     public TestDao testDao(){
        TestDao testDao = new TestDao();
        testDao.setName("我是testDao2");
        return testDao;
    }
}

@ToString
@Service
class TestService {

    @Autowired
    TestDao testDao;

}

@ToString
@Repository
class TestDao{

    @Getter
    @Setter
    private String name = "我是默认的TestDao";

}
```

如果使用`@Autowired`在容器中没有对应的组件名称，默认情况下会报错。
```
nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException
```
如果没有找到对应的 bean 不报错，可以通过`@Autowired(required = false)`来进行设置
```
@ToString
@Service
class TestService {
    
    // TestService 实例=====>TestService(testDao=null)

    @Autowired(required = false)
    TestDao testDao;

}
```
#### 加在方法、参数上
`@Autowired`注解不仅可以标注在属性上，也可以标注在方法上，当标注在方法上时，Spring容器创建当前对象，就会调用该方法完成赋值，方法使用的参数从IOC容器中获取。

通过测试打印对象的地址可以看到，方法中的参数确实是从IOC容器中获取的。
```
public class SpringBootExampleTaskApplication {


    public static void main(String[] args) {
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(TestConfig.class);

        TestService testService = annotationConfigApplicationContext.getBean(TestService.class);
        System.out.println("TestService 中的实例=====>" +testService);

        TestDao testDao = annotationConfigApplicationContext.getBean(TestDao.class);
        System.out.println("TestDao 中的实例=====>" +testDao);
        
        // TestService 中的实例=====>TestService(testDao=com.example.springboot.example.task.TestDao@5fe94a96)
        //TestDao 中的实例=====>com.example.springboot.example.task.TestDao@5fe94a96


    }

}


@ComponentScan({"com.example.springboot.example.task"})
@Configuration
class TestConfig{
}

@ToString
@Service
class TestService {


    TestDao testDao;

    @Autowired
    public void setTestDao(TestDao testDao) {
        this.testDao = testDao;
    }
}


@Repository
class TestDao{

    private String name = "我是默认的TestDao";

}
```
也可以加在参数上，与加在方法上类似也是从IOC容器中获取该对象。
```
@ToString
@Service
class TestService {


    TestDao testDao;

    public void setTestDao(@Autowired TestDao testDao) {
        this.testDao = testDao;
    }
}
```
在`Spring`创建对象的时候会默认调用组件的无参构造方法，如果只有一个有参构造，如果想要创建对象，则必须调用该有参构造；
所以当一个组件只有一个有参构造时，则可以不用写`@Autowired`注解。
```
@ToString
@Service
class TestService {


    TestDao testDao;
    
     // @Autowired
    public TestService(TestDao testDao) {
        this.testDao = testDao;
    }

}
```

除了通过构造方法的方式实例化组件，也可以通过用bean标注的形式，来实例化容器中的组件。
```
public class SpringBootExampleTaskApplication {


    public static void main(String[] args) {
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(TestConfig.class);

        TestService testService = annotationConfigApplicationContext.getBean(TestService.class);
        System.out.println("TestService 中的实例=====>" +testService);

        TestDao testDao = annotationConfigApplicationContext.getBean(TestDao.class);
        System.out.println("TestDao 中的实例=====>" +testDao);

        TestDao1 testDao1 = annotationConfigApplicationContext.getBean(TestDao1.class);
        System.out.println("TestDao1 中的实例=====>" +testDao);
        
        // TestService 中的实例=====>TestService(testDao=com.example.springboot.example.task.TestDao@639c2c1d)
        //TestDao 中的实例=====>com.example.springboot.example.task.TestDao@639c2c1d
        //TestDao1 中的实例=====>com.example.springboot.example.task.TestDao@639c2c1d

    }

}


@ComponentScan({"com.example.springboot.example.task"})
@Configuration
class TestConfig{

    @Bean
    public TestDao1 testDao1(TestDao testDao){
        TestDao1 testDao1 = new TestDao1();
        testDao1.setTestDao(testDao);
        return testDao1;
    }
}

@ToString
@Service
class TestService {


    TestDao testDao;

    @Autowired
    public TestService(TestDao testDao) {
        this.testDao = testDao;
    }

}


@Component
class TestDao{
}

class TestDao1{
    @Setter
    TestDao testDao;
}
```


#### 原理
```
/
 * @see AutowiredAnnotationBeanPostProcessor
 */
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired{}
```
在`@Autowired`注解文档注释上面，可以看到与之息息相关的一个类`AutowiredAnnotationBeanPostProcessor`，即`@Autowired`后置处理器；
可以看到该类实现了`MergedBeanDefinitionPostProcessor`接口，在`postProcessMergedBeanDefinition`方法上打一个断点，就可以看到`@Autowired`的调用栈。

`@Autowired`注解调用栈：
```
AbstractApplicationContext.refresh(容器初始化)
---> registerBeanPostProcessors (注册AutowiredAnnotationBeanPostProcessor) 
---> finishBeanFactoryInitialization
---> AbstractAutowireCapableBeanFactory.doCreateBean
---> AbstractAutowireCapableBeanFactory.applyMergedBeanDefinitionPostProcessors
---> MergedBeanDefinitionPostProcessor.postProcessMergedBeanDefinition
---> AutowiredAnnotationBeanPostProcessor.findAutowiringMetadata
```

核心调用：
```
postProcessMergedBeanDefinition`->`findAutowiringMetadata`->`buildAutowiringMetadata
```
相关源码：
```
@Override
public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
    // 调用 findAutowiringMetadata
    InjectionMetadata metadata = findAutowiringMetadata(beanName, beanType, null);
    metadata.checkConfigMembers(beanDefinition);
}

private InjectionMetadata findAutowiringMetadata(String beanName, Class<?> clazz, @Nullable PropertyValues pvs) {
		// Fall back to class name as cache key, for backwards compatibility with custom callers.
		String cacheKey = (StringUtils.hasLength(beanName) ? beanName : clazz.getName());
		// Quick check on the concurrent map first, with minimal locking.
		InjectionMetadata metadata = this.injectionMetadataCache.get(cacheKey);
		if (InjectionMetadata.needsRefresh(metadata, clazz)) {
			synchronized (this.injectionMetadataCache) {
				metadata = this.injectionMetadataCache.get(cacheKey);
				if (InjectionMetadata.needsRefresh(metadata, clazz)) {
					if (metadata != null) {
						metadata.clear(pvs);
					}
                    // 调用buildAutowiringMetadata
					metadata = buildAutowiringMetadata(clazz);
					this.injectionMetadataCache.put(cacheKey, metadata);
				}
			}
		}
		return metadata;
	}



private InjectionMetadata buildAutowiringMetadata(final Class<?> clazz) {
		LinkedList<InjectionMetadata.InjectedElement> elements = new LinkedList<>();
		Class<?> targetClass = clazz;//需要处理的目标类
       
		do {
			final LinkedList<InjectionMetadata.InjectedElement> currElements = new LinkedList<>();
 
            /*通过反射获取该类所有的字段，并遍历每一个字段，并通过方法findAutowiredAnnotation遍历每一个字段的所用注解，并如果用autowired修饰了，则返回auotowired相关属性*/  
 
			ReflectionUtils.doWithLocalFields(targetClass, field -> {
				AnnotationAttributes ann = findAutowiredAnnotation(field);
				if (ann != null) {//校验autowired注解是否用在了static方法上
					if (Modifier.isStatic(field.getModifiers())) {
						if (logger.isWarnEnabled()) {
							logger.warn("Autowired annotation is not supported on static fields: " + field);
						}
						return;
					}//判断是否指定了required
					boolean required = determineRequiredStatus(ann);
					currElements.add(new AutowiredFieldElement(field, required));
				}
			});
            //和上面一样的逻辑，但是是通过反射处理类的method
			ReflectionUtils.doWithLocalMethods(targetClass, method -> {
				Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
				if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
					return;
				}
				AnnotationAttributes ann = findAutowiredAnnotation(bridgedMethod);
				if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
					if (Modifier.isStatic(method.getModifiers())) {
						if (logger.isWarnEnabled()) {
							logger.warn("Autowired annotation is not supported on static methods: " + method);
						}
						return;
					}
					if (method.getParameterCount() == 0) {
						if (logger.isWarnEnabled()) {
							logger.warn("Autowired annotation should only be used on methods with parameters: " +
									method);
						}
					}
					boolean required = determineRequiredStatus(ann);
					PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
              	    currElements.add(new AutowiredMethodElement(method, required, pd));
				}
			});
    //用@Autowired修饰的注解可能不止一个，因此都加在currElements这个容器里面，一起处理		
			elements.addAll(0, currElements);
			targetClass = targetClass.getSuperclass();
		}
		while (targetClass != null && targetClass != Object.class);
 
		return new InjectionMetadata(clazz, elements);
	}
```
当`Spring` 容器启动时，`AutowiredAnnotationBeanPostProcessor` 组件会被注册到容器中，然后扫描代码，如果带有 `@Autowired` 注解，则将依赖注入信息封装到 `InjectionMetadata` 中。

最后创建 `bean`，即实例化对象和调用初始化方法，会调用各种 `XXXBeanPostProcessor` 对 `bean` 初始化，其中包括`AutowiredAnnotationBeanPostProcessor`，它负责将相关的依赖注入到容器中。

### @Resource、@Inject
Spring 自动装配除了`@Autowired`注解外，也支持JSR-250中的`@Resource`和JSR-330中的`@Inject`注解，来进行自动装配；

```
public class SpringBootExampleTaskApplication {


    public static void main(String[] args) {
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(TestConfig.class);

        TestService testService = annotationConfigApplicationContext.getBean(TestService.class);
//        TestService 实例=====>TestService(testDao=TestDao(name=我是默认的TestDao))
        System.out.println("TestService 实例=====>" +testService);

    }

}


@ComponentScan({"com.example.springboot.example.task"})
@Configuration
class TestConfig{

    @Bean(name = "testDao2")
     public TestDao testDao(){
        TestDao testDao = new TestDao();
        testDao.setName("我是testDao2");
        return testDao;
    }
}

@ToString
@Service
class TestService {


    @Resource
    TestDao testDao;

}


@ToString
@Repository
class TestDao{

    @Getter
    @Setter
    private String name = "我是默认的TestDao";

}
```

使用`@Inject`注解需要导入:
```
<dependency>
    <groupId>javax.inject</groupId>
    <artifactId>javax.inject</artifactId>
    <version>1</version>
</dependency>
```
```
@ToString
@Service
class TestService {

    // TestService 实例=====>TestService(testDao=TestDao(name=我是默认的TestDao))
    @Inject
    TestDao testDao;

}
```

 
### 使用Spring底层组件
通过实现`Aware`接口的子接口，来使用Spring的底层的组件。`Aware`接口类似于回调方法的形式在 Spring 加载的时候将我们自定以的组件加载。
```
/**
 * A marker superinterface indicating that a bean is eligible to be notified by the
 * Spring container of a particular framework object through a callback-style method.
 * The actual method signature is determined by individual subinterfaces but should
 * typically consist of just one void-returning method that accepts a single argument.
 */
public interface Aware {

}
```
![Aware子接口](/myblog/posts/images/essays/Aware子接口.png)

使用测试
```
@Component
class TestService implements ApplicationContextAware, EmbeddedValueResolverAware, BeanFactoryAware {

    public TestService() {
    }

    ApplicationContext applicationContext;


    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("获取实例名称===>" + beanFactory.getBean("TestService"));
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
        System.out.println("获取容器对象===> "+ applicationContext);
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        System.out.println(resolver.resolveStringValue("我是${os.name}，今年${10*2.1}岁"));
    }
}
```

关于这些`Aware`都是使用`AwareProcessor`进行处理的,比如:`ApplicationContextAwareProcessor`就是处理`ApplicationContextAware`接口的。

## Bean的生命周期
`Bean`的生命周期，即`Bean`的创建->初始化->销毁的过程。

### 注入Bean
我们可以使用 xml 配置的方式来指定，`bean` 在初始化、销毁的时候调用对应的方法：
```
<bean id="getDemoEntity" class="com.my.demo" init-method="init" destroy-method="destroy" />
```
也可以使用注解的方式，来调用bean在初始化、销毁的时候调用对应的方法：
```
public class MainTest {
    public static void main(String[] args) {
        // 获取Spring IOC容器
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(DemoConfiguration.class);
        System.out.println("容器初始化完成...");

        annotationConfigApplicationContext.close();
        System.out.println("容器销毁了...");
    }
}

@Configuration
class DemoConfiguration {
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public DemoEntity getDemoEntity() {
        return new DemoEntity();
    }
}

class DemoEntity {
    public DemoEntity(){
        System.out.println("调用了构造器...");
    }

    public void init(){
        System.out.println("调用了初始化方法...");
    }

    public void destroy(){
        System.out.println("调用了销毁方法...");
    }
}
```
需要注意的是，上面演示的是单实例 `bean`，如果是多实例 `bean`，初始化和销毁会不一样。

单实例 `bean`：
- 在容器启动的时候创建对象；
- 在容器关闭的时候销毁；

多实例 `bean`：
- 在每次获取bean的时候创建对象；
- 容器不会自动帮你处理，需要手动销毁 `bean`；

多实例注解代码：
```
@Scope("prototype")
@Bean(initMethod="init",destroyMethod="destroy")
public Test test（）{}
```

### InitializingBean、DisposableBean
通过让`Bean`实现 `InitializingBean`(定义初始化逻辑)和实现`DisposableBean`(销毁逻辑)实现初始化`bean`和销毁`bean`:

```
public class MainTest {
    public static void main(String[] args) {
        // 获取Spring IOC容器
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(DemoEntity.class);
        System.out.println("容器初始化完成...");

        annotationConfigApplicationContext.close();
        System.out.println("容器销毁了...");
    }
}

@Component
class DemoEntity implements InitializingBean, DisposableBean {
    public DemoEntity(){
        System.out.println("调用了构造器...");
    }

    @Override
    public void destroy(){
        System.out.println("调用了销毁方法...");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("调用了初始化方法...");
    }
}
```
### @PostConstruct、@PreDestroy
Java提供了对应的注解，也可以调用`Bean`的初始化方法和销毁方法：
- `@PostConstruct` 标注该注解的方法，在`bean`创建完成并且属性赋值完成 来执行初始化方法;
- `@PreDestroy`， 在容器销毁`bean`之前通知我们进行`bean`的清理工作;

这两个注解不是`spring`的注解是`JSR250`JDK带的注解。
```
public class MainTest {
    public static void main(String[] args) {
        // 获取Spring IOC容器
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(DemoEntity.class);
        System.out.println("容器初始化完成...");

        annotationConfigApplicationContext.close();
        System.out.println("容器销毁了...");
    }
}

@Component
class DemoEntity  {
    public DemoEntity(){
        System.out.println("调用了构造器...");
    }

    // 销毁之前调用
    @PreDestroy
    public void destroy(){
        System.out.println("调用了销毁方法...");
    }

    // 对象创建并赋值之后调用
    @PostConstruct
    public void init() {
        System.out.println("调用了初始化方法...");
    }
}
```

### BeanPostProcessor
除了上面的几种方法，也可以使用`BeanPostProcessor`,`Bean`的后置处理器，在初始化前后进行处理工作。

`postProcessBeforeInitialization`：会在初始化完成之前调用
`postProcessAfterInitialization`：会在初始化完成之后调用
```
public class MainTest {
    public static void main(String[] args) {
        // 获取Spring IOC容器
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(DemoConfiguration.class);
        System.out.println("容器初始化完成...");

        annotationConfigApplicationContext.close();
        System.out.println("容器销毁了...");
    }
}

@Configuration
class DemoConfiguration implements BeanPostProcessor {

    @Bean(initMethod = "init", destroyMethod = "destroy")
    public DemoEntity getDemoEntity(){
       return new DemoEntity();
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("调用了 postProcessBeforeInitialization");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("调用了 postProcessAfterInitialization");
        return bean;
    }
}

@Component
class DemoEntity  {
    public DemoEntity(){
        System.out.println("调用了构造器...");
    }

    public void destroy(){
        System.out.println("调用了销毁方法...");
    }

    public void init() {
        System.out.println("调用了初始化方法...");
    }
}
```
调用顺序：
>创建对象 --> postProcessBeforeInitialization --> 初始化 --> postProcessAfterInitialization --> 销毁

#### 原理
通过打断点，可以看到，在创建`bean`的时候会，会调用`AbstractAutowireCapableBeanFactory`类的`doCreateBean`方法，这也是创建`bean`的核心方法。
```
    try {
        populateBean(beanName, mbd, instanceWrapper);
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }

    // ======= initializeBean  =======
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    try {
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
                (mbd != null ? mbd.getResourceDescription() : null),
                beanName, "Invocation of init method failed", ex);
    }
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
```

调用栈大致如下：
```
populateBean()
{
    applyBeanPostProcessorsBeforeInitialization() -> invokeInitMethods() -> applyBeanPostProcessorsAfterInitialization()
}
```
在初始化之前调用`populateBean()`方法,给`bean`进行属性赋值,之后在调用`applyBeanPostProcessorsBeforeInitialization`方法；

`applyBeanPostProcessorsBeforeInitialization`源码：
```
	public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessBeforeInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}
```
该方法作用，遍历容器中所有的`BeanPostProcessor`挨个执行`postProcessBeforeInitialization`方法，一旦返回`null`，将不会执行后面`bean`的`postProcessBeforeInitialization`方法。

之后在调用`invokeInitMethods`方法，进行`bean`的初始化，最后在执行`applyBeanPostProcessorsAfterInitialization`方法，执行一些初始化之后的工作。

## AOP
AOP,全称：`Aspect-Oriented Programming`，译为面向切面编程 。AOP可以说是对OOP的补充和完善。在程序原有的纵向执行流程中,针对某一个或某些方法添加通知(方法),形成横切面的过程就叫做面向切面编程。

实现AOP的技术，主要分为两大类： 一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“切面”，从而使得编译器可以在编译期间织入有关“切面”的代码，属于静态代理。

作用：
- 将复杂的需求分解出不同的方面，将公共功能集中解决。例如：处理日志。
- 采用代理机制组装起来运行，在不改变原程序的基础上对代码段进行增强处理，增加新的功能。

### 动态代理
动态代理，可以说是AOP的核心了。在`Spring`中主要使用了两种[动态代理](https://whiteppure.github.io/myblog/posts/rookie/rookie-object-oriented/#动态代理)：
- JDK 动态代理技术
- CGLib 动态代理技术

JDK的动态代理时基于Java 的反射机制来实现的，是Java 原生的一种代理方式。他的实现原理就是让代理类和被代理类实现同一接口，代理类持有目标对象来达到方法拦截的作用。
通过接口的方式有两个弊端一个就是必须保证被代理类有接口，另一个就是如果相对被代理类的方法进行代理拦截，那么就要保证这些方法都要在接口中声明。接口继承的是`java.lang.reflect.InvocationHandler`。

CGLib 动态代理使用的 ASM 这个非常强大的 Java 字节码生成框架来生成`class` ，基于继承的实现动态代理，可以直接通过 super 关键字来调用被代理类的方法.子类可以调用父类的方法,不要求有接口。

### 使用
使用AOP大致可以分为三步：
1. 将业务逻辑组件和切面类都加入到容器中，并用`@Aspect`注解标注切面类。
2. 在切面类的通知方法上，要注意切面表达式的写法，标注通知注解，告诉`Spring`何时何地的运行：
    - `@Before`:前置通知，在目标方法运行之前执行；
    - `@After`: 后置通知，在目标方法运行之后执行，无论方法是否出现异常都会执行；
    - `@Around`: 环绕通知，通过`joinPoint.proceed()`方法手动控制目标方法的执行；
    - `@AfterThrowing`: 异常通知，在目标方法出现异常之后执行；
    - `@AfterReturning`: 返回通知，在目标方法返回之后执行；
3. 使用`@EnableAspectJAutoProxy`开启基于注解的AOP模式。
 

```
public class MainTest {
    public static void main(String[] args) {
        // 获取Spring IOC容器
        AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(DemoConfiguration.class);

        DemoEntity demoEntity = annotationConfigApplicationContext.getBean(DemoEntity.class);
        demoEntity.myAspectTest("123");

        annotationConfigApplicationContext.close();
    }
}

@EnableAspectJAutoProxy
@Configuration
class DemoConfiguration{

    @Bean
    public DemoEntity getDemoEntity(){
        return new DemoEntity();
    }

    @Bean
    public DemoAspect gerDemoAspect(){
        return new DemoAspect();
    }

}

@Aspect
class DemoAspect {

    @Pointcut("execution(* com.lilian.ticket.image.exchange.DemoEntity.myAspectTest(..))")
    public void pointer() {}

    @Before("pointer()")
    public void beforeTest(JoinPoint joinPoint) {
        System.out.println("调用了AOP，前置通知");
        Object[] args = joinPoint.getArgs();
        System.out.println("前置通知:目标方法参数：[" + args[0] + "]");
    }

    @After("pointer()")
    public void afterTest(JoinPoint joinPoint){
        System.out.println("调用了AOP，后置通知");
        Object[] args = joinPoint.getArgs();
        System.out.println("后置通知:目标方法参数：[" + args[0] + "]");
    }

    @Around("pointer()")
    public Object aroundTest(ProceedingJoinPoint joinPoint) {
        System.out.println("===调用了AOP，环绕通知===");
        System.out.println("环绕通知目标方法执行前");
        Object[] args = joinPoint.getArgs();
        System.out.println("环绕通知:目标方法参数：[" + args[0] + "]");
        Object proceed = null;
        try {
             proceed = joinPoint.proceed();
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
        System.out.println("环绕通知目标方法执行后\n");
        return proceed;
    }

    @AfterThrowing(pointcut="pointer()", throwing="ex")
    public void afterThrowingTest(JoinPoint joinPoint, Exception ex) {
        System.out.println("异常通知==>["+ex.getMessage()+"]\n");
    }

    @AfterReturning("pointer()")
    public void afterReturnTest(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        System.out.println("有返回值的后置通知:目标方法参数：[" + args[0] + "]");
    }

}

class DemoEntity {

    public String myAspectTest(String name) {
        System.out.println("调用了 myAspectTest 方法;\t name=[" + name + "]");
        // 当name传入null时，模拟异常
        name.split("123");
        return name;
    }
}
```
### @EnableAspectJAutoProxy
要想AOP起作用，就要加`@EnableAspectJAutoProxy`注解，所以AOP的原理可以从`@EnableAspectJAutoProxy`入手研究。

它是一个复合注解，启动的时候，给容器中导入了一个`AspectJAutoProxyRegistrar`组件：
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)
public @interface EnableAspectJAutoProxy {}
```

发现该类实现了`ImportBeanDefinitionRegistrar`接口，而该接口的作用是给容器中注册`bean`的；所以`AspectJAutoProxyRegistrar`作用是，添加自定义组件给容器中注册`bean`。
```
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	/**
	 * Register, escalate, and configure the AspectJ auto proxy creator based on the value
	 * of the @{@link EnableAspectJAutoProxy#proxyTargetClass()} attribute on the importing
	 * {@code @Configuration} class.
	 */
	@Override
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

        // 注册了 AnnotationAwareAspectJAutoProxyCreator 组件
		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
        // 获取 @EnableAspectJAutoProxy 中的属性，做一些工作
		if (enableAspectJAutoProxy != null) {
			if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
				AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
			}
			if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
				AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
			}
		}
	}
}
```
**`AspectJAutoProxyRegistrar`组件何时注册？**

通过对下面代码打断点
```
AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
```
可以看到该方法是给容器中注册了一个`AnnotationAwareAspectJAutoProxyCreator`组件，实际上是注册`AnnotationAwareAspectJAutoProxyCreator`组件。

![AOP核心组件1](/myblog/posts/images/essays/AOP核心组件.png)

可以看出`@EnableAspectJAutoProxy`注解最主要的作用实际上就是通过`@Import`注解把`AnnotationAwareAspectJAutoProxyCreator`这个对象注入到`spring`容器中。

现在只要把`AnnotationAwareAspectJAutoProxyCreator`组件何时注册搞懂，`AspectJAutoProxyRegistrar`组件何时注册也就明白了。

`AnnotationAwareAspectJAutoProxyCreator`继承关系：
```
AnnotationAwareAspectJAutoProxyCreator
    extends AspectJAwareAdvisorAutoProxyCreator
        extends AbstractAdvisorAutoProxyCreator
            extends AbstractAutoProxyCreator
                extends ProxyProcessorSupport implements SmartInstantiationAwareBeanPostProcessor,BeanFactoryAware
                    extends ProxyConfig implements Ordered, BeanClassLoaderAware, AopInfrastructureBean 
```
可以看到其中的一个父类`AbstractAutoProxyCreator`这个父类实现了`SmartInstantiationAwareBeanPostProcessor`接口，该接口是一个后置处理器接口；同样实现了`BeanFactoryAware`接口，这意味着，该类可以通过接口中的方法进行自动装配`BeanFactory`。

这两个接口的在AOP体系中具体的实现方法：
```
1.AbstractAutoProxyCreator
BeanFactoryAware重写：
- AbstractAutoProxyCreator.setBeanFactory

SmartInstantiationAwareBeanPostProcessor重写:
- AbstractAutoProxyCreator.postProcessBeforeInstantiation
- AbstractAutoProxyCreator.postProcessAfterInitialization

2.AbstractAdvisorAutoProxyCreator
BeanFactoryAware重写：
- AbstractAdvisorAutoProxyCreator.setBeanFactory -> initBeanFactory

3. AnnotationAwareAspectJAutoProxyCreator
BeanFactoryAware重写：
- AnnotationAwareAspectJAutoProxyCreator.initBeanFactory
```

在上面的任何方法搭上断点即可看到类似下面的方法调用栈：
```
AnnotationConfigApplicationContext.AnnotationConfigApplicationContext()
    ->AbstractApplicationContext.refresh() //刷新容器，给容器初始化bean
        ->AbstractApplicationContext.finishBeanFactoryInitialization()
            ->DefaultListableBeanFactory.preInstantiateSingletons()
                ->AbstractBeanFactory.getBean()
                    ->AbstractBeanFactory.doGetBean()
                        ->DefaultSingletonBeanRegistry.getSingleton()
                            ->AbstractBeanFactory.createBean()
                                ->AbstractAutowireCapableBeanFactory.resolveBeforeInstantiation()
                                    ->AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInstantiation()
                                        ->AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInstantiation()
                                            ->调用AOP相关的后置处理器
```

其中 `AbstractApplicationContext.refresh()` 方法，调用了 `registerBeanPostProcessors()`方法 ，它是用来注册后置处理器，以拦截 `bean` 的创建。也是在这个方法中完成了对 `AnnotationAwareAspectJAutoProxyCreator` 的注册。
在下面详细的展开。

注册完 `BeanPostProcessor` 后，还调用了方法 `finishBeanFactoryInitialization()` ，完成 `BeanFactory` 初始化工作，并创建剩下的单实例 `bean`。
```
@Override
public void refresh() throws BeansException, IllegalStateException {
    
    // .....

    // Register bean processors that intercept bean creation.
    registerBeanPostProcessors(beanFactory);

    // .....

    // Instantiate all remaining (non-lazy-init) singletons.
    finishBeanFactoryInitialization(beanFactory);

    // .....

}
```
#### registerBeanPostProcessors

`registerBeanPostProcessors`方法中注册了所有的`BeanPostProcessor`;注册顺序是：
1. 注册实现了`PriorityOrdered`接口的`BeanPostProcessor`;
2. 注册实现了 `Ordered` 接口的 `BeanPostProcessor`;
3. 注册常规的 `BeanPostProcessor` ,也就是没有实现优先级接口的 `BeanPostProcessor`;
4. 注册 `Spring` 内部 `BeanPostProcessor`;

由于`AnnotationAwareAspectJAutoProxyCreator`类间接实现了`Ordered`接口。所以它是在注册实现`Ordered`接口的`BeanPostProcessor`中完成注册。

注册时会调用`AbstractBeanFactory.getBean() -> AbstractBeanFactory.doGetBean()`创建`bean`。

`doGetBean()`方法作用：
- 创建`bean`：`createBeanInstance()`;
- 给`bean`中的属性赋值：`populateBean()`;
- 初始化`bean`：`initializeBean()`;

初始化`bean`时，`initializeBean`方法会调用`BeanPostProcessor`和`BeanFactory`以及`Aware`接口的相关方法。这也是`BeanPostProcessor`发挥初始化`bean`的原理。
```
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
    
    // ...

    invokeAwareMethods(beanName, bean);   //处理Aware接口的方法回调

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        // 执行后置处理器的postProcessBeforeInitialization方法
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }
    try {
        // 执行自定义的初始化方法，也就是在这执行 setBeanFactory方法
        invokeInitMethods(beanName, wrappedBean, mbd);  
    }

    // ...

    if (mbd == null || !mbd.isSynthetic()) {
        // 执行后置处理器的postProcessAfterInitialization方法
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    return wrappedBean;
}

// ...invokeAwareMethods方法简要 ...
private void invokeAwareMethods(String beanName, Object bean) {
    if (bean instanceof Aware) {
        if (bean instanceof BeanFactoryAware) {
            ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
        }
    }
}
```
`initializeBean`作用：
- 处理 `Aware` 接口的方法回调：`invokeAwareMethods()`;
- 执行后置处理器的`postProcessBeforeInitialization()`方法；
- 执行自定义的初始化方法：`invokeInitMethods()`;
- 执行后置处理器的`postProcessAfterInitialization()`方法;

`initializeBean`方法执行成功，`AnnotationAwareAspectJAutoProxyCreator`组件才会注册和初始化成功。

#### finishBeanFactoryInitialization
除了弄懂`AnnotationAwareAspectJAutoProxyCreator`组件何时注册，也需要知道它什么时候被调用，这就涉及到`finishBeanFactoryInitialization`方法。

继续看方法的调用：
```
AnnotationConfigApplicationContext.AnnotationConfigApplicationContext()
    ->AbstractApplicationContext.refresh() // 刷新容器，给容器初始化bean
        ->AbstractApplicationContext.finishBeanFactoryInitialization() // 从这继续
            ->DefaultListableBeanFactory.preInstantiateSingletons()
                ->AbstractBeanFactory.getBean()
                    ->AbstractBeanFactory.doGetBean()
                        ->DefaultSingletonBeanRegistry.getSingleton()
                            ->AbstractBeanFactory.createBean()
                                ->AbstractAutowireCapableBeanFactory.resolveBeforeInstantiation()
                                    ->AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInstantiation()
                                        ->AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInstantiation()
                                            ->调用AOP相关的后置处理器
```

`finishBeanFactoryInitialization`源码简要：
```
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {

    // ...
    
    // 注释大意： 实例化所有剩余的(非lazy-init)单例。
    // Instantiate all remaining (non-lazy-init) singletons.
    beanFactory.preInstantiateSingletons(); // 断点停在这里
}
```

`finishBeanFactoryInitialization` 方法也需要注册`Bean`。它会调用 `preInstantiateSingletons()` 方法遍历获取容器中所有的 `Bean`，实例化所有剩余的非懒加载初始化单例 `Bean`。

`preInstantiateSingletons()`方法源码简要：
```
	@Override
	public void preInstantiateSingletons() throws BeansException {

		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

		// Trigger initialization of all non-lazy singleton beans...
		for (String beanName : beanNames) {
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
            // 获取，非抽象、单例、非懒加载Bean
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
                // 是否 是FactoryBean类型
				if (isFactoryBean(beanName)) {
                    // ...
				}
				else {
					getBean(beanName); // 断点停在这
				}
			}
		}

        // ...
	}
```
`preInstantiateSingletons()` 调用 `getBean()` 方法，获取`Bean`实例，执行过程`getBean()->doGetBean()->getSingleton()->createBean()`，又回到了上面注册`Bean`的步骤。

这里要注意`createBean()`方法中的`resolveBeforeInstantiation()`方法，这里可以理解为缓存`Bean`,如果被创建了就拿来直接用，如果没有则创建`Bean`。
```
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
        throws BeanCreationException {

    // ...

    try {
        // 注释大意：给 BeanPostProcessors 一个返回代理而不是目标bean实例的机会。
        // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
        Object bean = resolveBeforeInstantiation(beanName, mbdToUse); // 断点停在这里
        if (bean != null) {
            return bean;
        }
    }

    // ...

    try {
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        if (logger.isTraceEnabled()) {
            logger.trace("Finished creating instance of bean '" + beanName + "'");
        }
        return beanInstance;
    }

    // ...
}
```

`resolveBeforeInstantiation()`、`applyBeanPostProcessorsBeforeInstantiation()`方法源码：
````
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        // Make sure bean class is actually resolved at this point.
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            Class<?> targetType = determineTargetType(beanName, mbd);
            if (targetType != null) {
                // 调用 applyBeanPostProcessorsBeforeInstantiation 方法
                bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName); // 断点停在这
                if (bean != null) {
                    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                }
            }
        }
        mbd.beforeInstantiationResolved = (bean != null);
    }
    return bean;
}

// ... 上面代码调用的方法 ...

protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
    // 遍历所有的 BeanPostProcessor
    for (BeanPostProcessor bp : getBeanPostProcessors()) {

        // //如果是 InstantiationAwareBeanPostProcessor 类型
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;

            // 调用 postProcessBeforeInstantiation 方法
            Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName); // 断点停在这
            if (result != null) {
                return result;
            }
        }
    }
    return null;
}
````
到了这里在回过头来看一下`AnnotationAwareAspectJAutoProxyCreator`组件实现的`SmartInstantiationAwareBeanPostProcessor`接口，继承关系：
```
SmartInstantiationAwareBeanPostProcessor 
    ->extends InstantiationAwareBeanPostProcessor
        ->extends BeanPostProcessor
```
到这就跟前边对上了，AOP相关的后置处理器也就是在这被调用的。

回头在看上面的`createBean()`方法，刚才看到的是`resolveBeforeInstantiation()`方法的调用栈，所以从层次结构上看`AnnotationAwareAspectJAutoProxyCreator`组件的调用
是在创建 `Bean`实例之前先尝试用后置处理器返回对象的。

![AOP@EnableAspectJAutoProxy原理](/myblog/posts/images/essays/AOP@EnableAspectJAutoProxy原理.png)



