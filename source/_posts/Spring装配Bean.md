---
title: Spring装配Bean
date: 2018-07-10 19:40:26
tags: Spring
categories:
- 后端
- Spring  
---
## 自动化装配bean（@Component、@ComponentScan和@Autowired) ## 
    
	package soundsystem;  
	
	public interface CompactDisc {  
	    void play();  
	}  
  
<!--more-->   

&emsp;
 
	package soundsystem;  
	  
	import org.springframework.stereotype.Component;  
	  
	@Component  
	public class SgtPeppers implements CompactDisc {  
	  
		private String title = "title";  
	    private String artist = "artist";   
	  
		@Override  
	    public void play() {  
	        System.out.println("Playing " + title + " by " + artist);  
	    }  
	}  
 
@Component表明类CompactDisc是一个组件类，默认ID为sgtPeppers。也可以指定bean的ID，`@Component("LonelyHeartsClub")`将ID指定为lonelyHeartsClub。@Named也可以来设置bean的ID，`@Named("lonelyHeartsClub")`，两者差别不大。
    
	package soundsystem;  
	
	import org.springframework.context.annotation.ComponentScan;  
	import org.springframework.context.annotation.Configuration;  
	
	@Configuration  
	@ComponentScan  
	public class CDPlayerConfig {  
	  
	}  
 
  
@ComponentScan启用组件扫描，默认扫描与配置类相同的包。可以通过设置@ComponentScan的value值来设置组件扫描的包，`@ComponentScan("soundsystem")`。可以设置多个包，`ComponentScan(basePackages={"soundsystem", "package1"})`。也可以设置扫描的类，`ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class})`  
  
测试，引入Spring和Junit的jar包  。
  
![](https://i.imgur.com/6GNBpjZ.png)  
![](https://i.imgur.com/SdUkQXL.png)  
  
运行提示找不到hamcrest包，降低Junit版本或者加入hamcrest包，由于降低Junit版本之后和spring适配有问题，只好加入hamrest包了。  
  
![](https://i.imgur.com/vArScZC.png)
  
	package soundsystem;
	
	import org.junit.Test;
	import org.junit.Assert;
	import org.junit.runner.RunWith;  
	
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.test.context.ContextConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
	
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(classes = CDPlayerConfig.class)
	public class CDPlayerTest {
	
	    @Autowired
	    private CompactDisc compactDisc;
	
	    @Test
	    public void cdShouldNotBeNull() {
	        Assert.assertNotNull(compactDisc);
	        compactDisc.play();
	    }
	}     
  
@Autowired可以将扫描到的bean装配到注解的地方，可以在类中的属性上注解，也可以在方法上注解。@Autowired的required属性默认为true，当没有bean时，会报错。将required属性设置为false，`@Autowired(required=false)`，当没有匹配的bean，这个bean就会处于未装配状态，不会报错。@Inject和@Autowired作用类似。  
	    
	private CompactDisc compactDisc;  
	  
	@Autowired
	public void setCompactDisc(CompactDisc compactDisc) {
	    this.compactDisc = compactDisc;
	}  
  
 
## 通过Java代码装配bean(JavaConfig)  
  
要装配的类：
  
	package soundsystem;
	
	public class SgtPeppers {
	
	    private String title = "title";
	    private String artist = "artist";
	
	
	    @Override
	    public void play() {
	        System.out.println("Playing " + title + " by " + artist);
	    }
	}   
  
&emsp; 
    
	package soundsystem;
	
	public class CDPlayer {
	
	    private CompactDisc compactDisc;
	
	    public CDPlayer(CompactDisc compactDisc) {
	        this.compactDisc = compactDisc;
	    }
	
	    public void play() {
	        compactDisc.play();
	    }
	}
   
配置类：
  
 
	package soundsystem;
	
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	
	@Configuration
	public class CDPlayerConfig {
	    @Bean
	    public CompactDisc sgtPeppers() {
	        return new SgtPeppers();
	    }
	  
	    @Bean
	    public CDPlayer cdPlayer() {
	        return new CDPlayer(sgtPeppers());
	    }
	}  
  
  
测试类：  
  
	package soundsystem;
	
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.test.context.ContextConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
	
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration(classes = CDPlayerConfig.class)
	public class CDPlayerTest {
	
	    @Autowired
	    private CDPlayer cdPlayer;
	
	    @Test
	    public void cdShouldNotBeNull() {
	        cdPlayer.play();
	    }
	}  
  
  
@Bean装配bean，配置类中，也可以用如下方式进行装配。在装配CDPlayer过程中，可以由sgtPeppers()或CompactDisc compactDisc找到装配的CompactDisc的bean，并注入到cdPlayer()方法中。

## 通过XML装配bean ##  
  
![](https://i.imgur.com/kqJi0Ra.png)  
  
xml配置：  
  
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:aop="http://www.springframework.org/schema/aop"
	       xmlns:tx="http://www.springframework.org/schema/tx"
	       xmlns:jee="http://www.springframework.org/schema/jee"
	       xmlns:context="http://www.springframework.org/schema/context"
	       xsi:schemaLocation="
	     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
	     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
	     http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
	     http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-3.0.xsd
	     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
	     ">
	
	    <bean id="compactDisc" class="soundsystem.SgtPeppers"></bean>
	
	    <bean id="cdPlayer" class="soundsystem.CDPlayer">
	        <constructor-arg ref="compactDisc"></constructor-arg>
	    </bean>
	</beans>
  
<constructor-arg>元素向CDPlayer中注入CompactDisc的bean，也可以使用c-命名空间。  
  
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:c="http://www.springframework.org/schema/c"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:aop="http://www.springframework.org/schema/aop"
	       xmlns:tx="http://www.springframework.org/schema/tx"
	       xmlns:jee="http://www.springframework.org/schema/jee"
	       xmlns:context="http://www.springframework.org/schema/context"
	       xsi:schemaLocation="
	     http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
	     http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd
	     http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.3.xsd
	     http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-3.0.xsd
	     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.3.xsd
	     ">
	
	    <bean id="compactDisc" class="soundsystem.SgtPeppers"></bean>
	
	    <bean id="cdPlayer" class="soundsystem.CDPlayer" c:_0-ref="compactDisc"></bean>
	</beans>
   
开始测试，在测试类中引用xml文件。使用ClassPathXmlApplicationContext和FileSystemXmlApplicationContext无法载入spring容器，原因未知。  
  
	package soundsystem;
	
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.test.context.ContextConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration({"/soundsystem/bean.xml"})
	public class CDPlayerTest {
	    @Autowired
	    private CDPlayer cdPlayer;
	
	    @Test
	    public void cdShouldNotBeNull() {
	        cdPlayer.play();
	    }
	}
  
当类SgtPeppers的两个属性需要传入值时，即：  
  
	package soundsystem;

	public class SgtPeppers implements CompactDisc {
	
	    private String title;
	    private String artist;
	
	    public SgtPeppers(String title, String artist) {
	        this.title = title;
	        this.artist = artist;
	    }
	
	    @Override
	    public void play() {
	        System.out.println("Playing " + title + " by " + artist);
	    }
	}
  
配置文件中装配SgtPeppers时需要传入参数的值：  
  
    <bean id="compactDisc" class="soundsystem.SgtPeppers">
        <constructor-arg value="title" />
        <constructor-arg value="artist" />
    </bean>  
  
这里也可以用c-命名空间：  
  
	<bean id="compactDisc" class="soundsystem.SgtPeppers" c:title="title" c:artist="artist"></bean>  
  
或：  
  
	<bean id="compactDisc" class="soundsystem.SgtPeppers" c:_0="title" c:_1="artist"></bean>
  
当类SgtPeppers的属性存在集合时，即：  
  
	package soundsystem;
	
	import java.util.List;

	public class SgtPeppers implements CompactDisc {
	
	    private String title;
	    private String artist;
	    private List<String> tracks;
	
	    public SgtPeppers(String title, String artist, List<String> tracks) {
	        this.title = title;
	        this.artist = artist;
	        this.tracks = tracks;
	    }
	
	    @Override
	    public void play() {
	        System.out.println("Playing " + title + " by " + artist);
	        tracks.stream().forEach(System.out::println);
	    }
	}
  
配置文件中需要传入List的值：  
  
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
     
c-命名空间的弱势就在于此，它无法做到加入集合的值。   
  
以上的类CDPlayer是通过构造函数注入compactDisc的bean，也可以通过setter方法注入。  
  
	package soundsystem;
	
	import org.springframework.beans.factory.annotation.Autowired;

	public class CDPlayer {
	
	    private CompactDisc compactDisc;
	
	    @Autowired
	    public void setCompactDisc(CompactDisc compactDisc) {
	        this.compactDisc = compactDisc;
	    }
	
	    public void play() {
	        compactDisc.play();
	    }
	} 
  
&emsp;   
  
	<bean id="cdPlayer" class="soundsystem.CDPlayer">
        <property name="compactDisc" ref="compactDisc"></property>
    </bean>   
  
也可以使用p-命名空间注入compactDisc，xml头部加入`xmlns:p="http://www.springframework.org/schema/p"`。  
  
	<bean id="cdPlayer" class="soundsystem.CDPlayer" p:compactDisc-ref="compactDisc"></bean>

p-命名空间也可以注入普通属性：
  
	<bean id="compactDisc" class="soundsystem.SgtPeppers" p:title="title" p:artist="artist"></bean>  
  
但不能注入集合，可以通过`<util:list>`元素注入集合的值，头部需要引入util。  
  
	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:p="http://www.springframework.org/schema/p"
	       xmlns:util="http://www.springframework.org/schema/util"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
	       http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
	
	    <bean id="compactDisc" class="soundsystem.SgtPeppers" p:title="title" p:artist="artist" p:tracks-ref="trackList"></bean>
	    <util:list id="trackList">
	        <value>s1</value>
	        <value>s2</value>
	        <value>s3</value>
	    </util:list>
	
	    <bean id="cdPlayer" class="soundsystem.CDPlayer" p:compactDisc-ref="compactDisc"></bean>
	</beans>
  
## 导入和混合配置 ##  
  
  
  
