---
title: Spring装配Bean
date: 2018-07-10 19:40:26
tags: Spring
categories：
- 后端
- Spring
--- 
## 自动化装配bean  

* @Component、@ComponentScan和@Autowired    
  
```java  
package soundsystem;  

public interface CompactDisc {  
    void play();  
}  
```  
  
```java  
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
```  
  
@Component表明类CompactDisc是一个组件类，默认ID为sgtPeppers。也可以指定bean的ID，`@Component("LonelyHeartsClub")`将ID指定为lonelyHeartsClub。@Named也可以来设置bean的ID，`@Named("lonelyHeartsClub")`，两者差别不大。
  
```  
package soundsystem;  

import org.springframework.context.annotation.ComponentScan;  
import org.springframework.context.annotation.Configuration;  

@Configuration  
@ComponentScan  
public class CDPlayerConfig {  
  
}  
```  
  
@ComponentScan启用组件扫描，默认扫描与配置类相同的包。可以通过设置@ComponentScan的value值来设置组件扫描的包，`@ComponentScan("soundsystem")`。可以设置多个包，`ComponentScan(basePackages={"soundsystem", "package1"})`。也可以设置扫描的类，`ComponentScan(basePackageClasses={CDPlayer.class, DVDPlayer.class})`  
  
测试，引入Spring和Junit的jar包  。
  
![](https://i.imgur.com/6GNBpjZ.png)  
![](https://i.imgur.com/SdUkQXL.png)  
  
运行提示找不到hamcrest包，降低Junit版本或者加入hamcrest包，由于降低Junit版本之后和spring适配有问题，只好加入hamrest包了。  
  
![](https://i.imgur.com/vArScZC.png)
  
```  
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
```    
  
@Autowired可以将扫描到的bean装配到注解的地方，可以在类中的属性上注解，也可以在方法上注解。@Autowired的required属性默认为true，当没有bean时，会报错。将required属性设置为false，`@Autowired(required=false)`，当没有匹配的bean，这个bean就会处于未装配状态，不会报错。@Inject和@Autowired作用类似。  
  
```  
private CompactDisc compactDisc;  
  
@Autowired
public void setCompactDisc(CompactDisc compactDisc) {
    this.compactDisc = compactDisc;
}  
```  
  
* JavaConfig  
  
要装配的类：
  
```
package soundsystem;

public class SgtPeppers {

    private String title = "title";
    private String artist = "artist";


    @Override
    public void play() {
        System.out.println("Playing " + title + " by " + artist);
    }
}
```  
  
```  
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
```
   
配置类：
  
```  
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
```  
  
测试类：  
```  
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
```  
  
@Bean装配bean，配置类中，也可以用如下方式进行装配。在装配CDPlayer过程中，可以由sgtPeppers()或CompactDisc compactDisc找到装配的CompactDisc的bean，并注入到cdPlayer()方法中。

 


   

  

  
