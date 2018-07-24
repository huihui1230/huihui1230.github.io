---
title: 面向切面的Spring
date: 2018-07-20 10:11:50
tags: Spring
categories:
- 后端
- Spring
---
## 面向切面编程  
  
### AOP术语  
  
![](https://i.imgur.com/XFFheUW.png)
  
<!--more-->
  
* 通知
  
切面的工作被称为通知，通知定义了切面是什么以及何时使用。通知有5种类型：  
  
1. 前置通知：在目标方式被调用之前调用通知功能。
2. 后置通知：在目标方法完成之后调用通知，不关心方法的输出是什么。
3. 返回通知：在目标方法成功执行之后调用通知，关心方法输出。
4. 异常通知：在目标方法抛出异常后调用通知。
5. 环绕通知：通知包裹了被通知的方法，在被通知方法调用之前和调用之后执行自定义的行为。
  
* 连接点
  
连接点是在应用执行过程中能够插入切面的一个点。  
  
* 切点
  
切点定义了切面在何处使用。切点匹配通知所要织入的一个或多个连接点，可以使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。  
  
* 切面
  
切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——它是什么，在何时和何处完成其功能。  
  
* 引入
  
引入是向现有的类添加新方法和属性。  
  
* 织入
  
织入是把切面应用到目标对象并创建新的代理对象的过程。织入点：  
  
1. 编译期：切面在目标类编译时被织入。
2. 类加载期：切面在目标类加载到JVM时被织入。
3. 运行期：切面在应用运行的某个时刻被织入。
  
### Spring AOP框架关键知识  
  
* Spring通知是Java编写的
* Spring在运行时通知对象
* Spring只支持方法级别的连接点
  
## 通过切点来选择连接点  
  
在Spring AOP中，使用AspectJ的切点表达式来定义切点。Spring AOP 支持的AspectJ切点指示器：  
  
| AspectJ指示器 |            描述             |
| ------------ | ----------------------------|
| arg() | 限制连接点匹配参数为指定类型的执行方法 |
| @args() | 限制连接点匹配参数由指定注解标注的执行方法 |
| execution() | 用于匹配连接点的执行方法 |
| this() | 限制连接点匹配AOP代理的bean引用为指定类型的类 |
| target | 限制连接点匹配目标对象为指定类型的类 |
| @target | 限制连接点匹配特定的执行对象，这些对象对应的类要具有指定类型的注解 |
| within() | 限制连接点匹配指定的类型 |
| @within() | 限制连接点匹配指定注解所标注的类型 |
| @annotation | 限定匹配带有指定注解的连接点 |
  
Spring使用AspectJ注解来声明通知方法:  
  
| 注解 | 通知 |
| ---- | --- |
| @After | 通知方法会在目标方法返回或抛出异常后调用 |
| @AfterReturning | 通知方法会在目标方法返回后调用 |
| @AfterThrowing | 通知方法会在目标方法抛出异常后调用 |
| @Around | 通知方法会将目标方法封装起来 |
| @Before | 通知方法会在目标方法调用之前执行 |
  
### 创建前置通知和后置通知：  
  
基础对象类：  
  
Performance.java  
  
```java
package concert;

public interface Performance {
    public void perform();
}
```
  
Leader.java  
  
```java
package concert;

public interface Performance {
    public void perform();
}
```
  
定义切面:  
  
引入包aspectjweaver  
  
Audience.java  
  
```java
package concert;

import org.aspectj.lang.annotation.*;

@Aspect
public class Audience {

    @Pointcut("execution(* concert.Performance.perform(..))")
    public void performance() {

    }

    @Before("performance()")
    public void silenceCellPhones() {
        System.out.println("Silencing cell phones");
    }

    @Before("performance()")
    public void takeSeats() {
        System.out.println("Taking seats");
    }

    @AfterReturning("performance()")
    public void applause() {
        System.out.println("CLAP CLAP CLAP!!!");
    }

    @AfterThrowing("performance()")
    public void demandRefund() {
        System.out.println("Demanding a refund");
    }
}
```
  
配置类：  
  
LeaderConfig.java  
  
```java
package concert;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@EnableAspectJAutoProxy
public class LeaderConfig {

    @Bean
    public Performance createLeader() {
        return new Leader();
    }

    @Bean
    public Audience createAudience() {
        return new Audience();
    }
}
```
  
测试类：  
  
LeaderConfigTest.java  
  
```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import concert.LeaderConfig;
import concert.Performance;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = LeaderConfig.class)
public class concert {

    @Autowired
    private Performance performance;

    @Test
    public void test() {
        performance.perform();
    }
}
```
  
运行结果：  
  
```
Silencing cell phones
Taking seats
perform
CLAP CLAP CLAP!!!
```
  
配置文件：  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="concert.Leader" id="leader" />

    <aop:aspectj-autoproxy proxy-target-class="true"/>

    <bean class="concert.Audience" id="audience" />
</beans>
```
  
### 创建环绕通知  
  
基础对象类：  
  
Performance.java  
Leader.java
  
创建切面：  
  
Audience.java  
  
```java
package concert;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;

@Aspect
public class Audience {

    @Pointcut("execution(* concert.Performance.perform(..))")
    public void performance() {

    }

    @Around("performance()")
    public void watchPerformance(ProceedingJoinPoint joinPoint) {
        try {
            System.out.println("Silencing cell phones");
            System.out.println("Taking seats");
            joinPoint.proceed();
            System.out.println("CLAP CLAP CLAP!!!");
        } catch (Throwable e) {
            System.out.println("Demanding a refund");
        }
    }
}
```
  
配置类：  
  
LeaderConfig.java  
  
测试类：  
  
LeaderConfigTest.java  
  
测试结果：  
  
```
Silencing cell phones
Taking seats
perform
CLAP CLAP CLAP!!!
```
  
### 处理通知中的参数  
  
基础对象类：  
  
CompactDisc.java  
  
```java
package concert;

import java.util.List;

public class CompactDisc {

    private List<String> playList;

    public CompactDisc(List<String> playList) {
        this.playList = playList;
    }

    public void playTrack(int n) {
        System.out.println(playList.get(n - 1));
    }
}
```
  
切面：  
  
TrackCounter.java  
  
```java
package concert;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

import java.util.HashMap;
import java.util.Map;

@Aspect
public class TrackCounter {

    private Map<Integer, Integer> trackCounts = new HashMap<>();

    @Pointcut("execution(* concert.CompactDisc.playTrack(int)) && args(trackNumber)")
    public void trackPlayed(int trackNumber) {}

    @Before("trackPlayed(trackNumber)")
    public void countTrack(int trackNumber) {
        int currentCount = getPlayCount(trackNumber);
        trackCounts.put(trackNumber, currentCount + 1);
    }

    public int getPlayCount(int trackNumber) {
        return trackCounts.containsKey(trackNumber) ? trackCounts.get(trackNumber) : 0;
    }

    public void print() {
        System.out.println(trackCounts);
    }
}
```
  
配置类：  
  
TrackCounterConfig.java  
  
```java
package concert;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

import java.util.ArrayList;
import java.util.List;

@Configuration
@EnableAspectJAutoProxy
public class TrackCounterConfig {

    @Bean
    public CompactDisc createCompactDisc() {
        List<String> tracks = new ArrayList<>();
        tracks.add("云烟成雨");
        tracks.add("借我");
        tracks.add("可能否");

        return new CompactDisc(tracks);
    }

    @Bean
    public TrackCounter createTrackCounter() {
        return new TrackCounter();
    }
}
```
  
测试类：  
  
TrackCounterConfigTest.java  
  
```java
package concert;

import org.aspectj.lang.annotation.Aspect;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.Random;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = TrackCounterConfig.class)
public class TrackCounterTest {

    @Autowired
    private CompactDisc compactDisc;

    @Autowired
    private TrackCounter trackCounter;

    @Test
    public void test() {
        Random random = new Random();
        for (int i = 0; i < 10; i++) {
            compactDisc.playTrack(random.nextInt(3) % 3 + 1);
        }

        trackCounter.print();
    }
}
```
  
测试结果：  
  
```
云烟成雨
可能否
借我
借我
可能否
可能否
可能否
云烟成雨
可能否
借我
{1=2, 2=3, 3=5}
```
  
### 通过注解引入新功能  
  
为类添加方法。  
  
基础对象类：  
  
Performance.java  
Leader.java  
Encoreable.java  
  
```java
package concert;

public interface Encoreable {
    void performEncore();
}
```
  
DefaultEncoreable.java  
  
```java
package concert;

public class DefaultEncoreable implements Encoreable {

    @Override
    public void performEncore() {
        System.out.println("encoreable");
    }
}
```
  
代理类：  
  

EncoreableIntroducer.java  
  
```java
package concert;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.DeclareParents;

@Aspect
public class EncoreableIntroducer {

    @DeclareParents(value = "concert.Performance+", defaultImpl = DefaultEncoreable.class)
    public static Encoreable encoreable;
}
```
  
配置类：  
  
EncoreableConfig.java  
  
```java
package concert;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@EnableAspectJAutoProxy
public class EncoreableConfig {

    @Bean
    public Performance createLeader() {
        return new Leader();
    }

    @Bean
    public EncoreableIntroducer createIntroducer() {
        return new EncoreableIntroducer();
    }
}
```
  
测试类：  
  
EncoreableConfigTest.java  
  
```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import concert.*;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = EncoreableConfig.class)
public class EncoreableConfigTest {

    @Autowired
    private Performance performance;

    @Test
    public void test() {
        performance.perform();

        Encoreable encoreable = (Encoreable) performance;
        encoreable.performEncore();
    }
}
```
  
运行结果：  
  
```
perform
encoreable
```
  
## 在XML中是声明切面  
  
Spring AOP在XML中的元素：  
  
| AOP配置元素 | 用途 |
| ---------- | ---- |
| &lt;aop:advisor&gt; | 定义AOP通知器 |
| &lt;aop:after&gt; | 定义AOP后置通知（不管被通知的方法是否执行成功） |
| &lt;aop:after-returning&gt; | 定义AOP返回通知 |
| &lt;aop:after-throwing&gt; | 定义AOP异常通知 |
| &lt;aop:around&gt; | 定义AOP环绕通知 |
| &lt;aop:aspect&gt; | 定义切面 |
| &lt;aop:aspectj-autoproxy&gt; | 启用@AspectJ注解驱动的切面 |
| &lt;aop:before&gt; | 定义AOP前置通知 |
| &lt;aop:config&gt; | 顶层的AOP配置元素。大多数的&lt;aop:*&gt;元素必须包含在&lt;aop:config&gt;中 |
| &lt;aop:declare-parents&gt; | 以透明的方式为被通知的对象引入额外的接口 |
| &lt;aop:pointcut&gt; | 定义切点 |
  

### 声明前置通知和后置通知  
  
基础对象类：  
  
Performance.java  
Leader.java  
  
切面：  
  
Audience.java  
  
```java
package concert;

public class Audience {

    public void silenceCellPhones() {
        System.out.println("Silencing cell phones");
    }

    public void takeSeats() {
        System.out.println("Taking seats");
    }

    public void applause() {
        System.out.println("CLAP CLAP CLAP!!!");
    }

    public void demandRefund() {
        System.out.println("Demanding a refund");
    }
}
```
  
配置：  
  
aspectj.xml  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="concert.Leader" id="leader" />
    <bean class="concert.Audience" id="audience" />

    <aop:config>
        <aop:aspect ref="audience">
            <aop:pointcut id="performance" expression="execution(* concert.Performance.perform(..))" />
            <aop:before method="silenceCellPhones" pointcut-ref="performance" />
            <aop:before method="takeSeats" pointcut-ref="performance" />
            <aop:after-returning method="applause" pointcut-ref="performance" />
            <aop:after-throwing method="demandRefund" pointcut-ref="performance" />
        </aop:aspect>
    </aop:config>
</beans>
```
  
测试类：  
  
LeaderConfigTest.java  
  
```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import concert.Performance;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:aspectj.xml")
public class concert {

    @Autowired
    private Performance performance;

    @Test
    public void test() {
        performance.perform();
    }
}
```
  
运行结果：  
  
```
Silencing cell phones
Taking seats
perform
CLAP CLAP CLAP!!!
```
  
### 声明环绕通知  
  
基础对象类：  
  
Performance.java  
Leader.java  
  
切面：  
  
Audience.java  
  
```java
package concert;

public class Audience {

    public void watchPerformance(ProceedingJoinPoint joinPoint) {
        try {
            System.out.println("Silencing cell phones");
            System.out.println("Taking seats");
            joinPoint.proceed();
            System.out.println("CLAP CLAP CLAP!!!");
        } catch (Throwable e) {
            System.out.println("Demanding a refund");
        }
    }
}
```
  
配置：  
  
aspectj.xml  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="concert.Leader" id="leader" />
    <bean class="concert.Audience" id="audience" />

    <aop:config>
        <aop:aspect ref="audience">
            <aop:pointcut id="performance" expression="execution(* concert.Performance.perform(..))" />
            <aop:around method="watchPerformance" pointcut-ref="performance" />
        </aop:aspect>
    </aop:config>
</beans>
```
  
测试类：  
  
LeaderConfigTest.java  
  
```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import concert.Performance;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:aspectj.xml")
public class concert {

    @Autowired
    private Performance performance;

    @Test
    public void test() {
        performance.perform();
    }
}
```
  
运行结果：  
  
```
Silencing cell phones
Taking seats
perform
CLAP CLAP CLAP!!!
```
  
### 为通知传递参数  
  
基础对象类：  
  
CompactDisc.java  
  
```java
package concert;

import java.util.List;

public class CompactDisc {

    private List<String> playList;

    public CompactDisc(List<String> playList) {
        this.playList = playList;
    }

    public void playTrack(int n) {
        System.out.println(playList.get(n - 1));
    }
}
```
  
切面：  
  
TrackCounter.java  
  
```java
package concert;

import java.util.HashMap;
import java.util.Map;

public class TrackCounter {

    private Map<Integer, Integer> trackCounts = new HashMap<>();

    public void countTrack(int trackNumber) {
        int currentCount = getPlayCount(trackNumber);
        trackCounts.put(trackNumber, currentCount + 1);
    }

    public int getPlayCount(int trackNumber) {
        return trackCounts.containsKey(trackNumber) ? trackCounts.get(trackNumber) : 0;
    }

    public void print() {
        System.out.println(trackCounts);
    }
}
```
  
配置：  
  
aspectj.xml  
  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean class="concert.TrackCounter" id="trackCounter"/>
    <bean class="concert.CompactDisc" id="compactDisc">
        <constructor-arg>
            <list>
                <value>凉凉</value>
                <value>借我</value>
                <value>佛系少女</value>
            </list>
        </constructor-arg>
    </bean>

    <aop:config>
        <aop:aspect ref="trackCounter">
            <aop:pointcut id="trackPlayed"
                          expression="execution(* concert.CompactDisc.playTrack(int)) and args(trackNumber)"/>
            <aop:before method="countTrack" pointcut-ref="trackPlayed"/>
        </aop:aspect>
    </aop:config>
</beans>
```
  
测试类：  
  
TrackCounterTest.java  
  
```java
package concert;

import org.aspectj.lang.annotation.Aspect;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.Random;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:aspectj.xml")
public class TrackCounterTest {

    @Autowired
    private CompactDisc compactDisc;

    @Autowired
    private TrackCounter trackCounter;

    @Test
    public void test() {
        Random random = new Random();
        for (int i = 0; i < 10; i++) {
            compactDisc.playTrack(random.nextInt(3) % 3 + 1);
        }

        trackCounter.print();
    }
}
```
  
运行结果：  
  
```
佛系少女
凉凉
借我
借我
借我
借我
佛系少女
佛系少女
佛系少女
凉凉
{1=2, 2=4, 3=4}
```
  
### 通过切面引入新的功能  
  

