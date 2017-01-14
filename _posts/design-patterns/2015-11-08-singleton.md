---
layout: single
title: 1.单例模式
excerpt: "设计模式 单例"
author_profile: false
sidebar:
  nav: "design-patterns"
categories: design-patterns
---

实现单例模式主要有如下几个关键点:  
1.构造函数一般为private  
2.通过静态方法或者枚举返回单例类对象  
3.确保单例类的对象有且只有一个, 尤其是在多线程环境下  
4.确保单例对象在反序列化时不会被重新构建对象  

## 1.饿汉式  
```java
public class Singleton{
  private static final Singleton INSTANCE = new Singleton();

  private Singleton(){}

  public static Singleton getInstance(){
    return INSTANCE;
  }  
}
```

优点:  

- 在多线程的环境下仍可确保有且只有一个对象.

缺点:   

- 不论对象是否被使用都会初始化

## 2.懒汉式  
```java
public class Singleton{
  private static  Singleton sInstance = null;

  private Singleton(){}

  public static synchronized Singleton getInstance(){
    if(sInstance == null){
      sInstance = new Singleton();
    }
    return sInstance;
  }  
}
```

优点:  

- 对象在使用时才初始化

缺点:  

- 不论对象sInstance是否已经初始化, 每次调用getInstance方法都会进行同步, 消耗不必要的资源

## 3.双检查锁(Double Check Lock, DCL)  
```java
public class Singleton{
  //sInstance最好能用volatile关键字修饰, 后面会给出解释
  private static  Singleton sInstance = null;

  private Singleton(){}

  public static Singleton getInstance(){
    if(sInstance == null){
      synchronized (Singleton.class){
        if(sInstance == null){
          sInstance = new Singleton();
        }
      }
    }
    return sInstance;
  }  
}
```
优点:  

- 对象在使用时才初始化
- 资源利用率高

双检查锁单例模式, 乍一看好像没什么缺点, 实际上它还是有问题的.  
sInstance = new Singleton(), 看起来是一句代码, 但实际上它并不是一个原子操作.
这句代码最终会被编译成多条汇编指令, 它大致做了如下三件事情:  

1. 给Singleton实例分配内存空间  
2. 调用Singleton构造函数, 初始化成员字段.  
3. 将sInstance指向分配的内存空间(此时sInstance就不是null了)  

Java编译器允许处理器乱序执行, 所以2和3的执行顺序是不确定的. 也就是说, 执行顺序可能是1-2-3, 也可能是 1-3-2. 假设A线程先执行了1和3, 还没来得及执行2就切换到了B线程.
B线程中sInstance已经不为null, 遂直接返回sInstance, 但Singleton实例实际上没有初始化, 此时客户端去使用sInstance就会报错.  

为了解决这个问题, 从JDK1.5开始, 可以使用**volatile**关键字来修饰sInstance, 这样就可以保证sInstance对象每次都是从主内存中获取.(JVM中每一线程运行都有一个线程栈, 线程栈中存储了此线程运行过程中的一些变量, 线程访问对象的值时, 会将对象的值load到线程本地内存中). **volatile**会对性能有些许影响, 不过考虑到程序的正确性, 这点影响还是值得的.  

使用**volatile**修饰之后, 仍然具有以下缺点:  

- 第一次加载稍慢, 由于Java内存模型的原因偶尔会失败
- 在高并发的环境下也有一定缺陷, 虽然发生的概率很小

DCL模式是使用最多的单例模式, 除非我们的并发场景比较复杂, 或JDK低于1.5 , 那么这种方式一般可以满足需求.

## 4.静态内部类单例模式  
```java
public class Singleton{
  private Singleton(){}
  public static Singleton getInstance(){
    return SingletonHolder.sInstance;
  }

  /**
   * 静态内部类
   */
  private static class SingletonHolder{
    private static final Singleton sInstance = new Singleton();
  }
}
```

第一次加载Singleton类时不会初始化sInstance, 只有第一次调用getInstance()时才会初始化sInstance.
第一次调用getInstance()会导致虚拟机加载SingletonHolder类, 这种方式不仅能够确保线程安全,
也保证了单例对象的唯一, 同时也延迟了对象的加载.

以上四种单例模式, 在反序列化时仍然会创建新的实例. 若要杜绝反序列化时创建新的实例, 需要在类中加入如下方法:

```java
private Object readResolve() throws ObjectStreamException{
  return sInstance;
}
```
也就是readResolve()时返回sInstance对象, 而不是重建对象.

## 5.枚举单例  
```java
public enum Singleton{
  INSTANCE;

  public void doSomething(){
    //...
  }
}
```

枚举在Java中与普通类一样, 不仅能够有字段, 还可以有自己的方法. 更中要的是, 枚举默认是线程安全的,
并且在任何情况下都是一个单例(即使反序列化).  

## 6.使用容器实现单例模式  
```java
public class SingletonManager{
  private static Map<String,Object> objMap = new HashMap<String,Object>();

  private SingletonManager(){}

  public static void registerService(String key, Object instance){
    if(!objMap.contains(key)){
      objMap.put(key,instance);
    }
  }

  public static Object getService(String key){
    return objMap.get(key);
  }
}
```

将单例对象注入到统一的管理器中, 使用时根据key获取, 方便管理多种类型的单例.
