---
layout: single
title: 12.备忘录模式
excerpt: "设计模式 备忘录模式"
author_profile: false
sidebar:
  nav: "design-patterns"
categories: design-patterns
---

## 什么是备忘录模式  
**备忘录模式**可以保存对象的当前状态, 以便之后可以恢复到当前状态.  


## 备忘录模式UML类图  
![备忘录模式](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter13_memorandum.png)

- Originator: 需要保存状态的类
- State: 状态类
- Caretaker: 用来存储状态, 不能对备忘录的内容进行操作和访问, 只能够将备忘录
传递给其他对象.

## 代码实例  

假设我们玩一款游戏, 游戏可以保存进度以便下次可以继续  

```java
public class Warcraft{
  private int mLevel;
  private int mBlood;

  public State createState(){
    State state = new State();
    state.level = mLevel;
    state.blood = mBlood;
    return state;
  }

  public void restoreState(State state){
    this.mLevel = statte.level;
    this.mBlood = state.blood;
  }

  public void play(){
    mLevel = 24;
    mBlood = 800;
  }
}

public class State{
  public int level;
  public int blood;
}

public class Caretaker{
  private State mState;

  public void saveState(State state){
    this.mState = state;
  }

  public State getState(){
    return mState;
  }
}

public class Client{
  public static void main(String[] args){
    //玩游戏
    Warcraft warcraft = new Warcraft();
    warcraft.play();

    //保存状态
    Caretaker caretaker = new Caretaker();
    caretaker.saveState(warcraft.createState());

    //第二次进入游戏, 恢复状态
    warcraft = new Warcraft();
    warcraft.restoreState(caretaker.getState());

  }
}
```
