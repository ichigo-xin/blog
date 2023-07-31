---
title: "手写Spring整合Jpa流程"
date: 2023-07-26T21:56:50+08:00
draft: false
tags: [Spring Data JPA]
categories: [code, spring]

---

在本篇文章中，我将手写模拟Spring整合Jpa流程。

## 项目搭建

项目依赖:

```java
    <dependencies>
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.1</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.28</version>
        </dependency>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.4.32.Final</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
            <scope>compile</scope>
        </dependency>
        <!-- 连接池 -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.8</version>
        </dependency>

        <!-- Spring test  -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.3.13</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.data</groupId>
                <artifactId>spring-data-bom</artifactId>
                <version>2020.0.14</version>
                <scope>import</scope>
                <type>pom</type>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

配置类

```java
package com.xin.config;

import com.alibaba.druid.pool.DruidDataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;


import javax.sql.DataSource;


/**
 * @author : ichigo-xin
 * @date 2023/7/18
 */
@Configuration
@EnableJpaRepositories(basePackages = "com.xin.repository")
@EnableTransactionManagement
@ComponentScan("com.xin")
public class SpringDataConfig {

    @Bean
    public DataSource druidDataSource(){
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        druidDataSource.setUrl("jdbc:mysql://localhost:3306/springdata?serverTimezone=UTC");
        druidDataSource.setUsername("root");
        druidDataSource.setPassword("123456");
        return druidDataSource;
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(){
        HibernateJpaVendorAdapter adapter = new HibernateJpaVendorAdapter();
        adapter.setGenerateDdl(true);
        adapter.setShowSql(true);

        LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
        factory.setDataSource(druidDataSource());
        factory.setPackagesToScan("com.xin.pojo");
        factory.setJpaVendorAdapter(adapter);
        return factory;
    }

    @Bean
    public PlatformTransactionManager transactionManager(){
        JpaTransactionManager txManager = new JpaTransactionManager();
        txManager.setEntityManagerFactory(entityManagerFactory().getObject());
        return txManager;
    }

}
```

pojo类：

```java
@Entity
@Table(name = "tb_customer")
@Getter
@Setter
@ToString
@AllArgsConstructor
@NoArgsConstructor
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "cust_id")
    private Long custId;

    @Column(name = "cust_name")
    private String custName;

    @Column(name = "cust_address")
    private String custAddress;
}
```

CustomerRepository:

```java
package com.xin.repository;


import com.xin.pojo.Customer;
import org.springframework.data.repository.PagingAndSortingRepository;
import org.springframework.stereotype.Component;

/**
 * @author: ichigo-xin
 * @create: 2023-07-18 00:47
 * @description:
 **/
//@Component
public interface CustomerRepository extends PagingAndSortingRepository<Customer, Long> {

}
```

application:

```java
package com.xin;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import com.xin.config.SpringDataConfig;
import com.xin.pojo.Customer;
import com.xin.repository.CustomerRepository;

/**
 * @author: ichigo-xin
 * @create: 2023-07-26 01:24
 * @description:
 **/
public class Application {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(SpringDataConfig.class);
        CustomerRepository repository = context.getBean(CustomerRepository.class);
        Customer customer = repository.findById(1L).get();
        System.out.println(customer);
    }
}
```

## 手写模拟Spring Data Jpa

在配置类中使用@EnableJpaRepositories(basePackages = "com.xin.repository")注解，就是开启spring data jpa，现在是可以查询出数据的。

我们把这个注解注释掉，此时运行报错：

**No qualifying bean of type 'com.xin.repository.CustomerRepository' available**

说明没有这个类型的bean，这时候我们就需要去找这个bean没有注入的原因。

### 问题原因

spring读取注册BeanDefinition的流程如下：需要创建BeanDefinition，再根据BeanDefinition创建对象（bean的创建就不继续了），我们在这几个类里面打上断点，调试看看。

![](/imgs/springdatajpa-2.png)

![](/imgs/springdatajpa-1.png)

![](/imgs/springdatajpa-3.png)

![](/imgs/springdatajpa-4.png)

![](/imgs/springdatajpa-5.png)

![](/imgs/springdatajpa-6.png)

在这两个红框的方法内，一个是判断类上面有没有@component注解，另一个是判断这个类是不是接口等

![](/imgs/springdatajpa-7.png)

可以看到如果是一个接口，他就不会注册成beandefinition。

### 自定义扫描器将接口也注册成beandefinition

我们自定义扫描器，不排除掉接口。

```java
package com.xin.my;

import org.springframework.beans.factory.annotation.AnnotatedBeanDefinition;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanDefinitionHolder;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.ClassPathBeanDefinitionScanner;
import org.springframework.context.annotation.ScannedGenericBeanDefinition;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.classreading.MetadataReader;

import java.io.IOException;
import java.util.Set;

/**
 * @author: ichigo-xin
 * @create: 2023-07-26 02:19
 * @description:
 **/
public class MyJpaClassPathBeanDefinitionScanner extends ClassPathBeanDefinitionScanner {

    public MyJpaClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry) {
        super(registry);
    }

    @Override
    protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {
        AnnotationMetadata metadata = beanDefinition.getMetadata();
        return metadata.isInterface();
    }

}
```

再运行代码，就会报错：

**Caused by: org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.xin.repository.CustomerRepository]: Specified class is an interface**

这说明CustomerRepository作为接口也注册成一个beandefinition了。只是不能进行实例化。在spring当中，每一个bean其实就是一个实例化的对象，那么接口肯定不能实例化的，这里我们就需要使用动态代理了。

自定义MyJpaProxy，实现JpaRepository接口。

```java
package com.xin.my;

import org.springframework.data.domain.Example;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.jpa.repository.JpaRepository;

import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;

/**
 * @author: ichigo-xin
 * @create: 2023-07-26 21:08
 * @description:
 **/
public class MyJpaProxy implements JpaRepository {

    EntityManager em;
    Class pojoClass;

    public MyJpaProxy(EntityManager em, Class pojoClass) {
        this.em = em;
        this.pojoClass = pojoClass;
    }

    @Override
    public Optional findById(Object id) {
        System.out.println("自定义JPA统一实现");
        return Optional.of(em.find(pojoClass, id));
    }

    ....

}
```

```java
package com.xin.my;

import javax.persistence.EntityManager;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

/**
 * @author: ichigo-xin
 * @create: 2023-07-26 21:01
 * @description:
 **/
public class MyJpaInvocationHandler implements InvocationHandler {

    EntityManager em;

    Class pojoClass;

    public MyJpaInvocationHandler(EntityManager em, Class pojoClass) {
        this.em = em;
        this.pojoClass = pojoClass;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        MyJpaProxy myJpaProxy = new MyJpaProxy(em, pojoClass);
        Method jpaMethod = myJpaProxy.getClass().getMethod(method.getName(), method.getParameterTypes());
        return jpaMethod.invoke(myJpaProxy, args);
    }
}
```

我们自定义MyJpaFactoryBean，在getObject方法里面使用Proxy.newProxyInstance(）来创建对象。

```java
package com.xin.my;

import com.xin.repository.CustomerRepository;
import org.springframework.beans.factory.FactoryBean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.stereotype.Component;

import javax.persistence.EntityManager;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Proxy;
import java.lang.reflect.Type;

/**
 * @author: ichigo-xin
 * @create: 2023-07-26 20:56
 * @description:
 **/

public class MyJpaFactoryBean implements FactoryBean {

    @Autowired
    LocalContainerEntityManagerFactoryBean entityManagerFactory;

    Class<?> repositoryInterface;

    public MyJpaFactoryBean(Class<?> repositoryInterface) {
        this.repositoryInterface = repositoryInterface;
    }

    @Override
    public Object getObject() throws Exception {
        EntityManager em = entityManagerFactory.createNativeEntityManager(null);

        //获取当前接口的pojo类
        ParameterizedType parameterizedType = (ParameterizedType) repositoryInterface.getGenericInterfaces()[0];
        Type type = parameterizedType.getActualTypeArguments()[0];
        Class clazz = Class.forName(type.getTypeName());

        return Proxy.newProxyInstance(
                CustomerRepository.class.getClassLoader(),
                new Class[]{repositoryInterface},
                new MyJpaInvocationHandler(em, clazz));
    }

    @Override
    public Class<?> getObjectType() {
        return repositoryInterface;
    }
}
```

接下来我们需要在容器加载BeanDefinition的时候将本来应该是实例化接口的步骤，替换成我们上面自定义的实现。

在 Spring 框架中，BeanDefinitionRegistryPostProcessor 是一个重要的扩展点接口，它允许开发者在容器加载 Bean 定义（BeanDefinition）之后，但在实例化 Bean 之前，对 Bean 定义进行自定义修改或添加新的 Bean 定义。

```java
package com.xin.my;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.core.type.filter.AssignableTypeFilter;
import org.springframework.data.repository.Repository;
import org.springframework.stereotype.Component;

/**
 * @author: ichigo-xin
 * @create: 2023-07-26 02:24
 * @description:
 **/
@Component
public class MyJpaBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        MyJpaClassPathBeanDefinitionScanner scanner = new MyJpaClassPathBeanDefinitionScanner(registry);
        scanner.addIncludeFilter(new AssignableTypeFilter(Repository.class));
        scanner.scan("com.xin.repository");
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

    }
}
```

同时给MyJpaClassPathBeanDefinitionScanner重写doscan方法：

```java
package com.xin.my;

import org.springframework.beans.factory.annotation.AnnotatedBeanDefinition;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanDefinitionHolder;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.ClassPathBeanDefinitionScanner;
import org.springframework.context.annotation.ScannedGenericBeanDefinition;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.core.type.classreading.MetadataReader;

import java.io.IOException;
import java.util.Set;

/**
 * @author: ichigo-xin
 * @create: 2023-07-26 02:19
 * @description:
 **/
public class MyJpaClassPathBeanDefinitionScanner extends ClassPathBeanDefinitionScanner {

    public MyJpaClassPathBeanDefinitionScanner(BeanDefinitionRegistry registry) {
        super(registry);
    }

    @Override
    protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {
        AnnotationMetadata metadata = beanDefinition.getMetadata();
        return metadata.isInterface();
    }

    @Override
    protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
        Set<BeanDefinitionHolder> beanDefinitionHolders = super.doScan(basePackages);
        for (BeanDefinitionHolder beanDefinitionHolder : beanDefinitionHolders) {
            ScannedGenericBeanDefinition beanDefinition = (ScannedGenericBeanDefinition) beanDefinitionHolder.getBeanDefinition();
            String beanClassName = beanDefinition.getBeanClassName();
            beanDefinition.getConstructorArgumentValues().addGenericArgumentValue(beanClassName);
            // 修改beanDefinition
            beanDefinition.setBeanClass(MyJpaFactoryBean.class);
        }
        return beanDefinitionHolders;
    }

}
```

我们在doscan方法中对beanDefinition的BeanClass进行替换。

运行代码，成功查询出来数据了。
![](/imgs/springdatajpa-8.png)
同时我们也可以看下代理对象
