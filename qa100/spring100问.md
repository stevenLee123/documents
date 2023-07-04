# spring 100问
## 1. spring是什么
spring是一个java开发的生态体系，包含spring framework，springboot等一系列开发框架其主要目的是简化开发，实现代码的解耦

## 2. spring的优缺点
简化开发
解耦

## BeanFactory和ApplicationContext
BeanFactory提供基本的bean获取功能，而ApplicationContext是BeanFactory的一个子接口，主要是提供BeanFactory的一些拓展功能，比如提BeanFactoryPostProcessor或BeanDefinitionRegistryPostProcessor实现对BeanDefinition的手动添加和修改操作，提供事件发布与监听的功能，提供BeanPostProcessor实现对bean初始化前后的修改等
ApplicationContext也是对BeanFactory功能的组合

ApplicationContext 提供的扩展功能：
1. MessageResource：国际化支持
2. ResourceLoader/ResourcePatternResolver:提供对资源文件的管理（各种配置的加载），提供通配符匹配资源的能力
3. EnvironmentCapable:提供对系统环境变量及其他配置参数的管理
4. ApplicationEventPublisher: 提供对事件发布与监听的支持

**MessageResource：国际化支持**
根据固定的key找到翻译后的结果
context.getMessage()
spring的国际化文件都放在messages.properties的文件中，
messages-zh.properties 内容
hi=hello
message-en.properites内容
hi=你好
调用方式：
```java
        System.out.println(applicationContext.getMessage("hi", null, Locale.CHINA));
        System.out.println(applicationContext.getMessage("hi", null, Locale.ENGLISH));
```

**ResourcePatternResolver资源读取**
```java
//读取配置文件
  Resource[] resources = applicationContext.getResources("classpath:application.properties");
        for (Resource resource : resources) {
            System.out.println(resource);
            
        }
        resources = applicationContext.getResources("classpath*:META-INF/spring.factories");
        for (Resource resource : resources) {
            System.out.println(resource);
        }

        
```
**EnvironmentCapable**
```java
 //取具体的变量配置信息
        System.out.println(applicationContext.getEnvironment().getProperty("java_home"));
        System.out.println(applicationContext.getEnvironment().getProperty("server.port"));       
```

**ApplicationEventPublisher**
用来发布事件
```java
//定义事件
public class UserRegisterEvent extends ApplicationEvent {
    public UserRegisterEvent(Object source) {
        super(source);
    }
}
//定义事件监听器，任意spring的bean都可以监听
@Component
public class Component2 {

    @EventListener
    public void eventListener(UserRegisterEvent event){
        System.out.println("收到用户注册事件");
        System.out.println(event);
    }
}

//发布事件
@Component
@Slf4j
public class Component1 {
    @Autowired
    private ApplicationEventPublisher context;
    //发布事件
    public void register(){
        log.info("用户注册");
        context.publishEvent(new UserRegisterEvent(this));
    }
}
applicationContext.getBean(Component1.class).register();
```

## 容器实现
**BeanFactory实现**
主要的实现 DefaultListableBeanFactory 
bean的定义：
> bean的class
> 使用范围scope
> 初始化
> 销毁

```java
public class DefaultListableBeanFactoryTest {
    public static void main(String[] args) {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        //DefaultListableBeanFactory 不会解析@Bean和@Configuration
        AbstractBeanDefinition beanDefinition =
                BeanDefinitionBuilder.genericBeanDefinition(Config.class)
                        .setScope("singleton")
                        .getBeanDefinition();
        beanFactory.registerBeanDefinition("config",beanDefinition);
        //添加beanfactory的后置处理器,会一次性添加五个bean
        AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
       //【1】
        for (String name :
                beanFactory.getBeanDefinitionNames()) {
            System.out.println(name);
        }
          //获取beanfactory后置处理器并调用
        Map<String, BeanFactoryPostProcessor> beansOfType = beanFactory.getBeansOfType(BeanFactoryPostProcessor.class);
        beansOfType.values().forEach(beanFactoryPostProcessor -> {
            beanFactoryPostProcessor.postProcessBeanFactory(beanFactory);
        });
        //【2】
        for (String name :
                beanFactory.getBeanDefinitionNames()) {
            System.out.println(name);
        }
           //【3】获取A中的B
       // System.out.println(beanFactory.getBean(A.class).getB());
         //bean的后置处理器，针对bean的生命周期各个阶段提供扩展，如@Autowired，@Resource
        //注册bean的后置处理器
        beanFactory.getBeansOfType(BeanPostProcessor.class)
                .values().forEach(beanFactory::addBeanPostProcessor);

        //【4】获取A中的B
        System.out.println(beanFactory.getBean(A.class).getB());
    }

    @Configuration
    static class Config{

        @Bean
        public A a(){
            return new A();
        }
        @Bean
        public B b(){
            return new B();
        }
    }
}
class A {
    public A() {
        System.out.println("构造 a");
    }

    @Autowired
    private B b;

    public B getB() {
        return b;
    }

    public void setB(B b) {
        this.b = b;
    }
}

class B {
    public B() {
        System.out.println("构造 b");
    }
}

```

【1】处打印结果：
config
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor  --解析@Autowired注解
org.springframework.context.annotation.internalCommonAnnotationProcessor   --解析@Resource注解
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
 
 @Autowired先按type后按name进行注入
 @Resource 先按name再按type进行注入
* DefaultListableBeanFactory不具备解析@Configuration 、@Bean等注解的能力，不具备自动解析配置文件中的占位符#{}、${}等功能提供通过手动的方式注入bean的定义

【2】处打印结果：
config
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
a
b
* 通过BeanfactoryPostProcessor实现容器的功能拓展
【3】处打印结果：
null
* 目前的条件下@Autowired功能失效
* 要注入B需要靠BeanPostProcessor（bean的生命周期内的后置处理器）
【4】处打印结果
构造 a
23:28:17.243 [main] DEBUG org.springframework.beans.factory.support.DefaultListableBeanFactory - Creating shared instance of singleton bean 'b'
构造 b
com.dxy.data.springtest.beanFactory.B@783a467b

* bean默认是延迟实例化（lazy）
* 可以使用以下方法进行提前实例化：

```java
  //提前实例化单例
        beanFactory.preInstantiateSingletons();
```

**ApplicationContext实现**
相比BeanFactory可以实现从文件或其他方式加载beanDefinition
相关类
* `ClasspathXmlApplicationContext` --从类路径下加载bean的配置文件
* `FileSystemXmlApplicationContext` --从文件系统路径下加载bean的配置文件
* `XmlBeanDefinitionReader` --从xml文件中读取bean的定义信息放入到BeanDefinition，最后作为属性传给beanFactory
* `AnnotationConfigApplicationContext` --从配置类@Configuration中加载beanDefinition
* `AnnotataionConfigServletWebServerApplicationContext` --在AnnotationConfigApplicationContext扩展web servlet容器
```java
package com.dxy.data.springtest.applicationcontext;

import org.springframework.boot.autoconfigure.web.servlet.DispatcherServletRegistrationBean;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext;
import org.springframework.boot.web.servlet.server.ServletWebServerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.DispatcherServlet;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * @Description:
 * @CreateDate: Created in 2023/5/17 09:04
 * @Author: lijie3
 */
public class ApplicationContextTest {

    public static void main(String[] args) {
        testServletWebServer();
    }

    private static void testServletWebServer(){
        AnnotationConfigServletWebServerApplicationContext context =
                new AnnotationConfigServletWebServerApplicationContext(WebConfig.class);

    }
    /**
     * 设置web的三个组件
     */
    @Configuration
    static class WebConfig{
        //设置tomcat的工厂
        @Bean
        public ServletWebServerFactory servletWebServerFactory(){
            return new TomcatServletWebServerFactory();
        }

        //设置前端转发器
        @Bean
        public DispatcherServlet dispatcherServlet(){
            return new DispatcherServlet();
        }
        //注册DispatcherServlet
        @Bean
        public DispatcherServletRegistrationBean registrationBean(DispatcherServlet dispatcherServlet){
            return  new DispatcherServletRegistrationBean(dispatcherServlet,"/");
        }
        //controller应用
        @Bean("/hello")
        public Controller controller(){
            return (request, response) -> {
                response.getWriter().println("hello");
                return null;
            };
        }
    }
}
```

## bean的生命周期

```java
/**
 * @Description:
 * @CreateDate: Created in 2023/5/17 09:17
 * @Author: lijie3
 */
@Component
@Slf4j
public class LifeCycleBean {
    public LifeCycleBean() {
        log.info("------construct.....");
    }


    @Autowired
    public void setJavaHome(@Value("${JAVA_HOME}") String javaHome){
        log.info("-----java_home:{}",javaHome);
    }

    @PostConstruct
    public void init(){
        log.info("-------init------");
    }

    @PreDestroy
    public void destroy(){
        log.info("-------destroy------");
    }

}
```

打印结果
2023-05-17 09:23:46.893  INFO 30130 --- [           main] c.d.d.s.Component.LifeCycleBean          : ------construct.....
2023-05-17 09:23:46.901  INFO 30130 --- [           main] c.d.d.s.Component.LifeCycleBean          : -----java_home:/Library/Java/JavaVirtualMachines/jdk1.8.0_291.jdk/Contents/Home
2023-05-17 09:23:46.902  INFO 30130 --- [           main] c.d.d.s.Component.LifeCycleBean          : -------init------
-------destroy------
## bean的后置处理器 -- BeaPostProcessor
DestructionAwareBeanPostProcessor 销毁相关的后置处理

```java

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeansException;
import org.springframework.beans.PropertyValues;
import org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor;
import org.springframework.stereotype.Component;

/**
 * @Description:
 * @CreateDate: Created in 2023/5/21 22:10
 * @Author: lijie3
 */
@Slf4j
@Component
public class MyBeanPostProcessor  implements InstantiationAwareBeanPostProcessor, DestructionAwareBeanPostProcessor {
    @Override
    public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.info("==========销毁之前执行，如@PreDestroy");
        }

    }

    @Override
    public boolean requiresDestruction(Object bean) {
        return DestructionAwareBeanPostProcessor.super.requiresDestruction(bean);
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.info("==========实例化之后执行，这里返回的对象如果不为null会替换掉原来的bean");
        }
        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {

        if(beanName.equals("lifeCycleBean")){
            log.info("==========实例化之后执行，返回true 继续执行依赖注入，返回false会跳过依赖注入");
        }
        return true;
    }


    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.info("==========依赖注入阶段执行，如@Autowired、@Value、@Resource");
        }
        return pvs;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.info("==========初始化之前执行，这里返回的对象会替换掉原本的bean，如@PostConstruct、@ConfigurationProperties");
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if(beanName.equals("lifeCycleBean")){
            log.info("==========初始化之前执行，这里返回的对象会替换掉原本的bean，如代理增强");
        }
        return bean;
    }
}
```

打印结果
```
[main] c.d.d.s.Component.MyBeanPostProcessor    : ==========实例化之后执行，这里返回的对象如果不为null会替换掉原来的bean
2023-05-21 22:46:55.026  INFO 69429 --- [           main] c.d.d.s.Component.LifeCycleBean          : ------construct.....
2023-05-21 22:46:55.033  INFO 69429 --- [           main] c.d.d.s.Component.MyBeanPostProcessor    : ==========实例化之后执行，返回true 继续执行依赖注入，返回false会跳过依赖注入
2023-05-21 22:46:55.033  INFO 69429 --- [           main] c.d.d.s.Component.MyBeanPostProcessor    : ==========依赖注入阶段执行，如@Autowired、@Value、@Resource
2023-05-21 22:46:55.037  INFO 69429 --- [           main] c.d.d.s.Component.LifeCycleBean          : -----java_home:/Library/Java/JavaVirtualMachines/jdk1.8.0_291.jdk/Contents/Home
2023-05-21 22:46:55.038  INFO 69429 --- [           main] c.d.d.s.Component.MyBeanPostProcessor    : ==========初始化之前执行，这里返回的对象会替换掉原本的bean，如@PostConstruct、@ConfigurationProperties
2023-05-21 22:46:55.038  INFO 69429 --- [           main] c.d.d.s.Component.LifeCycleBean          : -------init------
2023-05-21 22:46:55.038  INFO 69429 --- [           main] c.d.d.s.Component.MyBeanPostProcessor    : ==========初始化之前执行，这里返回的对象会替换掉原本的bean，如代理增强
ImportTest{name='this is a test'}
构造 a
构造 b
2023-05-21 22:46:55.577  INFO 69429 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 1 endpoint(s) beneath base path '/actuator'
2023-05-21 22:46:55.628  INFO 69429 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8888 (http) with context path ''
2023-05-21 22:46:55.646  INFO 69429 --- [           main] c.d.d.s.SpringtestApplication02          : Started SpringtestApplication02 in 3.07 seconds (JVM running for 3.794)
2023-05-21 22:46:55.690  INFO 69429 --- [           main] c.d.d.s.Component.MyBeanPostProcessor    : ==========销毁之前执行，如@PreDestroy
2023-05-21 22:46:55.690  INFO 69429 --- [           main] c.d.d.s.Component.LifeCycleBean          : -------destroy------
Disconnected from the target VM, address: '127.0.0.1:64535', transport: 'socket'
```
### 模板方法模式 --在bean生命周期阶段使用的设计模式
固定不变的步骤采用具体的方法实现，对与具体的步骤进行抽象，由子类来实现
BeanPostProcessor 调用的原理
```java
public class MyBeanFactory {

    private  List<BeanPostProcessor> processorList = new ArrayList<>();

    public  Object getBean(){
        Object bean = new Object();
        System.out.println("构造bean：" + bean);
        System.out.println("依赖注入 ：" + bean);
        //不同的beanPostProcessor调用的时机不一样
        for (BeanPostProcessor processor :
                processorList) {
            processor.postProcessAfterInitialization(bean,bean.toString());

        }
        System.out.println("初始化：" + bean);
        return bean;
    }
}
```

### 解析@ConfigurationProperties的bean后置处理器ConfigurationPropertiesBindingPostProcessor
```java
//必须要有get和set方法才能绑定成功
@Data
@ConfigurationProperties(prefix = "java")
public class JavaInfo {

    private String home;

    private String version;

    @Override
    public String toString() {
        return "JavaInfo{" +
                "home='" + home + '\'' +
                ", version='" + version + '\'' +
                '}';
    }
}

    GenericApplicationContext context = new GenericApplicationContext();

        context.registerBean("javaInfo", JavaInfo.class);
        context.getDefaultListableBeanFactory().setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
        context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
        context.registerBean(CommonAnnotationBeanPostProcessor.class);
        ConfigurationPropertiesBindingPostProcessor.register(context.getDefaultListableBeanFactory());
        context.refresh();
        final JavaInfo bean = context.getBean(JavaInfo.class);
        System.out.println(bean);

        //容器销毁
       context.close();
```

### @Autowired的bean后置处理器AutowiredAnnotationBeanPostProcessor

```java
public class Bean1 {
//    @Autowired
    private Bean2 bean2;
    @Autowired
    public void setBean2(Bean2 bean2) {
        this.bean2 = bean2;
    }

    @Autowired
    private Bean3 bean3;

//    @Value("${JAVA_HOME}")
    private String javaHome;

    @Override
    public String toString() {
        return "Bean1{" +
                "bean2=" + bean2 +
                ", bean3=" + bean3 +
                ", javaHome='" + javaHome + '\'' +
                '}';
    }

    @Autowired
    public void setJavaHome(@Value("${JAVA_HOME}")String javaHome) {
        this.javaHome = javaHome;
    }
}
//两种方式处理@Autowired注入方式
    public static void main(String[] args) throws Throwable {
        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        //通过这种方式注册bean不会走依赖注入、初始化过程
        beanFactory.registerSingleton("bean2", new Bean2());
        beanFactory.registerSingleton("bean3", new Bean3());
        //处理@Value注解
        beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());

        AutowiredAnnotationBeanPostProcessor processor = new AutowiredAnnotationBeanPostProcessor();
        processor.setBeanFactory(beanFactory);
        //解析value中的${}占位符
        beanFactory.addEmbeddedValueResolver(new StandardEnvironment()::resolvePlaceholders);

        Bean1 bean1 = new Bean1();

        System.out.println(bean1);
        //方式1：直接执行执行依赖注入,处理Autowired
//        processor.postProcessProperties(null,bean1,"bean1");
        //获取bean1上加了@Value、@Autowired注解的成员变量信息
        final Method findAutowiringMetadata = AutowiredAnnotationBeanPostProcessor.class.getDeclaredMethod("findAutowiringMetadata", String.class, Class.class, PropertyValues.class);
        findAutowiringMetadata.setAccessible(true);
        final InjectionMetadata metadata = (InjectionMetadata)findAutowiringMetadata.invoke(processor, "bean1", Bean1.class, null);
        System.out.println(metadata);
        //方式2：通过反射的方式进行注入
        metadata.inject(bean1,"bean1",null);

        System.out.println(bean1);
    // -------------原理展示-------------------
       //按类型查找属性注入演示
        Field bean3 = Bean1.class.getDeclaredField("bean3");
        DependencyDescriptor dependencyDescriptor = new DependencyDescriptor(bean3, false);
        //根据类型找到符合类型的属性bean
        final Object o = beanFactory.doResolveDependency(dependencyDescriptor, null, null, null);
        //找到容器中的bean3对象
        System.out.println(o);

        //按类型查找方法注入演示
        final Method setBean2 = Bean1.class.getDeclaredMethod("setBean2", Bean2.class);
        final DependencyDescriptor dependencyDescriptor1 = new DependencyDescriptor(new MethodParameter(setBean2, 0),false);
        final Object o1 = beanFactory.doResolveDependency(dependencyDescriptor1, null, null, null);
        //找到bean2对象
        System.out.println(o1);
           //值注入的方式
        final Method setJavaHome = Bean1.class.getDeclaredMethod("setJavaHome", String.class);
        final DependencyDescriptor dependencyDescriptor2 = new DependencyDescriptor(new MethodParameter(setJavaHome, 0), true);
        final Object o2 = beanFactory.doResolveDependency(dependencyDescriptor2, null, null, null);
        System.out.println(o2);
    }
```

## BeanFactory的后置处理器
* ConfigurationClassPostProcessor--处理@Configuration、@Import、@Bean、@ImportSource、@ComponentScan等和容器相关的注解
* mybatis提供的MapperScannerConfigurer 对mybatis的mapper接口进行扫描（可以被@MapperScaner代理）
```java
@Configuration
@ComponentScan("com.dxy.data.springtest.config")
public class Config1 {
    //实现对@Bean的解析
    @Bean
    public Bean1 bean1(){
        return new Bean1();
    }
}
    public static void main(String[] args) throws IOException {
        GenericApplicationContext context = new GenericApplicationContext();
        //在不进行其他容器后置处理器的情况下，@Configuration注解无法被解析到
        context.registerBean("config1", Config1.class);
        //使用BeanFactory的后置处理器
         context.registerBean(ConfigurationClassPostProcessor.class);
        context.refresh();

        for (String name :
                context.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        //容器销毁
       context.close();
    }
```

### ConfigurationClassPostProcessor实现原理 
手写模拟实现：
```java
public class ComponentScanPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {

        try {
            //扫描包下的二进制class文件
            final ComponentScan annotation = AnnotationUtils.findAnnotation(Config1.class, ComponentScan.class);
            if (annotation != null) {
                for (String s : annotation.basePackages()) {
                    //取到配置的包名
                    System.out.println(s);
                    CachingMetadataReaderFactory factory = new CachingMetadataReaderFactory();
                    //将包名转换为路径名
                    String path = "classpath*:" + s.replace(".", "/") + "/**/*.class";
                    System.out.println(path);
                    final Resource[] resources = new PathMatchingResourcePatternResolver().getResources(path);
                    final AnnotationBeanNameGenerator annotationBeanNameGenerator = new AnnotationBeanNameGenerator();
                    for (Resource resource : resources) {
                        //拿到类的元信息
                        final MetadataReader metadataReader = factory.getMetadataReader(resource);
                        System.out.println(metadataReader.getClassMetadata().getClassName());
                        //是否加了@Component注解
                        System.out.println("是否加了Component生注解：" + metadataReader.getAnnotationMetadata().hasAnnotation(Component.class.getName()));
                        System.out.println("是否加了Component派生注解：" + metadataReader.getAnnotationMetadata().hasMetaAnnotation(Component.class.getName()));
                        //注册beandefition
                        if (metadataReader.getAnnotationMetadata().hasAnnotation(Component.class.getName()) ||
                                metadataReader.getAnnotationMetadata().hasMetaAnnotation(Component.class.getName())) {
                            final AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(metadataReader.getClassMetadata().getClassName()).getBeanDefinition();
                            if(configurableListableBeanFactory instanceof DefaultListableBeanFactory){
                                DefaultListableBeanFactory beanFactory = (DefaultListableBeanFactory) configurableListableBeanFactory;
                                //生成bean的名字
                                final String beanName = annotationBeanNameGenerator.generateBeanName(beanDefinition, beanFactory);
                                //将beandefition注入到beanfactory
                                beanFactory.registerBeanDefinition(beanName, beanDefinition);
                            }
                        }
                    }
                }
            }
        } catch (Exception e) {

        }

    }
}
//启动类中进行注册
public static void main(String[] args) throws IOException {
        GenericApplicationContext context = new GenericApplicationContext();
        //在不进行其他容器后置处理器的情况下，@Configuration注解无法被解析到
        context.registerBean("config1", Config1.class);
        //注册beanFactory的后置处理器
        context.registerBean(ComponentScanPostProcessor.class);
        context.refresh();
        System.out.println("---------------------------");

        for (String name :
                context.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        //容器销毁
       context.close();
    }
```
### 关于对@Bean注解的解析原理（可以将下面的代码封装到BeanFactoryPostProcessor中实现对@Bean注解的解析）

```java
 GenericApplicationContext context = new GenericApplicationContext();
        //在不进行其他容器后置处理器的情况下，@Configuration注解无法被解析到
        context.registerBean("config1", Config1.class);
        CachingMetadataReaderFactory factory = new CachingMetadataReaderFactory();
        MetadataReader reader = factory.getMetadataReader(new ClassPathResource("com/dxy/data/springtest/config/Config1.class"));
        final Set<MethodMetadata> annotatedMethods = reader.getAnnotationMetadata().getAnnotatedMethods(Bean.class.getName());
        //实现对@Bean注解方法的解析
        for (MethodMetadata annotatedMethod : annotatedMethods) {
            System.out.println(annotatedMethod);

            final BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition();
            beanDefinitionBuilder.setFactoryMethodOnBean(annotatedMethod.getMethodName(),"config1");
            final AbstractBeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();
            context.getDefaultListableBeanFactory().registerBeanDefinition(annotatedMethod.getMethodName(),beanDefinition);
        }
        //注册beanFactory的后置处理器
        context.refresh();
        System.out.println("---------------------------");

        for (String name :
                context.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        //容器销毁
       context.close();
```
### mybatis-spring中的MapperFactoryBean
* 扫描包下的class文件
* 将class文件生成BeanDefinition
* 将BeanDefinition注册到BeanFactory中
通过MapperFactoryBean实现对mapper接口的代理


## Aware接口
Aware实现注入一些与容器相关的信息
* BeanNameAware接口注入bean的名字
* BeanFactoryAware接口注入BeanFactory容器
* ApplicationContextAware接口注入ApplicationContext容器
* EmbeddedValueResolverAware接口注入占位符${}对应的对应的值




## InitializingBean接口
实现bean的初始化功能

**aware接口的方法先执行，InitializingBean接口的方法后执行**
**Aware、InitializingBean属于spring的内置功能，而@PostConstruct、@Autowired等注解都属于spring的扩展功能，需要使用spring的bean的后置处理器，扩展功能可能在某些情况下（后置处理器缺失）可能会失效**

## @Autowired失效分析
失效情况演示：
```java
@Configuration
@Slf4j
public class Config2 {

    @Autowired
    public void setApplicationContext(ApplicationContext context){
        log.info("注入applicationContext");
    }

    @PostConstruct
    public void init(){
        log.info("初始化");
    }
    //添加beanFactory的后置处理器,会导致@Autowired、@PostConstruct失效
    @Bean
    public BeanFactoryPostProcessor processor(){
        return beanFactory -> {
            log.info("执行BeanFactoryPostProcessor");
        };
    }
}

 public static void main(String[] args) throws IOException {
        GenericApplicationContext context = new GenericApplicationContext();
        //在不进行其他容器后置处理器的情况下，@Configuration注解无法被解析到
        context.registerBean("config2", Config2.class);
        //注册beanFactory的后置处理器
        context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
        context.registerBean(CommonAnnotationBeanPostProcessor.class);
        context.registerBean(ConfigurationClassPostProcessor.class);
        context.refresh();
        System.out.println("---------------------------");

        for (String name :
                context.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        //容器销毁
        context.close();
    }
```
原因分析：
context.refresh() 的执行过程：
- 首先从beanfctory中获取BeanFactory的后置处理器，
- 然后添加bean的后置处理器，
- 然后初始化单例bean，

当java配置类中包含了BeanFactoryPostProcessor时，要先创建BeanFactoryPostProcessor的前提是先创建java泪痣类，而此时BeanPostProcessor还未准备好，导致配置类中的@Autowired，@PostConstruct等注解失效
解决方案是通过spring的内置功能代替扩展功能，实现Aware、InitializingBean接口来注入属性

## 初始化与销毁
```java
@Slf4j
public class Bean5 implements InitializingBean {
    @PostConstruct
    public void init(){
        log.info("初始化postConstruct");
    }
    @Override
    public void afterPropertiesSet() throws Exception {
        log.info("初始化afterPropertiesSet");
    }
    public void init2(){
        log.info("@bean的init注解的初始化");
    }
    @PreDestroy
    public void destroy1(){
        log.info("销毁@PreDestroy");
    }

    @Override
    public void destroy() throws Exception {
        log.info("销毁DisposableBean");
    }
    public void destroy2(){
        log.info("销毁通过@Bean");
    }
}
    public static void main(String[] args) throws IOException {
        GenericApplicationContext context = new GenericApplicationContext();
        //在不进行其他容器后置处理器的情况下，@Configuration注解无法被解析到
        context.registerBean("config3", Config3.class);
        //注册beanFactory的后置处理器
        context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
        context.registerBean(CommonAnnotationBeanPostProcessor.class);
        context.registerBean(ConfigurationClassPostProcessor.class);
        System.out.println("---------------------------");
        context.refresh();
        System.out.println("---------------------------");
        //容器销毁
        context.close();
    }
    //执行结果：
    //22:44:23.447 [main] INFO com.dxy.data.springtest.config.Bean5 - 初始化postConstruct -- 通过BeanPostProcessor执行
//22:44:23.447 [main] INFO com.dxy.data.springtest.config.Bean5 - 初始化afterPropertiesSet --通过 实现InitializingBean实现
//22:44:23.449 [main] INFO com.dxy.data.springtest.config.Bean5 - @bean的init注解的初始化 --通过BeanDefinition配置实现
//22:48:05.667 [main] INFO com.dxy.data.springtest.config.Bean5 - 销毁@PreDestroy
//22:48:05.668 [main] INFO com.dxy.data.springtest.config.Bean5 - 销毁DisposableBean
//22:48:05.668 [main] INFO com.dxy.data.springtest.config.Bean5 - 销毁通过@Bean
```

## Scope
scope中的类型：
* singleton 单例，获取容器中的bean返回同一个对象,单例使用其他域的类必须使用@Lazy
* prototype 原型，获取容器中的bean返回一个新的对象
* request request域中有效
* session session域中有效
* application  ServletContext应用程序域有效

### singleton使用需要注意的地方
- 单例A中注入多例B时，由于只会注入一次，每次通过A获取B时，获取到的都是同一个B，而不是每次创建新的B
解决思想：推迟其他scope bean的获取
解决方案1：使用@Lazy标记A的B属性，生成代理实现每次都通过代理创建新的B的实例
解决方法2: 在B的@Scope中添加属性，proxyMode= ScopedProxyMode.TARGET_CLASS,也是通过代理实现
解决方案3: 使用ObjectFactory<B> 来包装属性B，通过ObjectFactory.getObject()来实现生成新的B实例
解决方案4: 通过注入ApplicationContext,然后通过context.getBean()方法每次都拿到新的B实例

## AOP的三种实现
aop的实现不仅有代理，而且还使用了（aspectj）ajc来修改class实现增强（需要使用maven的编译插件，改方法使用的比较少），还可以使用jdk类加载的agent来实现（jdk16之前支持）

## AOP的proxy
两种代理方式：
jdk代理：只针对接口代理
```java
public class JdkProxyDemo {

    interface Foo{
        void foo();
    }
    //目标对象允许是final类型
    static class Target implements Foo{
        @Override
        public void foo() {
            System.out.println("target foo");
        }
    }

    public static void main(String[] args) {
        Target target = new Target();
        //用来加载在运行期间动态生成的字节码
        ClassLoader loader = JdkProxyDemo.class.getClassLoader();
        Foo proxy = (Foo) Proxy.newProxyInstance(loader, new Class[]{Foo.class}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("before ....");
                //通过反射执行目标方法
                method.invoke(target,args);
                System.out.println("after.....");
                return null;
            }
        });
        proxy.foo();
    }
}
```
cglib代理，可以对类进行代理，**被代理类和被代理的方法不能是final类型**
```java
public class CglibProxyDemo {

    static class Target{
        public void foo(){
            System.out.println("target foo");
        }
    }

    public static void main(String[] args) {
        Target target = new Target();
        //cglib代理
        final Target proxy = (Target) Enhancer.create(Target.class, new MethodInterceptor() {
            @Override
            public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                System.out.println("before ....");
                                //使用反射调用目标
//                final Object result = method.invoke(target, args);
                //methodProxy可以避免反射的调用,内部不是用的反射，spring用的是这种方式
                final Object result = methodProxy.invoke(target, args);
                                //调用proxy的父类方法也可以实现方法调用
                //final Object result = methodProxy.invokeSuper(proxy, args);
                System.out.println("after.....");
                return result;
            }
        });
        proxy.foo();
    }
}
```

### jdk动态代理原理
* jdk动态代理使用了asm来动态生成代理类
* 一个目标方法对应一个代理类
* 手写实现动态代理：
```java
package com.dxy.data.springtest.aop;

import java.lang.reflect.Method;
import java.lang.reflect.UndeclaredThrowableException;

/**
 * @Description: jdk代理模拟实现
 * @CreateDate: Created in 2023/5/23 09:09
 * @Author: lijie3
 */
public class $Proxy0 implements JdkProxyDemo02.Foo {

    private JdkProxyDemo02.InvocationHandler h;

    public $Proxy0(JdkProxyDemo02.InvocationHandler h) {
        this.h = h;
    }

    @Override
    public void foo() {
        //对于不确定代码的实现应该提供抽象方法
        try {
            h.invoke(this,foo,new Object[0]);
        }catch (Throwable e){
            e.printStackTrace();
        }
    }

    @Override
    public int bar() {
         int result = 0;
        try{
            result = (int) h.invoke(this,bar, new Object[0]);
            return result;
        }catch (RuntimeException |Error e){
            throw e;
            //检查异常转化为非检查异常后抛出
        }catch (Throwable e){
            throw new UndeclaredThrowableException(e);
        }
    }

    static Method foo;

    static Method bar;

   static {
       try {
           foo = JdkProxyDemo02.Foo.class.getDeclaredMethod("foo");
           bar = JdkProxyDemo02.Foo.class.getDeclaredMethod("bar");
       } catch (NoSuchMethodException e) {
           throw new NoSuchMethodError(e.getMessage());
       }
   }
}

package com.dxy.data.springtest.aop;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * @Description: 实际使用
 * @CreateDate: Created in 2023/5/23 08:26
 * @Author: lijie3
 */
public class JdkProxyDemo02 {

    interface Foo {
        void foo();

        int bar();
    }

    //目标对象允许是final类型
    static class Target implements Foo {
        @Override
        public void foo() {
            System.out.println("target foo");
        }

        @Override
        public int bar() {
            System.out.println("target bar");
            return 1;
        }
    }

    interface InvocationHandler {
        Object invoke(Object proxy,Method method, Object[] args) throws Throwable;
    }

    public static void main(String[] args) {
        Foo proxy0 = new $Proxy0(new InvocationHandler() {
            @Override
            public Object invoke(Object proxy,Method method, Object[] args) throws InvocationTargetException, IllegalAccessException {
                //实现功能增强
                System.out.println("before....");
                //调用目标
                Object result = method.invoke(new Target(), args);
                System.out.println("after......");
                return result;
            }
        });
        proxy0.foo();
        proxy0.bar();
    }
}
```
* jdk动态代理生成字节码
  * 使用了ASM来生成字节码
  * 动态代理时通过反射来执行被代理的方法，而jdk会通过编译优化实现了对反射的优化，多次调用时将会将反射调用优化成正常的方法调用

* AOP表达式
| 切入点指示符 | 含义                                                       |
| ------------ | ---------------------------------------------------------- |
| execution    | 匹配执行方法的连接点                                       |
| within       | 匹配指定类型内的执行方法                                   |
| this         | 匹配当前AOP代理对象类型的执行方法(可能包括引入接口)        |
| target       | 匹配当前目标对象类型的执行方法(不包括引入接口)             |
| args         | 匹配当前执行的方法传入的参数为指定类型的执行方法           |
| @target      | 匹配当前目标对象类型的执行方法，其中目标对象持有指定的注解 |
| @within      | 匹配所有持有指定注解类型内的方法                           |
| @args        | 匹配当前执行的方法传入的参数持有指定注解的执行             |
| @annotation  | 匹配当前执行方法持有指定注解的方法                         |

### cglib代理类的内部实现
cglib 底层可以通过两个代理类避免反射的调用，直接调用被代理对象的方法，提高效率
一个cglib的代理类对应两个fastclass，可以匹配到多个方法
代码模拟实现：
```java
//被代理对象
public class Target {

    public void save(){
        System.out.println("target save");
    }

    public void save(int i){
        System.out.println("target save:"+ i);
    }
}
//代理类
public class Proxy extends Target{

    private MethodInterceptor methodInterceptor;

    static Method save0;

    static Method save1;

    static MethodProxy save0Proxy;

    static MethodProxy save1Proxy;

    static{
        try {
            save0 = Target.class.getDeclaredMethod("save");
            save1 = Target.class.getDeclaredMethod("save", int.class);
            //()V 无参方法
            save0Proxy = MethodProxy.create(Target.class,Proxy.class,"()V","save","saveSuper");
            //(I)V 带一个整型参数的方法
            save1Proxy = MethodProxy.create(Target.class,Proxy.class,"(I)V","save","saveSuper");
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
    }

    public void setMethodInterceptor(MethodInterceptor methodInterceptor) {
        this.methodInterceptor = methodInterceptor;
    }

    /**
     * 带原始功能的save
     */
    public void saveSuper(){
        super.save();
    }

    public void saveSuper(int i){
        super.save(i);
    }

    @Override
    public void save() {
        try {
            methodInterceptor.intercept(this,save0,new Object[0],save0Proxy);
        } catch (Throwable e) {
            e.printStackTrace();
        }
    }

    @Override
    public void save(int i) {
        try {
            methodInterceptor.intercept(this,save1,new Object[]{i},save1Proxy);
        } catch (Throwable e) {
            e.printStackTrace();
        }
    }
}
//使用
public class CglibDemo {

    public static void main(String[] args) {
        Proxy proxy = new Proxy();
        Target target = new Target();
        proxy.setMethodInterceptor(new MethodInterceptor() {
            @Override
            public Object intercept(Object p, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                System.out.println("before");
//                return   method.invoke(target, objects);//反射调用
                return   methodProxy.invoke(target, objects);//内部无反射，结合目标调用
//                return   methodProxy.invokeSuper(p, objects);//内部无反射，结合代理用
            }
        });
        proxy.save();
        proxy.save(10);
    }
}
```
### MethodProxy是如何避免反射调用的


## spring对jdk和cglib代理的统一
概念： 
  * aspect --切面类
  * pointcut --切点
  * advice -- 通知方法
  * advisor切面 -- 包含一个通知和一个切点

### 使用spring的原生AOP接口实现aop
```java
public class SpringAopDemo {

    public static void main(String[] args) {
        //1.切点准备
        AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
        pointcut.setExpression("execution(* foo())");
        //2.准备通知
        MethodInterceptor advice = new MethodInterceptor() {
            @Override
            public Object invoke(MethodInvocation invocation) throws Throwable {
                System.out.println("before....");
                Object result = invocation.proceed();
                System.out.println("after.....");
                return result;
            }
        };
        //3. 准备切面
        DefaultPointcutAdvisor advisor = new DefaultPointcutAdvisor(pointcut,advice);

        //4. 创建代理
        Target target = new Target();
        ProxyFactory factory = new ProxyFactory();
        factory.setTarget(target);
        factory.addAdvisor(advisor);
//        factory.setInterfaces(target.getClass().getInterfaces());
//        factory.setProxyTargetClass(true);
        I1 proxy = (I1) factory.getProxy();
        //spring默认情况下不知道target是否实现了接口，会使用cglib增强，
        // 当使用了factory.setInterfaces()方法设置了接口，则会使用jdk实现
        System.out.println(proxy.getClass());
        proxy.foo();
        proxy.bar();
    }
    interface I1{
        void foo();

        void bar();
    }

    static class Target implements I1{
        @Override
        public void foo() {
            System.out.println("target foo");
        }

        @Override
        public void bar() {
            System.out.println("target bar");
        }
    }
}
```
### spring对增强代理的选择
三种情况：
* 在ProxyConfig中当proxyTargetClass=false，且目标实现了接口，用jdk实现代理
* proxyTargetClass=false，且目标没有实现接口，用cglib实现代理
* proxyTargetClass=true,使用cglib实现

### 切点匹配规则
```java
public class SpringAopDemo2 {

    public static void main(String[] args) throws NoSuchMethodException {
        AspectJExpressionPointcut pointcut1 = new AspectJExpressionPointcut();
        pointcut1.setExpression("execution(* bar())");
        //判断方法是否匹配
        System.out.println(pointcut1.matches(T1.class.getMethod("foo"), T1.class)); //false
        System.out.println(pointcut1.matches(T1.class.getMethod("bar"), T1.class));//true
        AspectJExpressionPointcut pointcut2 = new AspectJExpressionPointcut();
        //根据注解判断
        pointcut2.setExpression("@annotation(org.springframework.transaction.annotation.Transactional)");
        System.out.println(pointcut2.matches(T1.class.getMethod("foo"), T1.class));//true
        System.out.println(pointcut2.matches(T1.class.getMethod("bar"), T1.class));//false
        //spring中Transactional注解的匹配
        StaticMethodMatcherPointcut pointcut3 = new StaticMethodMatcherPointcut() {
            @Override
            public boolean matches(Method method, Class<?> targetClass) {
                 MergedAnnotations annotations = MergedAnnotations.from(method);
//                检查方法上是否存在Transactional注解
                if(annotations.isPresent(Transactional.class)){
                    return true;
                }
//                检查类上是否有Transactional注解,查找继承关系上的Transactional
                annotations =MergedAnnotations.from(targetClass, MergedAnnotations.SearchStrategy.TYPE_HIERARCHY);
                if(annotations.isPresent(Transactional.class)){
                    return true;
                }
                return false;
            }
        };
        System.out.println(pointcut3.matches(T1.class.getMethod("foo"), T1.class));//true
        System.out.println(pointcut3.matches(T1.class.getMethod("bar"), T1.class));//false
        System.out.println(pointcut3.matches(T2.class.getMethod("foo"), T2.class));//true
        System.out.println(pointcut3.matches(T3.class.getMethod("foo"), T3.class));//true

    }
    static class T1{
        @Transactional
        public void foo(){
        }
        public void bar(){
        }
    }
    @Transactional
    static class T2{

        public void foo(){
        }
        public void bar(){
        }
    }
    @Transactional
    interface I3{
        void foo();
    }

    static class T3 implements I3{
        @Override
        public void foo() {

        }
    }
```
### spring中的两种切面 @Aspect、Advisor
手动注册切面处理
```java
package com.dxy.data.springtest.aop.springaop;

import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.aop.Advisor;
import org.springframework.aop.aspectj.AspectJExpressionPointcut;
import org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator;
import org.springframework.aop.support.DefaultPointcutAdvisor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ConfigurationClassPostProcessor;
import org.springframework.context.support.GenericApplicationContext;

import java.util.List;

/**
 * @Description:
 * @CreateDate: Created in 2023/5/24 08:55
 * @Author: lijie3
 */
public class SpringAopDemo3 {

    public static void main(String[] args) throws NoSuchMethodException {

        GenericApplicationContext context = new GenericApplicationContext();
        context.registerBean("aspect1", Aspect1.class);
        context.registerBean("config", Config.class);
        context.registerBean(ConfigurationClassPostProcessor.class);
        //切面处理类
        context.registerBean(AnnotationAwareAspectJAutoProxyCreator1.class);
        context.refresh();
//        for (String beanDefinitionName : context.getBeanDefinitionNames()) {
//            System.out.println(beanDefinitionName);
//        }

        AnnotationAwareAspectJAutoProxyCreator1 creator = context.getBean(AnnotationAwareAspectJAutoProxyCreator1.class);
        //打印找到的切面
        final List<Advisor> advisors = creator.findEligibleAdvisors(T1.class, "t1");
        for (Advisor advisor : advisors) {
            System.out.println(advisor);
        }

        System.out.println("============");
        //判断是否需要创建代理
        final T1 p1 = (T1) creator.wrapIfNecessary(new T1(), "t1", "target1");
        System.out.println(p1.getClass());
        final T2 p2 = (T2) creator.wrapIfNecessary(new T2(), "t2", "target2");
        System.out.println(p2.getClass());
        p1.foo();
        p2.bar();

    }

    static class T1 {
        public void foo() {
            System.out.println("t1 foo");
        }

        public void bar() {
            System.out.println("t1 bar");
        }
    }

    static class T2 {
        public void bar() {
            System.out.println("t2 bar");
        }
    }

    interface I3 {
        void foo();
    }

    static class T3 implements I3 {
        @Override
        public void foo() {

        }
    }

    //使用高级切面
    @Aspect
    static class Aspect1 {

        @Before("execution(* foo())")
        public void before() {
            System.out.println("aspect before....");
        }

        @After("execution(* foo())")
        public void after() {
            System.out.println("aspect after....");
        }

    }

    @Configuration
    static class Config {
        @Bean
        public Advisor advisor(MethodInterceptor advice3) {
            AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
            pointcut.setExpression("execution(* foo())");
            return new DefaultPointcutAdvisor(pointcut, advice3);
        }

        @Bean
        public MethodInterceptor advice3() {
            return new MethodInterceptor() {
                @Override
                public Object invoke(MethodInvocation invocation) throws Throwable {
                    System.out.println("advice3 before.....");
                    final Object proceed = invocation.proceed();
                    System.out.println("advice3 after.....");
                    return proceed;
                }
            };
        }
    }

    static class AnnotationAwareAspectJAutoProxyCreator1 extends AnnotationAwareAspectJAutoProxyCreator {
        @Override
        protected List<Advisor> findEligibleAdvisors(Class<?> beanClass, String beanName) {
            return super.findEligibleAdvisors(beanClass,beanName);
        }

        @Override
        protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
            return super.wrapIfNecessary(bean, beanName, cacheKey);
        }
    }

}
```
### 创建代理的时机
* 当Bean1 与其他Bean不存在循环依赖时，在初始化之后创建代理
* 当Bean1与Bean2存在循环依赖时，在构造器执行之后，在依赖注入之前创建代理

**在依赖注入和初始化时不应该被增强，仍然要使用原始对象**

###  切面优先级
* 高级切面@Aspect注解切面可以使用@Order(200)来指定优先级
* 低级切面Advisor 可以使用setOrder(300)方法来指定优先级
* @Order注解中的值越大，优先级越低
* @Order加在方法上不起任何作用

### 高级切面转化低级切面,并执行链式调用 --使用适配器模式，责任链模式
```java
    public static void main(String[] args) throws Throwable {
        AspectInstanceFactory factory = new SingletonAspectInstanceFactory(new Aspect1());
        //高级切面转化为低级切面
        List<Advisor> advisorList = new ArrayList<>();
        for (Method method : Aspect1.class.getDeclaredMethods()) {
            if (method.isAnnotationPresent(Before.class)) {
                final String value = method.getAnnotation(Before.class).value();
                //设置切点
                AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
                pointcut.setExpression(value);
                //通知
                AspectJMethodBeforeAdvice advice = new AspectJMethodBeforeAdvice(method, pointcut, factory);
                //切面
                Advisor advisor = new DefaultPointcutAdvisor(pointcut, advice);
                advisorList.add(advisor);
            }
        }
        System.out.println("==========");
        for (Advisor advisor : advisorList) {
            System.out.println(advisor);

        }
        //4. 创建代理
        T1 target = new T1();
        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.setTarget(target);
        proxyFactory.addAdvice(ExposeInvocationInterceptor.INSTANCE); //准备将methodInvocation放入当前线程
        proxyFactory.addAdvisors(advisorList);
        System.out.println("==========");
        //通知类型转换,所有的通知类型都转化为环绕通知
        final List<Object> methodInterceptorList = proxyFactory.getInterceptorsAndDynamicInterceptionAdvice(T1.class.getMethod("foo"), T1.class);
        for (Object interceptor : methodInterceptorList) {
            System.out.println(interceptor);
        }
        //创建并执行调用
        MethodInvocation methodInvocation = new ReflectiveMethodInvocation1(null,target,T1.class.getMethod("foo"),new Object[0],T1.class,methodInterceptorList);
        methodInvocation.proceed();
    }
```
### aop中的责任链模式执行逻辑
```java
public class SpringAopDemo5 {

    public static void main(String[] args) throws Throwable {
        //调用链执行基本逻辑

        T1 target = new T1();
        List<MethodInterceptor> adviceList = new ArrayList<>();
        adviceList.add(new Advice1());
        adviceList.add(new Advice2());

        MyInvocation invocation = new MyInvocation(target,T1.class.getMethod("foo"),new Object[0],adviceList);
        invocation.proceed();
    }

    static class Advice1 implements MethodInterceptor{

        @Override
        public Object invoke(MethodInvocation invocation) throws Throwable {
            System.out.println("advice1.before()");
            Object result = invocation.proceed();
            System.out.println("advice1.after()");
            return result;
        }
    }
    static class Advice2 implements MethodInterceptor{

        @Override
        public Object invoke(MethodInvocation invocation) throws Throwable {
            System.out.println("advice2.before()");
            Object result = invocation.proceed();
            System.out.println("advice2.after()");
            return result;
        }
    }

    static class MyInvocation implements MethodInvocation{
    //目标对象
        private Object target;
        //目标方法
        private Method method;
        //方法参数
        private Object[] args;

        private int count = 1;

        List<MethodInterceptor> methodInterceptorList;

        public MyInvocation(Object target, Method method, Object[] args, List<MethodInterceptor> methodInterceptorList) {
            this.target = target;
            this.method = method;
            this.args = args;
            this.methodInterceptorList = methodInterceptorList;
        }

        @Override
        public Method getMethod() {
            return method;
        }

        @Override
        public Object[] getArguments() {
            return args;
        }

        //调用每一个环绕通知，并调用目标方法，这里使用了递归调用
        @Override
        public Object proceed() throws Throwable {
            if(count > methodInterceptorList.size()){
                //调用次数大于通知数量，则需要调用目标，返回并结束
                return method.invoke(target, args);
            }
            MethodInterceptor methodInterceptor = methodInterceptorList.get(count++ -1);
            return methodInterceptor.invoke(this);
        }

        @Override
        public Object getThis() {
            return target;
        }

        @Override
        public AccessibleObject getStaticPart() {
            return method;
        }
    }

    static class T1 {
        public void foo() {
            System.out.println("t1 foo");
        }

        public void bar() {
            System.out.println("t1 bar");
        }
    }
}
```
### 动态通知调用
静态：不带参数绑定,执行时不需要切点
```java
        @Before("execution(* foo())")
        public void before() {
            System.out.println("aspect before....");
        }
```   
动态：带参数绑定,需要切点对象
```java
        @After("execution(* foo()) && args(x)")
        public void after(int x) {
            System.out.println("aspect after....");
        }
```   
## springmvc
### dispatchServlet 初始化时机
基本环境配置：
```java
@Configuration
@ComponentScan
public class WebConfig {


    //内嵌web容器工厂
    @Bean
    public TomcatServletWebServerFactory tomcatServletWebServerFactory(){
        return new TomcatServletWebServerFactory();
    }

    //创建前端控制器
    @Bean
    public DispatcherServlet dispatcherServlet(){
        return new DispatcherServlet();
    }

    //注册DispatcherServlet
    public DispatcherServletRegistrationBean dispatcherServletRegistrationBean(DispatcherServlet dispatcherServlet){
        return new DispatcherServletRegistrationBean(dispatcherServlet,"/");
    }

}

    public static void main(String[] args) {
        AnnotationConfigServletWebServerApplicationContext context =
                new AnnotationConfigServletWebServerApplicationContext(WebConfig.class);

    }
```
* DispatcherServlet的初始化走的是servlet的体系，默认在首次收到请求会初始化dispatcherServlet

配置文件绑定到bean对象，对servlet配置进行修改
```java
@Configuration
@ComponentScan
@PropertySource("classpath:application.properties")
//绑定application.properties配置到配置类中
@EnableConfigurationProperties({WebMvcProperties.class, ServerProperties.class})
public class WebConfig {


    //内嵌web容器工厂
    @Bean
    public TomcatServletWebServerFactory tomcatServletWebServerFactory(){
        return new TomcatServletWebServerFactory();
    }

    //创建前端控制器
    @Bean
    public DispatcherServlet dispatcherServlet(){
        return new DispatcherServlet();
    }

    //注册DispatcherServlet
    public DispatcherServletRegistrationBean dispatcherServletRegistrationBean(DispatcherServlet dispatcherServlet,WebMvcProperties webMvcProperties){

        final DispatcherServletRegistrationBean dispatcherServletRegistrationBean = new DispatcherServletRegistrationBean(dispatcherServlet, "/");
        //设置启动顺序,设置后可以控制dispatcherServlet在容器启动时就创建
        dispatcherServletRegistrationBean.setLoadOnStartup(webMvcProperties.getServlet().getLoadOnStartup());
        return dispatcherServletRegistrationBean;
    }
}
```
```
spring.mvc.servlet.load-on-startup=1
```
### RequestMappingHandlerMapping
将用户请求（request）映射成controller中的方法（handler）
```java
    public static void main(String[] args) throws Exception {
        AnnotationConfigServletWebServerApplicationContext context =
                new AnnotationConfigServletWebServerApplicationContext(WebConfig.class);
//        解析@RequestMapping及派生注解，生成路径与控制器方法的映射关系，在初始化时就生成
        RequestMappingHandlerMapping handlerMapping = context.getBean(RequestMappingHandlerMapping.class);
        //获取映射结果
        final Map<RequestMappingInfo, HandlerMethod> handlerMethods = handlerMapping.getHandlerMethods();
        handlerMethods.forEach((k,v) -> {
            log.info("{}-{}",k,v);
        });
        //模拟请求，获取到执行链对象
        HandlerExecutionChain chain = handlerMapping.getHandler(new MockHttpServletRequest("get", "/hello"));
        System.out.println(chain);
    }
      @Bean
    public RequestMappingHandlerMapping requestMappingHandlerMapping(){
        return new RequestMappingHandlerMapping();
    }

```
### RequestmappingHandlerAdapter --处理器适配器
用来调用控制器使用RequestMapping注解标注的方法
```java
    public static void main(String[] args) throws Exception {
        AnnotationConfigServletWebServerApplicationContext context =
                new AnnotationConfigServletWebServerApplicationContext(WebConfig.class);
//        解析@RequestMapping及派生注解，生成路径与控制器方法的映射关系，在初始化时就生成
        RequestMappingHandlerMapping handlerMapping = context.getBean(RequestMappingHandlerMapping.class);
        //获取映射结果
        final Map<RequestMappingInfo, HandlerMethod> handlerMethods = handlerMapping.getHandlerMethods();
        handlerMethods.forEach((k,v) -> {
            log.info("{}-{}",k,v);
        });
        //模拟请求，获取到执行链对象
        final MockHttpServletRequest request = new MockHttpServletRequest("GET", "/hello");
        request.setParameter("name","dxyer");
        HandlerExecutionChain chain = handlerMapping.getHandler(request);
        System.out.println(chain);
        final MockHttpServletResponse response = new MockHttpServletResponse();
        //处理器的执行
        MyRequestMappingHandlerAdapter adapter  = context.getBean(MyRequestMappingHandlerAdapter.class);
        final ModelAndView modelAndView = adapter.invokeHandlerMethod(request, response, (HandlerMethod) chain.getHandler());
        log.info("result:{}",modelAndView);
        //adapte中的参数解析器
        System.out.println("==================");
        for (HandlerMethodArgumentResolver argumentResolver : adapter.getArgumentResolvers()) {
            System.out.println(argumentResolver);
        }
        System.out.println("==================");
        for (HandlerMethodReturnValueHandler returnValueHandler : adapter.getReturnValueHandlers()) {
            System.out.println(returnValueHandler);
        }
    }

    @Bean
    public MyRequestMappingHandlerAdapter myRequestMappingHandlerAdapter(){
        return new MyRequestMappingHandlerAdapter();
    }
  ```
### 自定义参数解析器
  参照上面的adapter添加参数解析器的方式，可以自定义参数解析器对参数来进行解析
```java
@Target({ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Token {
}
public class TokenArgumentResolver implements HandlerMethodArgumentResolver {
    //匹配支持某个参数
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        final Token token = parameter.getParameterAnnotation(Token.class);

        return token != null;
    }

    //解析参数
    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
       return webRequest.getHeader("token");
    }
}
    @Bean
    public MyRequestMappingHandlerAdapter myRequestMappingHandlerAdapter(){
        final TokenArgumentResolver tokenArgumentResolver = new TokenArgumentResolver();

        final MyRequestMappingHandlerAdapter myRequestMappingHandlerAdapter = new MyRequestMappingHandlerAdapter();

        myRequestMappingHandlerAdapter.setCustomArgumentResolvers(List.of(tokenArgumentResolver));
        return myRequestMappingHandlerAdapter;
    }

     request.addHeader("token", UUID.randomUUID().toString());


    @GetMapping("/hello")
    public String test1(@RequestParam("name") String name, @Token String token){
        return "hello " + name + ":" + token;
    }
  ```

  ### 自定义返回值处理器 （@ReponseBody是通过自定义返回值处理器来返回json对象的）

  ```java
  @Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Yml {
}

public class YmlReturnValueHandler implements HandlerMethodReturnValueHandler {
    @Override
    public boolean supportsReturnType(MethodParameter returnType) {
        final Yml methodAnnotation = returnType.getMethodAnnotation(Yml.class);
        return methodAnnotation != null;
    }

    /**
     * returnValue返回的真实值
     * @param returnValue
     * @param returnType
     * @param mavContainer
     * @param webRequest
     * @throws Exception
     */
    @Override
    public void handleReturnValue(Object returnValue, MethodParameter returnType, ModelAndViewContainer mavContainer, NativeWebRequest webRequest) throws Exception {
        //数据处理
        String str = new Yaml().dump(returnValue);
        //拿到响应
        final HttpServletResponse nativeResponse = webRequest.getNativeResponse(HttpServletResponse.class);
        //输出到响应流中
        nativeResponse.setContentType("text/plain;charset=utf-8");
        nativeResponse.getWriter().println(str);
        //通知spring请求已经处理完成
        mavContainer.setRequestHandled(true);

    }
}

    @Bean
    public MyRequestMappingHandlerAdapter myRequestMappingHandlerAdapter(){
        final TokenArgumentResolver tokenArgumentResolver = new TokenArgumentResolver();

        final MyRequestMappingHandlerAdapter myRequestMappingHandlerAdapter = new MyRequestMappingHandlerAdapter();

        final YmlReturnValueHandler ymlReturnValueHandler = new YmlReturnValueHandler();

        myRequestMappingHandlerAdapter.setCustomArgumentResolvers(List.of(tokenArgumentResolver));
        myRequestMappingHandlerAdapter.setCustomReturnValueHandlers(List.of(ymlReturnValueHandler));
        return myRequestMappingHandlerAdapter;
    }
        @Yml
    @GetMapping("/user")
    public User getUser(){
        return new User("dxyer",22);
    }


        final byte[] contentAsByteArray = response.getContentAsByteArray();
        System.out.println(new String(contentAsByteArray, StandardCharsets.UTF_8));
  ```

### 参数解析
参数解析绑定示例：
```java
    public static void main(String[] args) throws Exception {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(WebConfig.class);
        final DefaultListableBeanFactory defaultListableBeanFactory = context.getDefaultListableBeanFactory();
//控制器方法封装
        HandlerMethod handlerMethod = new HandlerMethod(new TestController(),
                TestController.class.getMethod("test2", String.class, int.class,String.class));
        //准备ModelAndViewContainer
        ModelAndViewContainer container = new ModelAndViewContainer();
        MockHttpServletRequest request = new MockHttpServletRequest("GET", "/test2");
        request.setParameter("name","dxyer");
        request.setParameter("age","18");

        //做数据类型转换
        DefaultDataBinderFactory factory = new DefaultDataBinderFactory(null);

        for (MethodParameter methodParameter : handlerMethod.getMethodParameters()) {
            //设置参数名解析起
            methodParameter.initParameterNameDiscovery(new DefaultParameterNameDiscoverer());
            final Annotation[] parameterAnnotations = methodParameter.getParameterAnnotations();
            RequestParamMethodArgumentResolver resolver = new RequestParamMethodArgumentResolver(defaultListableBeanFactory,true);// true，不带@RequestParam的参数都会使用该解析器来进行解析
             Object value = null;
            if(resolver.supportsParameter(methodParameter)){
                 value = resolver.resolveArgument(methodParameter, container, new ServletWebRequest(request), factory);
            }
            String annotations = "";
            if(parameterAnnotations != null && parameterAnnotations.length > 0){
                for (Annotation parameterAnnotation : parameterAnnotations) {
                    annotations += parameterAnnotation.annotationType().getSimpleName() + ",";
                }
            }
            System.out.println(methodParameter.getParameterIndex()+ ":"
                    + methodParameter.getParameterType().getSimpleName()+ ","
                            + methodParameter.getParameterName() + ","
            + annotations + "->" + value) ;
        }
        
    }
     

```

### 参数解析组合模式
逐一调用解析器来判断是否支持当前参数 --HandlerMethodArgumentResolverComposite
```java
 HandlerMethodArgumentResolverComposite composite = new HandlerMethodArgumentResolverComposite();
        //多个解析器组合
        composite.addResolvers( new RequestParamMethodArgumentResolver(defaultListableBeanFactory,true));
```

路径参数解析器
PathVariableMethodArgumentResolver

**方法参数名的获取**
默认情况下 javac test1.java编译出来的参数名不会被记录
javac -parameters test1.java 进行编译时，会记录方法参数名 --通过反射可以获取
javac -g test1.java 编译后在本地变量表中能记录参数名称 --通过ASM能获取到，在spring中可以通过ParameterNameDiscoverer,只能获取普通类方法上的参数名称，对接口无效
spring的DefaultParameterNameDiscoverer 同时支持了上面两种方式



### 对象绑定与类型转换
底层转换接口（有两套接口）
* Printer接口将其他类型转化为String
* Parser把String转化为其他类型
* Formatter综合Printer和Parser功能
* Converter把S转化为类型T

高层转换接口

### spring对泛型参数的获取
使用GenericTypeResolver可以直接获取到泛型参数信息
```java
public class SG<WebConfig> {
}
public class SubG extends SG<WebConfig>{
}
   public static void main(String[] args) {
        //使用jdk的api获取泛型参数类型
        Type type = SubG.class.getGenericSuperclass();
        if(type instanceof ParameterizedType){
            ParameterizedType type1= (ParameterizedType) type;
            System.out.println(type1.getActualTypeArguments()[0]);
        }

        //使用spring的api获取泛型参数类型
        final Class<?> aClass = GenericTypeResolver.resolveTypeArgument(SubG.class, SG.class);
        System.out.println(aClass);
    }
```

### ControllerAdvice 控制器增强
* 添加@ExceptionHandler 添加异常处理
* 添加@ModelAttribute 补充模型数据
* @InitBinder 绑定数据


### 返回值处理器

### messageConverter

### @ControllerAdvice +ResponseBodyAdvice
实现对返回值的自动类型转换
```java
    @ControllerAdvice
    static class MyControllerAdvice implements ResponseBodyAdvice<Object>{

        //判断满足条件转换返回值
        @Override
        public boolean supports(MethodParameter returnType, Class<? extends HttpMessageConverter<?>> converterType) {
            //只有返回application/json的数据才需要包装
            if(returnType.getMethodAnnotation(ResponseBody.class)!= null ||
                    AnnotationUtils.findAnnotation(returnType.getContainingClass(),ResponseBody.class)!= null
            ){
                return true;
            }
            return false;
        }

        //实现转换逻辑
        @Override
        public Object beforeBodyWrite(Object body, MethodParameter returnType, MediaType selectedContentType, Class<? extends HttpMessageConverter<?>> selectedConverterType, ServerHttpRequest request, ServerHttpResponse response) {
            if(body instanceof Result){
                return body;
            }
            return Result.ok(body);
        }
    }
```

### @ControllerAdvice + @ExceptionHandler
实现对异常的处理

### tomcat的异常处理
```
    //定制tomcat的错误页面
    @Bean
    public ErrorPageRegistrar errorPageRegistrar(){
        return new ErrorPageRegistrar() {
            @Override
            public void registerErrorPages(ErrorPageRegistry webserverFactory) {
                //通过请求转发跳转到/error页面
                webserverFactory.addErrorPages(new ErrorPage(("/error")));
            }
        };
    }
    //定制tomcat的错误页面 processor
    @Bean
    public ErrorPageRegistrarBeanPostProcessor errorPageRegistrarBeanPostProcessor(){
        return new ErrorPageRegistrarBeanPostProcessor();
    }
```
controller中的控制器方法
```java
    @RequestMapping("/test")
    public ModelAndView test(){
        int i  =1/0;
        return null;
    }

        @RequestMapping("/error")
    @ResponseBody
    public Map<String,Object> error(HttpServletRequest request){
        Map<String,Object> map =new HashMap<>();
        final Throwable e = (Throwable) request.getAttribute(RequestDispatcher.ERROR_EXCEPTION);
        map.put("error",e.getMessage());
        return map;
    }
```

### BasicErrorController--提供基本的错误处理映射
配置BasicErrorController不需要自己在controller中定义error路径
```
    @Bean
    public BasicErrorController basicErrorController(){
        final ErrorProperties errorProperties = new ErrorProperties();
        errorProperties.setIncludeException(true);
        return new BasicErrorController(new DefaultErrorAttributes(),errorProperties);
    }
```









## 3. spring的基本实现原理
使用简单工厂（beanFactory.getBean(...)） + 配置文件 + 反射(实例化bean) 实现bean的创建和管理

## 4. IoC 和 DI 的区别
IoC: Inversion of Control ,控制反转，是代码设计的一种思想，是将控制对象创建的工作由程序员转移到框架中，实现代码的松耦合
DI： Dependency Injection,依赖注入，是spring 框架中对IoC的一种实现，在对象A依赖于对象B的关系中，框架实现将B注入到A中，而不用程序员自己进行set注入

## 5. 紧耦合和松耦合的区别
紧耦合： 尽可能的合理划分功能模块，功能模块之间耦合紧密
松耦合： 模块间的关系尽可能简单，功能快之间的耦合度低
在spring框架中通过依赖注入的方式降低了各个实例之间的耦合

## 6. 对BeanFactory的理解
BeanFactory是spring框架的顶层容器接口，主要功能是根据bean的定义实现对bean的创建，BeanFactory的扩展接口或类还实现了对bean的更加细粒度的控制，比如bean定义的修改，bean销毁等

## 7. BeanDefinition的作用是什么
BeanDefinition 规定了创建Bean的具体细节，比如bean是单例还是原型类型，是否懒加载、依赖的bean的列表，是否自动注入、bean初始化后调用方法、bean销毁后调用的方法




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
3. 使用反射根据条件推断需要使用的类构造器，生成bean的实例
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

## 16 spring 整合mybatis的原理（与springboot的starter的自动注入原理类似）
核心思想:将myabtis的配置类、sqlSession、mapper的代理类都放入到spring容器，由spring来管理
用到的知识： 
1. FactoryBean，通过FactoryBean对象来包裹mybatis的mapper接口，从而将myatbis的mapper代理对象放入spring中管理
2. 通过ClassPathBeanDeifintionScanner扫描@MapperScan指定的basePackage中的mapper接口，拿到mapper接口的beanDefinition定义
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

主要流程如下

0. 启动main方法开始

1. 初始化配置：通过类加载器，（loadFactories）读取classpath下所有的spring.factories配置文件，创建一些初始配置对象；通知监听者应用程序启动开始，创建环境对象environment，用于读取环境配置 如 application.yml

2. 创建应用程序上下文-createApplicationContext，创建 bean工厂对象

3. 刷新上下文（启动核心）
    * 3.1 配置工厂对象，包括上下文类加载器，对象发布处理器，beanFactoryPostProcessor
    * 3.2 注册并实例化bean工厂发布处理器，并且调用这些处理器，对包扫描解析(主要是class文件)
    * 3.3 注册并实例化bean发布处理器 beanPostProcessor
    * 3.4 初始化一些与上下文有特别关系的bean对象（创建tomcat服务器）
    * 3.5 实例化所有bean工厂缓存的bean对象（剩下的）
    * 3.6 发布通知-通知上下文刷新完成（启动tomcat服务器）

4. 通知监听者-启动程序完成，启动tomcat或其他web容器

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
   1. 使用@Import 导入了AutoConfigurationImportSelector实现对spring.factories文件中的EnableAutoConfiguration 的值列表解析
   2. AutoConfigurationImportSelector 是DeferredImportSelector的子类，是一个延迟的importSelector，当没有实现DeferredImportSelector的getImportGroup时，会直接调用DeferredImportSelector子类的selectImport
   3. 如果实现了getImportGroup方法，则会调用getImportGroup返回的group的子类的selectImport
   4. 在调用group的selectImports方法之前，会先调用group的process方法，process方法里会真正的进入spi机制加载自动配置类的逻辑，使用SpringFactoriesLoader的loadFactoryNames拿到所有自动配置类的完整类名列表（实际上读取META_INFO/spring.factories文件是在创建SpringApplication时就已经进行了读取，这里拿到的列表是直接从cache中get到的键为EnableAutoConfiguration的完整类名的list）
4. @ComponentScan： 扫描包，相当于在spring.xml配置中的`<context:component-scan>`,扫描指定路径下的class
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


## 24 spring 中@Import的用法
@Import是导入其他配置类或是其他组件的工具: 
   * 导入普通bean，通过import导入的bean（即使没有使用@Configuration注解标注）在spring中会被认为是一个配置bean，可以在被导入的类中使用@Bean注解
   * 导入实现了ImportSelector接口的类,会执行 selectImports的方法拿到一组类名，从而加载数组中对应的类到spring容器中
   * 导入DeferredImportSelector接口的类，与ImportSelector类似，也会去执行相应的方法拿到类名然后加载类到spring容器中
   * 导入ImportBeanDefinitionRegistrar接口的实现类，执行registerBeanDefinitions方法将自定义的BeanDefinition注入到spring容器中（mybatis就是使用的这种方式实现将mapper接口注入到spring容器中从而实现sql的查询）
@ImportResource 导入一个xml格式的配置文件

## 25 spring的Bean的后置处理器

## 26 spring的BeanFactory的后置处理器

## 27 spring @Value原理 
通过AutowiredAnnotationBeanPostProcessor 在bean被实例化后立即通过反射进行@Autowired或@Value的属性注入

## 28. @Resource 与@Autowired的区别
@Resource是jdk带的注解，在spring中是先根据属性名字再根据类型查找bean
@Autowired是spring提供的注解，是先根据类型、然后在根据名字查找bean

## 29. @Configuration的理解
默认情况下@Configurtaion在spring中会生成一个代理对象，用来保证直接执行@Bean注解的方法时拿到的bean是单例的

## 30 @Primary 的作用
在spring加载bean时，如果遇到两个类型相同且bean的名称相同的bean时，会自动忽略掉其中一个,这种情况下不会报错
```java
@Service
public class UserService {}

@Configuration
@ComponentScan(value = {"com.dxy.data.springtest.service"})
public class TestConfig {

    @Bean
    public UserService userService(){
        return new UserService();
    }
//    @Bean
//    public UserService userService2(){
//        return new UserService();
//    }
}
```
但是当两个bean的名称不同而类型不同时，如果没有在某个bean上指定@Primary，则会报错：
```java
@Configuration
@ComponentScan(value = {"com.dxy.data.springtest.service"})
public class TestConfig {

    @Bean
    public UserService userService(){
        return new UserService();
    }
    //报错
    @Bean
    public UserService userService2(){
        return new UserService();
    }
}
```
这时需要指定@Primary修饰其中的一个bean来将其作为首选的bean

## 31.注册一个bean的几种方式
* @Component及其派生注解
* @Bean
* @Import的四种导入方式
* @ImportResource导入xml配置文件来创建
* 使用FactoryBean 通过getObject拿到Bean（默认情况下这个bean并不是spring 启动时进行创建的，可以使用SmartFacoryBean设置isEagerInit = true来初始化时加载）
* 通过`ApplicationContext.registerBean()/register(）`方法来注册
* 通过`ApplicationContext.register()`注册一个beanDefition

## 32.bean的作用域
* singleton
* prototype
web环境下
* session 利用session.setAttribute()将bean设置到session中
* request 利用request.setAttribute()
* application 利用application.setAttribute()


## springboot

生成干净的只有pom.xml的文件：
 curl -G https://start.spring.io/pom.xml -d packaging=war -o pom.xml

 ### boot执行流程
**springApplication构造分析**
```java
SpringApplication.run(App.class,args)
```
SpringApplication构造器中的执行流程：
* 根据引导类确定BeanDefinition的来源（BeanDefinition的一个来源）
* 推断应用的类型（非web应用、servlet web应用，react web应用）
* 添加applicationContext的初始化器
* 添加监听器与事件
* 主类推断

源码
```java
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
		this.bootstrapRegistryInitializers = getBootstrapRegistryInitializersFromSpringFactories();
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```
推断应用类型：
```java
static WebApplicationType deduceFromClasspath() {
		if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
				&& !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
			return WebApplicationType.REACTIVE;
		}
		for (String className : SERVLET_INDICATOR_CLASSES) {
			if (!ClassUtils.isPresent(className, null)) {
				return WebApplicationType.NONE;
			}
		}
		return WebApplicationType.SERVLET;
	}
```
代码演示
```java
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        SpringApplication springApplication = new SpringApplication(BootDemo01.class);
        Set<String> sources = new HashSet<>();
        //添加bean的来源
//        sources.add("classpath:application.xml");
//        springApplication.setSources(sources);
        //推断应用类型
        final Method deduceFromClasspath = WebApplicationType.class.getDeclaredMethod("deduceFromClasspath");
        deduceFromClasspath.setAccessible(true);
        System.out.println("应用类型为："+ deduceFromClasspath.invoke(null));
        //对applicationContext扩展
        springApplication.addInitializers(new ApplicationContextInitializer<ConfigurableApplicationContext>() {
            @Override
            public void initialize(ConfigurableApplicationContext applicationContext) {
                if (applicationContext instanceof GenericApplicationContext) {
                    GenericApplicationContext gac = (GenericApplicationContext) applicationContext;
                    //对beandefinition进行扩展
                    gac.registerBean("bean3",Bean3.class);
                }
            }
        });
        //监听器添加
        springApplication.addListeners(new ApplicationListener<ApplicationEvent>() {
            @Override
            public void onApplicationEvent(ApplicationEvent event) {
                //打印所有的事件
                System.out.println("\t事件为："+ event.getClass());
            }
        });
        //主类推断，判断main方法所在的类
        Method deduceMainApplicationClass = SpringApplication.class.getDeclaredMethod("deduceMainApplicationClass");
        deduceMainApplicationClass.setAccessible(true);
        System.out.println("\t主类是： "+deduceMainApplicationClass.invoke(springApplication));
        ConfigurableApplicationContext context = springApplication.run(args);
        for (String beanDefinitionName : context.getBeanDefinitionNames()) {
            //初始化器提供的来源为null
           log.info("name:{},source:{}",beanDefinitionName,context.getBeanFactory().getBeanDefinition(beanDefinitionName).getResourceDescription());
        }
        context.close();
    }
    @Bean
    public ServletWebServerFactory servletWebServerFactory(){
        return new TomcatServletWebServerFactory();
    }
```

**springApplication run方法分析（一共12步）** 

1. 得到springApplicationRunListener，实际上是发布事件，发布application starting事件
事件发布演示：
```java
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException, ClassNotFoundException, InstantiationException {
        SpringApplication springApplication = new SpringApplication(BootDemo02.class);
        //添加监听器
        springApplication.addListeners(e -> System.out.println("发布了事件：" + e.getClass()));
        //获取事件发布器名称
        List<String> names = SpringFactoriesLoader.loadFactoryNames(SpringApplicationRunListener.class, BootDemo02.class.getClassLoader());
        for (String name : names) {
            System.out.println(name);
            Class<?> clazz = Class.forName(name);
            Constructor<?> constructor = clazz.getConstructor(SpringApplication.class, String[].class);
            SpringApplicationRunListener publisher = (SpringApplicationRunListener) constructor.newInstance(springApplication, args);
            //发布事件
            //springboot开始启动
            final DefaultBootstrapContext bootstrapContext = new DefaultBootstrapContext();
            publisher.starting(bootstrapContext);
            //准备环境完成之后
            publisher.environmentPrepared(bootstrapContext,new StandardEnvironment());
            //在spring容器创建并调用初始化器之后触发发送该事件
            GenericApplicationContext context = new GenericApplicationContext();
            publisher.contextPrepared(context);
            //所有beanDefinition加载完毕之后发布该事件
            publisher.contextLoaded(context);
            //refresh()方法调用结束后，spring容器初始化完成，触发该事件
            context.refresh();
            publisher.started(context);

            //springboot启动完毕
            publisher.running(context);
            //springboot启动过程中出现了错误
            publisher.failed(context,new Exception("出错演示"));
        }
```

2. 封装启动args (在第12步会使用这个封装的参数来执行)
```java
        //2. 封装启动参数
        DefaultApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
```

3.  初始化环境变量对象 ApplicationEnvironment
默认包括系统环境变量和系统属性
  
 添加commandLine的属性配置来源
4. 将属性中驼峰、减号连接、下划线连接的属性进行统一匹配成建好匹配
```java
 ConfigurationPropertySource.attach(environment);
```

5. 对ApplicationEnvironment 进一步增强，使用EnvironmentPostProcessor进行处理
 * 从各个来源添加配置信息，如添加application.properties 属性配置来源

6. 将配置文件中的键值与SpringApplication中的属性进行绑定
7. 输出springboot的banner信息

  

8. 创建spring容器
9. 准备容器
10.  从各种来源加载各种bean定义
11.  refresh刷新容器
```java
        SpringApplication springApplication = new SpringApplication(BootDemo03.class);
        springApplication.addInitializers(new ApplicationContextInitializer<ConfigurableApplicationContext>() {
            @Override
            public void initialize(ConfigurableApplicationContext applicationContext) {
                System.out.println("执行初始化器增强");
            }
        });
        //8.根据前面的推断构造spring 容器
        final GenericApplicationContext applicationContext = createApplicationContext(WebApplicationType.SERVLET);
        //9. 准备容器
        for (ApplicationContextInitializer initializer : springApplication.getInitializers()) {
            initializer.initialize(applicationContext);
        }
        //10. 从各种来源加载各种bean定义
        //演示从配置类中加载bean定义
        AnnotatedBeanDefinitionReader reader = new AnnotatedBeanDefinitionReader(applicationContext.getDefaultListableBeanFactory());
        reader.register(Config1.class);
        //从xml中读取bean定义
//        XmlBeanDefinitionReader reader2 = new XmlBeanDefinitionReader(applicationContext.getDefaultListableBeanFactory());
//        reader2.loadBeanDefinitions(new ClassPathResource("application.xml"));
        ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(applicationContext.getDefaultListableBeanFactory());
        scanner.scan("com.dxy.data.springtest.boot.bean");
        //11 执行refresh刷新容器
        applicationContext.refresh();
        for (String beanDefinitionName : applicationContext.getBeanDefinitionNames()) {
            System.out.println("name:" + beanDefinitionName +
                    ",-----source:" + applicationContext.getBeanFactory().getBeanDefinition(beanDefinitionName).getResourceDescription());
        }
```

12.  执行runner 两种runner方法执行
```java
        @Bean
        public CommandLineRunner commandLineRunner(){
            return new CommandLineRunner() {
                @Override
                public void run(String... args) throws Exception {
                    System.out.println("执行commandrunner");
                }
            };
        }

        @Bean
        public ApplicationRunner applicationRunner(){
            return new ApplicationRunner() {
                @Override
                public void run(ApplicationArguments args) throws Exception {
                    System.out.println("执行application runner");
                }
            };
        }



        //12.执行runner方法
        for (CommandLineRunner runner : applicationContext.getBeansOfType(CommandLineRunner.class).values()) {
            runner.run(args);
        }
        for (ApplicationRunner applicationRunner : applicationContext.getBeansOfType(ApplicationRunner.class).values()) {
            applicationRunner.run(applicationArguments);
        }
```

**boot启动总结**



## tomcat内嵌容器

### tomcat中的重要组件
Connector 协议端口
Engine  
  host
  Context 应用，包括html、jsp、class文件（servlet、filter、listener）等

tomcat 手动启动
```java
    public static void main(String[] args) throws IOException, LifecycleException {
        Tomcat tomcat = new Tomcat();
        tomcat.setBaseDir("tomcat");
        //设置docbase,临时目录
        File docBase = Files.createTempDirectory("boot.").toFile();
        //创建tomcat项目，在tomcat中称为context
         Context context = tomcat.addContext("", docBase.getAbsolutePath());

        //添加 servlet
        context.addServletContainerInitializer(new ServletContainerInitializer() {
            @Override
            public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException {
                final HelloServlet helloServlet = new HelloServlet();
                ctx.addServlet("hello",helloServlet).addMapping("/hello");
            }
        }, Collections.emptySet());
        //启动tomcat
        tomcat.start();

        //设置连接信息
        Connector connector = new Connector(new Http11Nio2Protocol());
        connector.setPort(8080);
        tomcat.setConnector(connector);
        docBase.deleteOnExit();
    }

   //sevlet
   public class HelloServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=utf-8");
        resp.getWriter().println("<h2>hello</h2>");
    }
} 
```

### spring mvc 与tomcat的整合
下面的代码实际上是在spring的onfresh()方法中执行了，实现了对tomcat的创建与启动
```java
public class TomcatDemo02 {

    public static void main(String[] args) throws IOException, LifecycleException {
        Tomcat tomcat = new Tomcat();
        tomcat.setBaseDir("tomcat");
        //设置docbase,临时目录
        File docBase = Files.createTempDirectory("boot.").toFile();
        //创建tomcat项目，在tomcat中称为context
         Context context = tomcat.addContext("", docBase.getAbsolutePath());
         //创建spring context
        WebApplicationContext springContext = getWebApplicationContext();


        //添加 servlet
        context.addServletContainerInitializer(new ServletContainerInitializer() {
            @Override
            public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException {
                final HelloServlet helloServlet = new HelloServlet();
                ctx.addServlet("hello",helloServlet).addMapping("/hello");
//                 DispatcherServlet dispatcherServlet = springContext.getBean(DispatcherServlet.class);
//                 ctx.addServlet("dispatcherServlet",dispatcherServlet).addMapping("/");
                //将spring管理的servlet都放入servletcontext中
                for (ServletRegistrationBean registrationBean : springContext.getBeansOfType(ServletRegistrationBean.class).values()) {
                    registrationBean.onStartup(ctx);
                }
            }
        }, Collections.emptySet());
        //启动tomcat
        tomcat.start();

        //设置连接信息
        Connector connector = new Connector(new Http11Nio2Protocol());
        connector.setPort(8080);
        tomcat.setConnector(connector);
        docBase.deleteOnExit();
    }

    public static WebApplicationContext getWebApplicationContext(){
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(Config.class);
        context.refresh();
        return context;
    }

    @Configuration
    static class Config{
        @Bean
        public DispatcherServletRegistrationBean registrationBean(DispatcherServlet dispatcherServlet){
            return new DispatcherServletRegistrationBean(dispatcherServlet,"/");
        }
        @Bean
        public DispatcherServlet dispatcherServlet(WebApplicationContext applicationContext){
            return new DispatcherServlet(applicationContext);
        }

        @Bean
        public RequestMappingHandlerAdapter requestMappingHandlerAdapter(){
            RequestMappingHandlerAdapter handlerAdapter = new RequestMappingHandlerAdapter();
            handlerAdapter.setMessageConverters(List.of(new MappingJackson2HttpMessageConverter()));
            return handlerAdapter;
        }
        @RestController
        static class MyController{
            @GetMapping("/hello2")
            public Map<String,Object> hello() {
                Map<String, Object> map = new HashMap<>();
                map.put("hello","hello spring");
                return map;
            }
        }
    }
}
```

## 自动配置类
第三方应用的配置类需要进行整合时为了方便需要进行自动配置
1. 通过@Import来导入第三方的配置类
```java
public class AutoConfigDemo01 {

    public static void main(String[] args) {

        GenericApplicationContext context = new GenericApplicationContext();
        context.registerBean("MyConfig",MyConfig.class);
        context.registerBean(ConfigurationClassPostProcessor.class);
        context.refresh();
        for (String beanDefinitionName : context.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }
    }

    //本项目配置类
    @Configuration
    //通过import导入第三方配置类
    @Import(ThirdAutoConfig1.class)
    static class MyConfig{

    }
    //第三方配置类
    @Configuration
    static class ThirdAutoConfig1 {
        @Bean
        public Bean1 bean1() {
            return new Bean1();
        }
    }
}
```
2. 实现importSelector接口，结合@Import 导入配置类
```java
 //本项目配置类
    @Configuration
    //通过import导入第三方配置类
    @Import(MyImportSelector.class)
    static class MyConfig{

    }

    static class MyImportSelector implements ImportSelector{

        @Override
        public String[] selectImports(AnnotationMetadata importingClassMetadata) {
            return new String[]{ThirdAutoConfig1.class.getName(),ThirdAutoConfig2.class.getName()};
        }

    }


    //第三方配置类
    @Configuration
    static class ThirdAutoConfig1 {

        @Bean
        public Bean1 bean1() {
            return new Bean1();
        }
    }
    @Configuration
    static class ThirdAutoConfig2 {

        @Bean
        public Bean2 bean2() {
            return new Bean2();
        }
    }

```
3. 使用spring.factories配置文件添加配置类实现自动导入配置类
spring.factories
```
com.dxy.data.springtest.autoconfig.AutoConfigDemo01$MyImportSelector =\
com.dxy.data.springtest.autoconfig.AutoConfigDemo01.ThirdAutoConfig1,\
com.dxy.data.springtest.autoconfig.AutoConfigDemo01.ThirdAutoConfig2
```

```java
    static class MyImportSelector implements ImportSelector{

        @Override
        public String[] selectImports(AnnotationMetadata importingClassMetadata) {
            List<String> list = SpringFactoriesLoader.loadFactoryNames(MyImportSelector.class, null);
            return list.toArray(new String[0]);
        }

    }
```
4. 当配置类中出现了相同的bean，默认情况下本项目的bean会生效
- @Import导入的配置类先会被解析，本项目的配置类中Bean会后解析
- springboot中牧人不允许相同名称相同类型的Bean被注册
- 实现DeferredImportSelector接口可以将第三方的配置类导入推出，优先导入本项目的配置类中的Bean
- 出现同名bean时，使用@ConditionalOnMissingBean 忽略已经导入的bean

### 事务自动配置

### 条件自动装配

### FactoryBean
* 工厂类产生的对象会走初始化后的流程，可以被代理
* 当通过BeanFactory根据类型获取FactoryBean创建的对象时，如果没有在工厂类指定返回对象的类型，则spring会抛出异常

### @Indexed
编译时根据@Indexed生成META-INF/spring.components文件
如果存在文件则直接根据文件中的配置扫描指定的bean，不存在走包扫描方式


### @Value


### @Autowired

### 事件监听器





