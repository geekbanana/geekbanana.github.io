---
layout: single
title: 20.装饰模式
excerpt: "设计模式 装饰模式"
author_profile: false
sidebar:
  nav: "design-patterns"
categories: design-patterns
---

Java中的BufferedReader即是采用装饰模式进行设计的.  

我们先来看一下BufferedReader的使用方法

```java
Reader reader = new FileReader(file);
BufferedReader br = new BufferedReader(reader);
br.readLine();//读取一行数据
```

FileReader中只有`public int read(char[] buffer, int offset, int count)`方法, 可以将指定范围的字符读入字符数组buffer中.  

现想给FileReader增加一个按行读取的readLine()方法, 可以:  
1. 采用继承的方式, 在子类中增加readLine()方法
2. 采用装饰模式, 将FileReader对象作为一个参数传递到BufferedReader中, 在BufferedReader中增加readLine()方法.  

相较于继承, 装饰模式的优点是避免了继承模式的臃肿. BufferedReader只需对外暴露它自己所特有的几个方法, 而不会暴露FileReader的方法. 减少了客户使用的复杂性.  

FileReader和BufferReader大致代码如下:  
```java
public class FileReader{
  public FileReader(File file){
    //...
  }

  public int read(char[] buffer, int offset, int count) throws IOException {
    //...
  }
}

public class BufferedReader{
  private Reader reader;

  public BufferedReader(Reader reader){
    this.reader = reader;
    //..
  }

  public String readLine(){
    //通过reader.read(...)获得一行的字符串
    read.read(buffer,offset,count);

    String line = 将字符数组buffer转换为字符串;

    return line;
  }
}
```
