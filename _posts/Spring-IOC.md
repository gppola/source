---
title: Spring IOC
date: 2020-12-02 20:26:37
tags: Spring
---

ioc就是绑定接口和类，使用构造函数或者get set方法将对象自动依赖注入，aop就是动态代理模式
```
1. 配置Bean
<bean class=""> @Bean #Component

2. 加载Spring容器
xml: new ClassPathXmlApplicationContext("xml")
@: new AnnotationConfigApplicationContext(配置类)

3.
使用Bean
spring.getBean("user")
```
![](/images/spring_ioc/init.png)


## Q&A
- Dependency injection
https://en.wikipedia.org/wiki/Dependency_injection
>Dependency injection is one form of the broader technique of inversion of control. A client who wants to call some services should not have to know how to construct those services. Instead, the client delegates the responsibility of providing its services to external code (the injector). The client is not allowed to call the injector code; it is the injector that constructs the services. The injector then injects (passes) the services into the client which might already exist or may also be constructed by the injector. The client then uses the services. This means the client does not need to know about the injector, how to construct the services, or even which actual services it is using. The client only needs to know about the intrinsic interfaces of the services because these define how the client may use the services. **This separates the responsibility of "use" from the responsibility of "construction".**

- IoC vs DI
https://martinfowler.com/articles/injection.html#InversionOfControl
>As a result I think we need a more specific name for this pattern. Inversion of Control is too generic a term, and thus people find it confusing. As a result with a lot of discussion with various IoC advocates we settled on the name Dependency Injection.

- BeanFactory getBean 和 ApplicationContext getBean 都可以调用getBean,都可以作为容器，二者区别是什么？

Application 包含 BeanFactory 
BeanFactory 职责单一，简单工厂模式，仅仅负责生产Bean getBean
Application 更多的负责对外打交道、解析配置、配置文件怎么生产Bean原材料，相关Spring ioc加载的一切相关配套服务（监听器，对外扩展接口的调用）