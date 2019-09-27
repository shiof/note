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

>`beanDefinitionMap`： IOC容器
>
>`rootBeanDefinition`：容器的根节点

### Spring Bean 的生命周期

1. `Bean`初始化和销毁

    1.1 `@Bean`属性指定方法

    ~~~java
    @Configuration
    public class InitBean {
        @Bean(value = "xiaoming", initMethod = "init", destroyMethod = "destroy")
        public UserEntity userEntity(){
            return new UserEntity("xiaoming","123456", 18);
        }
    }
    ~~~

    1.2 `@PostConstruct`和`@PreDestroy`
    
    ~~~java
    public class UserEntity {
       private String username;
       private String password;
       private int age;
       
       @PostConstruct
       private void init() {
       }
    
       @PreDestroy
       private void destroy() {
       }
    }
    ~~~
    
    1.3 实现`InitializingBean`和`DisposableBean`接口
    
    ~~~java
    public class UserEntity implements InitializingBean, DisposableBean {
        private String username;
        private String password;
        private int age;
    
        @Override
        public void afterPropertiesSet() throws Exception {
            
        }
        
        @Override
        public void destroy() throws Exception {
            
        }
    }
    ~~~

    >`InitializingBean`接口：当`bean`属性赋值和初始化完成时，调用`afterPropertiesSet`方法() 。
    >
    >`DisposableBean`接口： 当`bean`被销毁时，调用`destroy`方法。

2. `Bean`前后置处理器

~~~java
public class BeanEntity implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return null;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return null;
    }
}
~~~

> 实现`Bean`功能增强。
