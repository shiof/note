---
layout: article
title:	Spring序列- Bean生命周期（1）
date:	2019-09-26 16:03:55
categories:
    - article
tags:
    - Java
    - Spring
---

### Spring Bean注册的方式

注解注册方式

1. `@bean`： 导入第三方jar包，或者不在`Spring`扫描路径中的类，以及没有加上`@Component`等注解，可以在方式上加上`@bean`，然后返回该对象的实例。

2. `@ComponentScan`：在`Spring`扫描路径中的类，可以在该类上加`@Component`、`@Controller`、`@Service`、`@Reponsitory`等注解。

3. `@Import`： 没有加上的`@Component`等注解，可以使用`@Import`来注册Bean（一般`@EnableXXX`等注解也是用它实现）。
    
    3.1 实现`ImportSelector`接口的方式，然后在`@Import`中引入该类。
    
    3.2 实现`ImportBeanDefinitionRegistrar` 接口，把所有需要添加到容器中的Bean添加到容器中。也可以用来判断是依赖的Bean是否已经被注册，如果没有就不加入到容器中。

4. 实现`FactoryBean`接口 

