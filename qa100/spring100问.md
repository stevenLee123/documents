# spring 100问
## 1. spring是什么
spring是一个java开发的生态体系，包含spring framework，springboot等一系列开发框架其主要目的是简化开发，实现代码的解耦

## 2. spring的优缺点
简化开发
解耦


## 3. spring的基本实现原理
使用简单工厂模式（beanFactory.getBean(...)） + 反射(实例化bean) 实现bean的创建和管理

## 4. IoC 和 DI 的区别
IoC: Inversion of Control ,控制反转，是代码设计的一种思想，是将控制对象创建的工作由程序员转移到框架中，实现代码的松耦合
DI： Dependency Injection,依赖注入，是spring 框架中对IoC的一种实现，在对象A依赖于对象B的关系中，框架实现将B注入到A中，而不用程序员自己进行set注入

## 5. 紧耦合和松耦合的区别
紧耦合： 尽可能的合理划分功能模块，功能模块之间耦合紧密
松耦合： 模块间的关系尽可能简单，功能快之间的耦合度低
在spring框架中通过依赖注入的方式降低了各个实力之间的耦合

## 6. 对BeanFactory的理解
BeanFactory是spring框架的顶层容器接口，主要功能是根据bean的定义实现对bean的创建，BeanFactory的扩展接口或类还实现了对bean的更加细粒度的控制，比如bean定义的修改，bean销毁等

## 7. BeanDefinition的作用是什么
BeanDefinition 规定了创建Bean的具体细节，比如bean是单例还是原型类型，是否懒加载、依赖的bean的列表，是否自动注入、bean初始化后调用方法、bean销毁后调用的方法

## 8. BeanFactory和ApplicationContext的区别是什么
BeanFactory提供基本的bean获取功能，而ApplicationContext是BeanFactory的一个子接口，主要是提供BeanFactory的一些拓展功能，比如提BeanFactoryPostProcessor或BeanDefinitionRegistryPostProcessor实现对BeanDefinition的手动添加和修改操作，提供事件发布与监听的功能，提供BeanPostProcessor实现对bean初始化前后的修改等
ApplicationContext也是是对BeanFactory组功能的组合

ApplicationContext 提供的扩展功能：
1. MessageResource：国际化支持
2. ResourceLoader:提供对资源文件的管理（各种配置的加载）
3. EnvironmentCapable:提供对系统环境变量及其他配置参数的管理
4. ApplicationEventPublisher: 提供对事件发布与监听的支持


## 9.BeanFactory和FacotoryBean的区别
BeanFactory是spring bean的容器（工厂），用来生产bean
FactoryBean是spring 中的接口接口，实现FactoryBean的类是spring中的一种特殊的bean，通过接口的getObject方法获取到的才是bean的真实实例

## 10. IoC的优缺点
优点：实现松耦合，提高程序的灵活性和可维护性
缺点：创建对象的步骤变得复杂，不直观，使用反射创建对象，效率上或有损耗，当使用XML配置Bean时，配置修改比较复杂

## 11. IoC的加载过程

IoC加载的过程可以从Bean的生命周期的角度来看

从编码到spring容器启动bean的完整周期可以分成以下四个阶段：
1. 概念阶段，即将bean的定义写入到XML中，或被@Configuration标注的java配置类中，或使用@Component注解在具体的bean上，定义Bean的细节，比如Bean的scope、bean的init-method、destroy-method等
2. 模型阶段，这一阶段spring容器将配置文件或配置类中的bean定义加载解析成BeanDefinition，放在BeanDefinitionMap中
3. 实例化阶段，这一阶段spring容器通过遍历baanDefinitionMap拿到bean的定义，根据定义的细节使用反射、cglib（或动态代理）创建bean的实例，这时bean的属性还没有进行依赖注入
4. 最终使用阶段，这一阶段spring容器通过依赖注入将bean的属性进行注入，这一阶段之后，通过beanFactory.getBean()获取到bean就是这个阶段产生的bean

## 12. spring的IoC扩展点及调用时机

## 13. Bean和对象的区别

## 14. spring bean的生命周期
1. 通过componentScan注解扫描包下的bean（被注解Component标注的类）
2. 拿到包下的所有的class文件
3. 使用反射根据条件推断需要使用的类构造器
4. 使用反射对标注了Autowired的属性进行依赖注入(通过beanPostProcessor实现)
5. 判断类的方法中是否有标注了PostConstruct注解，如果有，利用反射调用标注了PostConstruct的方法（初始化前的方法），或是判断类是否实现了BeanPostProcessor，如果实现了，则调用processor的postProcessBeforeInitialization方法（实际上PostConstruct也是使用BeanPostProcessor实现的）
6. 判断bean是否实现了InitializingBean接口，如果实现了，就调用afterPropertiesSet()方法进行初始化
7. 判断bean是否实现了BeanPostProcessor接口，有就调用postProcessAfterInitialization方法（初始化后）
8. 判断bean是否实现了各种aware接口，如果实现了，就调用相应的aware接口中的方法（一般是用于将容器中的资源直接给到bean方便bean的操作）
9. 判断Bean是否配置了AOP拦截，有则使用jdk代理或cglib实现代理
10. 将bean放入singletonBeanMap（单例池中）
11. 判断类的方法中是否有标注了PreDestroy注解的方法，有就利用反射调用标注了

## 15. spring 循环依赖中的三级缓存
1. 一级缓存： singletonObjects，存储已经完全初始化好，经历了完整的bean创建生命周期的bean，可以直接取出来使用
2. 二级缓存：earlySingletonObjects，存储刚刚实例化但还未进行属性注入的bean，或是需要bean需要aop时，进行提前aop后的aop代理对象
3. 三级缓存： singletonFactories，存储被ObjectFactory包装的刚刚进行实例化且还未进行属性注入的bean对象
解决循环依赖的核心思想：找到能打破循环依赖的点

## 16 spring 整合mybatis的原理（可能与springboot的starter的自动注入原理类似）
核心思想:将myabtis的配置类、sqlSession、mapper的代理类都放入到spring容器，有spring来管理
用到的知识： 
1. FactoryBean，通过FactoryBean对象来包裹mybatis的mapper接口，从而将myatbis的mapper代理对象放入spring中管理
2. 通过ClasspathBeanDeifintionScanner扫描@MapperScan指定的basePackage中的mapper接口，拿到mapper接口的beanDefinition定义
3. @Import: 在spring的一个配置类中引入其他配置类或其他组件的方式，在mybatis中，是通过ImportBeanDefinitionRegistrar引入由FactoryBean包装的mapper接口的beanDefinition

## 17 spring 的包扫描原理

1. 扫描的时机： 在BeanFactoryprocessor的调用时进行包扫描
2. 扫描得到的结果：扫描实际上拿到的是容器中bean的BeanDefinition
3. 扫描启动的方式：通过@ComponentScan中的basePackages拿到要秒扫的路径
4. 扫描中用到的核心类：ClasspathBeanDefinitionScanner
5. 扫描使用的技术：ASM，spring为什么没有通过反射的方式来进行class文件的扫描，因为使用反射来扫描是需要将包下的所有的class加载到虚拟机内存中，如果class文件过多则会造成启动过程中内存占用过高的问题。使用ASM技术可以解析每个class文件对象，得到class的元数据信息
6. 在spring扫描包的过程中，如果扫描到两个bean的名字相同（可能是由于包路径重复导致的重复扫描），当beandefinition中的classname不一致，则会报错，如果classname一致，可以判定为同一个包下的同一个bean被扫描了两次导致的问题，spring会忽略掉被再次扫描的那个beandefinition

## 18 spring中什么样的类才算配置类

一般我们在项目中在某个java类上加上@Configuration注解就代表这个类是个配置类，在spring里，spring的配置类是通过ConfigurationClassPostProcessor这个类进行加载的，ConfigurationClassPostProcessor调用ConfigurationClassUtils的checkConfigurationClassCandidate方法检查当前被检查类是否在类上有@Configuration、@ComponentScan、@Import、@ImportResource注解，或是当前类的方法上是否有@Bean注解，如果有则表示当前类是一个spring的java配置类

## 20 springboot 事件监听器发布顺序
1. ApplicationStartingEvent：运行开始时发送，在进行仁和处理之前（久安厅起和初始化程序注册除外）发送
2. 当已知要在上下文中使用环境但在创建上下文之前，发送ApplicatoinEventEnvironmentPreparedEvent
3. 准备ApplicationContext并调用ApplicationContextInitializers之后但在加载任何bean定义之前，发送ApplicationContextInitializedEvent
4. 在加载完bean定义之后，刷新上下文之前发送ApplicationPreparedEvent
5. 在刷行上下文之后，调用任何应用程序和命令行之前发送ApplicationStartedEvent
6. 发送带有LivessState.CORRECT的AvailablityChangeEvent，指示该应用程序被视为活跃状态
7. 调用任何应用程序和命令行运行程序之后，发送ApplicationReadyEvent
8. 紧随其后发送ReadabilityState.ACCEPTING_TRAFFIC的AvailabilityChangeEvent,指示应用程序已经准备就绪，可以处理请求
9. 如果启动时发生异常，则发送ApplicationFailedEvent


## 19 springboot 启动流程

springboot在spring 的启动流程基础上，增加了前期的准备工作（配置文件解析、环境加载、创建spring上下文ServletWebServerApplicationContext），主要利用事件监听器（观察者模式）进行功能扩展，实现对各个事件的监听，实现启动各个流程之间的解偶

具体流程（结合代码）：
初始化SpringApplication：
1. 在main方法中调用SpringApplication.run(TestApplication.class,args)方法
2. 创建SpringApplication，并执行run（args）方法
3. 将启动类放入primarySources
4. 根据classpth下的类推断出应该生成什么类型的applicationContext（webflux、servlet）
5. 使用spi机制通过SpringFactoryLoader获取key为org.springframework.context.ApplicationContextInitializer的所有值
6. 使用spi机制通过SpringFactoryLoader获取key为org.springframework.context.ApplicationListener的所有值（读取监听器）
7. 推断启动main方法的类（使用RuntimeException获取运行栈）
run方法的执行：
8. 使用StopWatch记录开始时间
9. 开启headless模式
10. 使用spi机制通过SpringFactoryLoader从spring.factories中获取SpringApplicationRunListener的组件，用来运行监听器
11. 根据命令行参数 实例化一个ApplicationArguments
12. 预初始化环境，读取环境变量、配置文件信息
13. 忽略实现了BeanInfo接口的bean
14. 打印banner信息
15. 创建spring 的上下文ApplicationContext（通过反射实现）
16. 预初始化上下文（环境变量Envoriment的设置）
17. 获取当前spring上下问的BeanFactory，解析main方法对应类（启动类）上的注解信息
18. refreshContext加载spring的IOC容器（spring的配置类都是通过invokeBeanFacotoryPostProcessors这个方法去执行的），这一步时spring 容器启动最关键的一步，加载所有的自动配置类，创建servlet容器
19. 记录springboot启动结束时间

## 20 实现自定义starter
1. 编写自动配置类
2. 使用spi机制编写spring.factories 文件，将自定义的自动配置类放在key EnableAutoConfiguration下
3. 关于工程结构：官方推荐建一个starter（辅助性的依赖管理）和一个autoconfigure（实现自动配置类及其他逻辑）
4. 关于starter的命名：以自己的名称+spring-boot-starter开头：test1-spring-boot-starter

## 21 springboot自动配置原理
1. 从启动类上的@SpringBootApplication作为起点，标志启动类是一个配置类
2. @Configuration：配置类，也是容器中的一个组件
3. @EnableAutoConfiguration： 开启自动配置功能，加载自动配置类
   原理：
   1. 使用@Import 导入了AutoConfigurationImportSelector实现对spring.factories文件中的EnableAutoConfiguration
   2. AutoConfigurationImportSelector 是DeferredImportSelector的子类，是一个延迟的importSelector，当没有实现DeferredImportSelector的getImportGroup时，会直接调用DeferredImportSelector子类的selectImport
   3. 如果实现了getImportGroup方法，则会调用getImportGroup返回的group的子类的selectImport
   4. 在调用group的selectImports方法之前，会先调用group的process方法，process方法里会真正的进入spi机制加载自动配置类的逻辑，使用SpringFactoriesLoader的loadFactoryNames拿到所有自动配置类的完整类名列表（实际上读取META_INFO/spring.factories文件是在创建SpringApplication时就已经进行了读取，这里拿到的列表是直接从cache中get到的键为EnableAutoConfiguration的完整类名的list）
4. @ComponentScan： 扫描包，相当于在spring.xml配置中的<context:component-scan>,扫描指定路径下的class
    扩展点：
   1. TypeExcludeFilter: springboot对外提供的扩展类，可供我们按照自定义的方式进行排除
   2. AutoConfigurationExcludeFilter：排除所有自动配置类且是自动配置类中的一个自动配置功能


## 22 自动配置类的底层原理
1. 使用@Configuration注解标志为配置类@Configuration的proxyBeanMethods属性：
   1. 当值为false时，不会使用cglib对@Configuration标记类进行动态代理增强，调用@bean的方式会走方法本地，产生的bean将不会是单例的
   2. 设为true时会创建代理对bean方法的调用则会走cglib的动态代理
2. 使用@Import 导入各类组件
3. 使用@ConfigurationProperties标志在配置文件中配置的属性，使用@EnableConfigurationProperties来启用配置属性
4. 使用各种@Conditional的变种注解实现对配置类或配置组件是否生效的控制




## 23 springboot 启动web容器（tomcat的原理）


## 24 spring 中@Import、@ImportSource的用法
1. @Import是导入其他配置类或是其他组件的工具: @Configuration标注的config类、ImportBeanDefinitionRegistrar类、ImportSelector的子类

## 25 spring的Bean的后置处理器

## 26 spring的BeanFactory的后置处理器

