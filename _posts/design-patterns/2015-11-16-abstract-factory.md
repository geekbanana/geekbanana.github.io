---
layout: single
title: 5.抽象工厂模式
excerpt: "设计模式 抽象工厂模式"
author_profile: false
sidebar:
  nav: "design-patterns"
categories: design-patterns
---

## 什么是抽象工厂模式  
上一篇博客我们讲了**工厂模式**, 那什么又是**抽象工厂模式**呢?  
**抽象工厂模式**通常有一个抽象工厂, 多个具体工厂, 每个具体工厂实现的是一整套具体产品. 什么是一整套具体产品? 举例来说:  

假设有两条汽车生产线分别生产BMW X5系列汽车和BMW X7系列汽车:  
生产一台汽车需要: 1.生产车身; 2.生产轮胎. (那么我们就可以把这步骤抽取出来, 作为抽象工厂类的方法).  
X5生产线需要: 1.生产X5车身; 2.生产适合X5的轮胎. (那么这两个方法的实现就放在X5Factory中)  
X7生产线需要: 1.生产X7车身; 2. 生产适合X7的轮胎. (同样,我们将这两个方法的实现放在X7Factory中)  

显而易见, X5Factory实现的X5那一套具体产品: 生产X5车身, 生产X5轮胎. X7Factory实现的就是X7那一套具体产品: 生产X7车身, 生产X7轮胎.  

## UML  
![抽象工厂模式](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter06_abstract_factory.png)

## 代码范例  
抽象类:

```java
//车身
public abstract class CarBody{
  public abstract CarBody bodyMethod();
}

//轮胎
public abstract class Tyre{
  public abstract Tyre tyreMethod();
}

//抽象工厂类
public abstract class Factory{
  public abstract CarBody productCarBody();
  public abstract Tyre productTyre();
}
```

X5相关类:

```java
//X5车身
public class X5CarBody{
  public CarBody bodyMethod(){
    //X5车身的方法...
  }
}

//X5轮胎
public class X5Tyre{
  public Tyre tyreMethod(){
    //X5轮胎的方法...
  }
}

//X5工厂类
public class X5Factory{
  public CarBody productCarBody(){
    return new X5CarBody();
  }
  public Tyre productTyre(){
    return new X5Tyre();
  }
}
```

X7相关类:

```java
//X7车身
public class X7CarBody{
  public CarBody bodyMethod(){
    //X7车身的方法...
  }
}

//X7轮胎
public class X7Tyre{
  public Tyre tyreMethod(){
    //X7轮胎的方法...
  }
}

//X7工厂类
public class X7Factory{
  public CarBody productCarBody(){
    return new X7CarBody();
  }
  public Tyre productTyre(){
    return new X7Tyre();
  }
}
```
