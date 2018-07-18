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
  
`@Profile`注解用于给bean标注，当该profile被激活时，才装配该profile对应的bean。  
  
### 在Java中配置profile bean  
  
People.java  

```java
package com.myapp;

public interface People {

    void speak();
}
```
  
<!--more-->
  
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
