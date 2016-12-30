---
layout: single
title: 21.享元模式
excerpt: "设计模式 享元模式"
author_profile: false
sidebar:
  nav: "design-patterns"
categories: design-patterns
---

## 什么是享元模式  
享元模式(Flyweight)是对象池的一种实现, 可有效的支持大量细粒度对象的使用. 通俗地说, 享元模式
就是一种用于对象缓存的模式, 当一个类的对象需要重复创建与销毁时, 可以将创建的对象缓存起来, 下一次
创建对象时先从缓存中取, 若缓存中不存在时再重新创建.  

享元对象中部分状态是可以共享的, 可共享的状态称之为 *内部状态* , 内部状态不会随着环境改变而改变;
不可共享的的状态称之为 *外部状态* , 外部状态会随着外部环境的变化而变换.

经典的享元模式设计, 通常会将对象存放在一个Map中, 以对象的内部状态作为key, 此对象作为value.  

## 享元模式应用举例  
以购买火车票为例, 用户的每次一查询, 服务器都要返回一个结果给用户, 这个结果就封装在一个对象中.
如果不采用享元模式, 结果返回后, 这个对象也就使用完毕, 等待销毁, 如此一来就会造成大量重复对象
的创建与销毁, 给服务器造成很大的负担.  

若采用享元模式, 假设第一个用户查询从青岛到广州的车票, 结果返回给用户后, 我们将对象存储在一个Map中,
当第二个用户查询从青岛到广州的车票时, 我们先从Map中取对象, 若对象存在, 则直接使用这个对象; 若不存在,
则重新创建一个对象. 这样就可以节省一些对象的重复创建问题.   

```java
public interface Ticket {
    public void showTicketInfo(String bunk);
}

// 火车票
class TrainTicket implements Ticket {
    public String from; // 始发地
    public String to; // 目的地
    public String bunk; // 铺位
    public int price;

    TrainTicket(String from, String to) {
        this.from = from;
        this.to = to;
    }

    @Override
    public void showTicketInfo(String bunk) {
        price = new Random().nextInt(300);
        System.out.println("购买 从 " + from + " 到 " + to + "的 "
                + bunk + " 火车票" + ", 价格 : " + price);
    }

}

/**
 * 车票工厂,以出发地和目的地为key缓存车票
 *
 * @author mrsimple
 */
public class TicketFactory {
    static Map<String, Ticket> sTicketMap = new ConcurrentHashMap<String, Ticket>();

    public static Ticket getTicket(String from, String to) {
        String key = from + "-" + to;//内部状态作为key
        if (sTicketMap.containsKey(key)) {
            System.out.println("使用缓存 ==> " + key);
            return sTicketMap.get(key);
        } else {
            System.out.println("创建对象 ==> " + key);
            Ticket ticket = new TrainTicket(from, to);
            sTicketMap.put(key, ticket);
            return ticket;
        }
    }
}

public class Test {
    public static void main(String[] args) {
        Ticket ticket01 = TicketFactory.getTicket("北京", "青岛");
        ticket01.showTicketInfo("上铺");
        Ticket ticket02 = TicketFactory.getTicket("北京", "青岛");
        ticket02.showTicketInfo("下铺");
        Ticket ticket03 = TicketFactory.getTicket("北京", "青岛");
        ticket03.showTicketInfo("坐票");
    }
  }
```

## 不区分状态的享元模式  
若对象不区分状态, 在Android中可以使用`android.support.v4.util.Pools`轻松的实现享元模式.
