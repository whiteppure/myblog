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
- 独立运行的 `Spring` 项目<br>
`SpringBoot` 可以以 jar 包的形式独立运行，运行一个 `SpringBoot` 项目只需通过 `java–jar xx.jar` 来运行。

- 内嵌 Servlet 容器<br>
`SpringBoot` 可选择内嵌 `Tomcat`、`Jetty` 或者 `Undertow`，这样我们无须以 `war` 包形式部署项目。

- 提供 `starter` 简化 `Maven` 配置<br>
`Spring` 提供了一系列的 `starter` pom 来简化 `Maven` 的依赖加载，例如，当你使用了`spring-boot-starter-web` 时，会自动加入依赖包。

- 自动配置 `Spring`<br>
`SpringBoot` 会根据在类路径中的 jar 包、类，为 jar 包里的类自动配置 Bean，这样会极大地减少我们要使用的配置。当然，`SpringBoot` 只是考虑了大多数的开发场景，并不是所有的场景，若在实际开发中我们需要自动配置 `Bean`，而 `SpringBoot` 没有提供支持，则可以自定义自动配置。

- 准生产的应用监控<br>
`SpringBoot` 提供基于 `http、ssh、telnet` 对运行时的项目进行监控。

- 无代码生成和 xml 配置<br>
`SpringBoot` 的神奇的不是借助于代码生成来实现的，而是通过条件注解来实现的，这是 `Spring 4.x` 提供的新特性。`Spring 4.x` 提倡使用 Java 配置和注解配置组合，而 `SpringBoot` 不需要任何 xml 配置即可实现 `Spring` 的所有配置。

## @SpringBootApplication
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

当Springboot启动的时候，会执行`AutoConfigurationImportSelector`这个类中的`getCandidateConfigurations`方法，这个方法会帮我们加载`META-INF/spring.factories`文件里面的当`@ConditionXXX`注解条件满足的类。

## 自动装配
`Spring`利用依赖注入（DI），完成对IOC容器中各个组件的依赖关系赋值。

Spring提供三种装配方式：
- 基于注解的自动装配
- 基于XML配置的显式装配
- 基于Java配置的显式装配

详细本篇博客，详细介绍基于注解的自动装配
| 自动装配   | 来源                          | 支持@Primary | springboot支持属性 |
| ---------- | ----------------------------- | ------------ | ------------------ |
| @Autowired | Springboot原生                | 支持         | boolean required   |
| @Resource  | JSR-250，JDK自带              | 不支持       | 无其他属性         |
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
使用`@Autowired`注解通常将其加载属性上或者构造器上，让其自动注入；
默认是按照类型去容器中寻找对应的组件，例如：
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
nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: ...
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
在Spring创建对象的时候会默认调用组件的无参构造方法，如果只有一个有参构造，如果想要创建对象，则必须调用该有参构造；
所以当一个组件只有一个有参构造时，则可以不用写`@Autowirea`注解。
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
- `Spring` 容器启动时，`AutowiredAnnotationBeanPostProcessor` 被注册到容器中；
- 扫描代码，如果带有 `@Autowired` 注解，则将依赖注入信息封装到 `InjectionMetadata` 中;
- 创建 `bean` 也就是实例化对象和初始化，会调用各种 `BeanPostProcessor` 对 `bean` 初始化，`AutowiredAnnotationBeanPostProcessor` 负责将相关的依赖注入进来；

`@Autowired`注解调用栈：
```
AbstractApplicationContext.refresh(容器初始化)
---> registerBeanPostProcessors (注册AutowiredAnnotationBeanPostProcessor) 
---> finishBeanFactoryInitialization 
---> AbstractAutowireCapableBeanFactory.doCreateBean
---> AbstractAutowireCapableBeanFactory.applyMergedBeanDefinitionPostProcessors
---> MergedBeanDefinitionPostProcessor.postProcessMergedBeanDefinition
---> AutowiredAnnotationBeanPostProcessor.findAutowiringMetadata
---> InjectionMetadata.checkConfigMembers
```

核心方法`findAutowiringMetadata`
```
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
`checkConfigMembers`方法
```
public void checkConfigMembers(RootBeanDefinition beanDefinition) {
		Set<InjectedElement> checkedElements = new LinkedHashSet<>(this.injectedElements.size());
		for (InjectedElement element : this.injectedElements) {
			Member member = element.getMember();
			if (!beanDefinition.isExternallyManagedConfigMember(member)) {
				beanDefinition.registerExternallyManagedConfigMember(member);
				checkedElements.add(element);
				if (logger.isTraceEnabled()) {
					logger.trace("Registered injected element on class [" + this.targetClass.getName() + "]: " + element);
				}
			}
		}
		this.checkedElements = checkedElements;
	}
```
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

使用`@Inject`注解需要导入
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

关于这些`Aware`都是使用`AwareProcessor`进行处理的；
比如:`ApplicationContextAwareProcessor`就是处理`ApplicationContextAware`接口的。

