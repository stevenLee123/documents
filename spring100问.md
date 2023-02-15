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

