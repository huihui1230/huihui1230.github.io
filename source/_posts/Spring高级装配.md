---
title: Spring高级装配
date: 2018-07-13 19:17:39
tags: Spring
categories: 
- 后端
- Spring
copyright: true
---
## 环境与profile  
  
多个相同的bean，但是其实现方式不同，在代码实际运行中，只会选择其中的一种。`@Profile`注解用于给bean标注，当该profile被激活时，才装配该profile对应的bean。`@ActiveProfiles`注解用于标注被激活的profile，可以设置多个。  
  
<!--more-->
  
### 在Java中配置profile bean  
  
基础对象的类：  

People.java  

```java
package com.myapp;

public interface People {

    void speak();
}
```
  
Man.java  
  
```java
package com.myapp;

public class Man implements People {

    @Override
    public void speak() {
        System.out.println("i am man");
    }
}
```
  
Woman.java  
  
```java
package com.myapp;

public class Woman implements People {

    @Override
    public void speak() {
        System.out.println("i am woman");
    }
}
```
  
LittlePeople.java  
  
```java
package com.myapp;

public interface LittlePeople {

    void speak();
}
```
  
Boy.java  
  
```java
package com.myapp;

public class Boy implements LittlePeople {

    @Override
    public void speak() {
        System.out.println("i am a boy");
    }
}
	```
  
Girl.java  
  
```java
package com.myapp;

public class Girl implements LittlePeople {

    @Override
    public void speak() {
        System.out.println("i am a girl");
    }
}
```
  
配置的类：  
  
ManConfig.java  
  
```java
package com.myapp;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
@Profile("man")
public class ManConfig {

    @Bean
    public People createMan() {
        return new Man();
    }

    @Bean
    public LittlePeople createBoy() {
        return new Boy();
    }
}
```
  
WomanConfig.java  
  
```java
package com.myapp;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

@Configuration
@Profile("woman")
public class WomanConfig {

    @Bean
    public People createWoman() {
        return new Woman();
    }

    @Bean
    public LittlePeople createGirl() {
        return new Girl();
    }
}
```
  
测试类：  
  
PeopleConfigTest.java  
  
```java
package profiles;

import com.myapp.*;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {ManConfig.class, WomanConfig.class})
@ActiveProfiles("man")
public class PeopleConfigTest {

    @Autowired
    private People people;

    @Autowired
    private LittlePeople littlePeople;

    @Test
    public void test() {
        people.speak();
        littlePeople.speak();
    }
}
```
  
运行结果：  
  
```
i am man
i am a boy
```
  
### 在XML中配置profile bean  
  
配置文件：  
  
profile.xml  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <beans profile="man">
        <bean class="com.myapp.Man"></bean>
        <bean class="com.myapp.Boy"></bean>
    </beans>

    <beans profile="woman">
        <bean class="com.myapp.Woman"></bean>
        <bean class="com.myapp.Girl"></bean>
    </beans>
</beans>
```
  
测试类：  
  
PeopleConfigTest.java  
  
```java
package profiles;

import com.myapp.*;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:profile.xml")
@ActiveProfiles("man")
public class PeopleConfigTest {

    @Autowired
    private People people;

    @Autowired
    private LittlePeople littlePeople;

    @Test
    public void test() {
        people.speak();
        littlePeople.speak();
    }
}
```
  
运行结果：  
  
```
i am man
i am a boy
```
  
## 条件化的bean  
  
`@Conditional`注解当某个条件存在时才会创建bean。  
  
基础对象类：  
  
Animal.java  
  
```java
package com.myapp;

public class Animal {

    public void play() {
        System.out.println("It's fun");
    }
}
```
  
条件类：  
  
MagicExistsCondition.java  
  
```java
package com.myapp;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class MagicExistsCondition implements Condition {
    
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        Environment environment = conditionContext.getEnvironment();
        return environment.containsProperty("magic");
    }
}
```
  
类`MagicExistsCondition`继承接口`Condition`，当`matches`方法返回`true`时，创建bean。
  
配置类：  
  
AnimalConfig.java  
  
```java
package com.myapp;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AnimalConfig {

    @Bean
    @Conditional(MagicExistsCondition.class)
    public Animal createAnimal() {
        return new Animal();
    }
}
```
  
测试类：  
  
AnimalConfigTest.java  
  
```java
package bean;

import com.myapp.Animal;
import com.myapp.AnimalConfig;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AnimalConfig.class)
public class AnimalConfigTest {

    @Autowired
    private Animal animal;

    @Test
    public void test() {
        animal.play();
    }
}
```
  
运行报错，`Animal`没有被装配进来。将`matches`方法的返回值设置为true。  
  
```java
package com.myapp;

import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class MagicExistsCondition implements Condition {

    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        return true;
    }
}
```
  
重新运行测试类，运行结果：  
  
```
It's fun
```
  
## 处理自动装配的歧义性  
  
### 标示首选的bean  
  
当多个相同的bean存在时，spring并不知道要装配哪一个。`@Primary`设置首先被装配的bean，当spring装备bean时，会选择有该注释的bean。  
  
基础对象类：  
  
People.java  
Man.java  
Woman.java  
LittlePeople.java  
Boy.java  
Girl.java  
  
配置类：  
  
ManConfig.java  
  
```java
package com.myapp;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

@Configuration
public class ManConfig {

    @Bean
    @Primary
    public People createMan() {
        return new Man();
    }

    @Bean
    public LittlePeople createBoy() {
        return new Boy();
    }
}
```
  
WomanConfig.java  
  
```java
package com.myapp;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.Profile;

@Configuration
public class WomanConfig {

    @Bean
    public People createWoman() {
        return new Woman();
    }

    @Bean
    @Primary
    public LittlePeople createGirl() {
        return new Girl();
    }
}
```
  
测试类：  
  
PeopleConfigTest.java  
  
```java
package profiles;

import com.myapp.*;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {ManConfig.class, WomanConfig.class})
@ActiveProfiles("man")
public class PeopleConfigTest {

    @Autowired
    private People people;

    @Autowired
    private LittlePeople littlePeople;

    @Test
    public void test() {
        people.speak();
        littlePeople.speak();
    }
}
```
  
运行结果：  
  
```
i am man
i am a girl
```
  
XML中的配置：  
  
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <beans>
        <bean class="com.myapp.Man" primary="true"></bean>
        <bean class="com.myapp.Boy"></bean>
    </beans>

    <beans>
        <bean class="com.myapp.Woman"></bean>
        <bean class="com.myapp.Girl" primary="true"></bean>
    </beans>
</beans>
```
  
### 限定自动装配的bean  
  
`@Qualifier`注解是限定符，指定想要注入进去的是哪个bean。  
  
基础对象类：  
  
People.java  
Man.java  
Woman.java  
LittlePeople.java  
Boy.java  
Girl.java  
  
配置类：  
  
ManConfig.java  
  
```java
package com.myapp;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ManConfig {

    @Bean
    @Qualifier("man")
    public People createMan() {
        return new Man();
    }

    @Bean
    public LittlePeople createBoy() {
        return new Boy();
    }
}
```
  
WomanConfig.java  
  
```java
package com.myapp;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class WomanConfig {

    @Bean
    public People createWoman() {
        return new Woman();
    }

    @Bean
    @Qualifier("girl")
    public LittlePeople createGirl() {
        return new Girl();
    }
}
```
  
测试类：  
  
```java
package profiles;

import com.myapp.*;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {ManConfig.class, WomanConfig.class})
public class PeopleConfigTest {

    @Autowired
    @Qualifier("man")
    private People people;

    @Autowired
    @Qualifier("girl")
    private LittlePeople littlePeople;

    @Test
    public void test() {
        people.speak();
        littlePeople.speak();
    }
}
```
  
运行结果：  
  
```
i am man
i am a girl
```

在XML中的配置只需指定bean的id即可。  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <beans>
        <bean class="com.myapp.Man" id="man"></bean>
        <bean class="com.myapp.Boy" id="boy"></bean>
    </beans>

    <beans>
        <bean class="com.myapp.Woman" id="woman"></bean>
        <bean class="com.myapp.Girl" id="girl"></bean>
    </beans>
</beans>
```
  
## bean的作用域  
  
Spring的作用域包括：  

* 单例：整个应用中只会创建一个bean实例。  
* 原型：每次注入都会创建一个新的bean实例。  
* 会话：web应用的每个会话创建一个bean实例。  
* 请求：web应用的每个请求创建一个bean实例。  
  
创建的bean默认是单例模式。  
  
基础对象类：  
  
Animal.java  
  
```java
package com.myapp;

public class Animal {

    private int i = 0;

    public void play() {
        i++;
        System.out.println(i);
    }
}
```
  
配置类：  
  
AnimalConfig.java  
  
```java
package com.myapp;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AnimalConfig {

    @Bean
    public Animal createAnimal() {
        return new Animal();
    }
}
```
  
测试类：  
  
```java
package bean;

import com.myapp.Animal;
import com.myapp.AnimalConfig;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AnimalConfig.class)
public class AnimalConfigTest {

    @Autowired
    private Animal animal1;

    @Autowired
    private Animal animal2;

    @Test
    public void test() {
        animal1.play();
        animal2.play();
    }
}
```
  
输出结果：  
  
```
1
2
```
  
说明在两次注入Animal时，注入的是同一个bean。  
  
原型模式在标注bean的位置标记Scope。  
  
配置类：  
  
AnimalConfig.java  
  
```java
package com.myapp;

import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

@Configuration
public class AnimalConfig {

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public Animal createAnimal() {
        return new Animal();
    }
}
```
  
测试结果：  
  
```
1
1
```
  
说明两次注入的Animal是不同的bean。  
  
XML中的配置：  
  
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="animal" class="com.myapp.Animal" scope="prototype"></bean>
</beans>
```
  
会话和请求作用域使用`WebApplicationContext`中的`SCOPE_SESSION`和`SCOPE_REQUEST`，bean以作用域代理的方式进行注入，代理可以是接口或者类，使用`proxyMode = ScopedProxyMode.INTERFACES`或`proxyMode = ScopedProxyMode.TARGET_CLASS`指定为代理接口或代理类。  
  
```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_REQUEST,proxyMode = ScopedProxyMode.TARGET_CLASS)
public Animal createAnimal() {
    return new Animal();
}
```
  
在XML配置中，使用`<aop:scoped-proxy>`告诉Spring创建一个作用域代理，默认是类代理，将`proxy-target-class`属性设置为false生成接口代理。  
  
## 运行时值注入  
  
之前注入`CompactDisc`时，在运行之前就已经在代码中设置了bean的属性的值，这属于硬编码。实际运用中，需要灵活的指定bean的属性的值。  
  
```xml
<bean id="compactDisc" class="soundsystem.SgtPeppers">
    <constructor-arg value="title"></constructor-arg>
    <constructor-arg value="artist"></constructor-arg>
    <constructor-arg>
        <list>
            <value>s1</value>
            <value>s2</value>
            <value>s3</value>
            <value>s4</value>
        </list>
    </constructor-arg>
</bean>
```
  
Spring提供了两种运行时值注入的方式：  
* 属性占位符
* Spring表达式语言
  
### 注入外部的值  
  
基础对象类：  
  
Animal.java  
  
```java
package com.myapp;

public class Animal {

    private String name;

    public Animal(String name) {
        this.name = name;
    }

    public void play() {
        System.out.println(name);
    }
}
```
  
值配置：  
  
animal.properties  
  
```
name=Peppa
```
  
配置类：  
  
AnimalConfig.java  
  
```java
package com.myapp;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.PropertySource;
import org.springframework.core.env.Environment;

@Configuration
@PropertySource("classpath:animal.properties")
public class AnimalConfig {

    @Autowired
    Environment environment;

    @Bean
    public Animal createAnimal() {
        return new Animal(environment.getProperty("name"));
    }
}
```
  
测试类：  
  
AnimalConfigTest.java  
  
```java
package bean;

import com.myapp.Animal;
import com.myapp.AnimalConfig;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AnimalConfig.class)
public class AnimalConfigTest {

    @Autowired
    private Animal animal;

    @Test
    public void test() {
        animal.play();
    }
}
```
  
测试结果:  
  
```
Peppa
```
  
### 深入学习Spring的Environment
  
* getProperty()  
  
1. String getProperty(String key)
2. String getProperty(String key, String defaultValue)
3. T getProperty(String key, Class<T> type)
4. T getProperty(String key, Class<T> type, T defaultValue)
  
第一种直接返回属性对应的值；第二种当没有设置属性的值时，使用传入的默认值，返回第二个参数的值；第三种可以设置返回值的类型，比如“3”可以使String，也可以是Integer；第四种当没有设置属性的值时，返回默认值。  
  
* getRequiired()
  
属性必须被定义，否则会报IllegalStateException。  
  
* containsProperty()
  
判断某个属性是否存在，返回Boolean值。  
  
* getPropertyAsClass()
  
将属性解析为类。  
  
* getActiveProfiles()
  
返回激活profile名称的数组  
  
* getDefaultProfiles()
  
返回默认profile名称的数组  
  
* acceptsProfiles(String ... profiles)
  
如果environment支持给定的profile，则返回true。  
  
### 解析属性占位符
  
这个地方没搞懂！！！  
  
### 使用Spring表达式语言进行装配
  
SpELl表达式好像很牛逼的样子。  
  

