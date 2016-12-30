---
layout: single
title: 18.组合模式
excerpt: "设计模式 组合模式"
author_profile: false
sidebar:
  nav: "design-patterns"
categories: design-patterns
---

## 什么是组合模式  
我们先来看一幅公司的组织结构树状图
![公司组织结构树状图](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter19_composite_tree.png)

像公司组织这样的结构, 整体(总公司)可以拆分出部分(子公司), 并且部分又与整体的结构类似,
对于这种结构我们即可使用**组合模式**.  
在**组合模式**中, 我们称根部的节点(总公司)为 *根结构件* , 称子公司这样的节点为 *枝干构件* ,
像行政部、研发部这些没有子节点的节点我们称之为 *叶子构件* .  

## 安全组合模式UML类图  
![安全组合模式](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter19_composite_safe.png)

## 安全组合模式代码模板  
```java
public abstract class Component{
  protected String name;

  public Component(String name){
    this.name = name;
  }

  public abstract void doSomething();
}

//具体枝干节点
public class Composite extends Component{
  private List<Component> components = new ArrayList<Component>();

  public Composite(String name){
    super(name);
  }

  public void doSomething(){
    for(Component c : components){
      c.doSomething();
    }
  }

  public void addChild(Component child){
    components.add(child);
  }

  public Component getChildAt(int index){
    return components.getChildAt(index);
  }

  public void removeChild(Component child){
    components.removeChild(child);
  }
}

//叶子节点
public class Leaf extends Component{
  public Leaf(String name){
    super(name);
  }

  public void doSomething(){
    System.out.println(".....");
  }
}

//客户端
public class Client{
  public static void main(String[] args){
    //构造一个根节点
    Composite root = new Composite("root");

    //构造两个枝干节点
    Composite branch1 = new Composite("branch1");
    Composite branch2 = new Composite("branch2");

    //构造六个叶子节点
    Leaf leaf1 = new Leaf("leaf1");
    Leaf leaf2 = new Leaf("leaf2");
    Leaf leaf3 = new Leaf("leaf3");
    Leaf leaf4 = new Leaf("leaf4");
    Leaf leaf5 = new Leaf("leaf5");
    Leaf leaf6 = new Leaf("leaf6");

    root.addChild(leaf1);
    root.addChild(leaf2);
    branch1.addChild(leaf3);
    branch1.addChild(leaf4);
    branch2.addChild(leaf5);
    branch2.addChild(leaf6);

    root.addChild(branch1);
    branch1.addChild(branch2);

    //执行方法
    root.doSomething();
  }
}
```

这就是**组合模式**中的 *安全组合模式* , 它与面向对象设计原则中的**依赖倒置原则**是相违背的.
因此还有一种复合依赖倒置原则的 *透明组合模式* .  

## 透明组合模式UML类图  
![透明组合模式](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter19_composite_transparent.png)  

透明组合模式与安全组合模式的区别在于, 透明组合模式将子节点中的addChild, removeChild, getChildAt
等方法也抽取到了Component中.  
