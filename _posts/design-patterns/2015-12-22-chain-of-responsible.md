---
layout: single
title: 8.责任链模式
excerpt: "设计模式 责任链模式"
author_profile: false
sidebar:
  nav: "design-patterns"
categories: design-patterns
---

## 什么是责任链模式  
一个请求(或消息)从发出到被接受的过程中, 可能要经过多个逻辑处理,
我们把每一个逻辑处理封装到一个类中, 并把这些逻辑处理类加到一条链中,
而这个请求或者消息只需从这条链的开头, 一个节点一个节点的往下执行,
不需要考虑什么时候该执行哪个处理逻辑, 而是每个节点拿到请求(或消息)时
先判断是否需要处理请求或消息, 然后再由此节点继续传递或结束请求(或消息).

## 责任链模式(自链表) UML类图  
![责任链模式_自链表](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter09_cor_selfchain.png)  

- Request: 请求或消息
- Handler: 抽象的逻辑处理
- Handler1, Handler2: 具体的逻辑处理

## 责任链模式(自链表) 代码实例  

```java
//请求或消息
public class Request{
  //...
}

//抽象的逻辑处理
public abstract class Handler{
  public Handler mNext;
  public void handleRequest(Reqeust request);
}

//具体的处理逻辑
public class Handler1 extends Handler{
  public void handlerRequest(Request request){
    System.out.println("Handler1对request进行一些处理, 然后传递给下一个Handler");
    mNext.handleRequest(request);
  }
}

//具体的处理逻辑
public class Handler2 extends Handler{
  public void handlerRequest(Request request){
    System.out.println("Handler2对request进行一些处理, 然后传递给下一个Handler");
    mNext.handleRequest(request);
  }
}
```

## 责任链模式(非自链表) UML类图  
![责任链模式_非自链表](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter09_cor_not_selfchain.png)  

- Request: 请求或消息
- Handler: 抽象的逻辑处理
- Handler1, Handler2: 具体的逻辑处理
- HandlerContainer: 存储Handler的容器

## 责任链模式(非自链表) 实例代码  
用过OkHttp的同学都知道, OkHttp可以通过多次调用`addInterceptor(Interceptor)`添加拦截器. OkHttp中所有的拦截器(包括OkHttp自带的拦截器和用户添加的拦截器)都存储在一个ArrayList中, 通过遍历这个ArrayList取出其中的每一个拦截器处理请求. 这就是非自链表的责任链模式.  

```java
//OKHttp中拦截器的添加
private Response getResponseWithInterceptorChain() throws IOException {
  // Build a full stack of interceptors.
  List<Interceptor> interceptors = new ArrayList<>();
  interceptors.addAll(client.interceptors());// 用户自定义的拦截器
  interceptors.add(retryAndFollowUpInterceptor);
  interceptors.add(new BridgeInterceptor(client.cookieJar()));
  interceptors.add(new CacheInterceptor(client.internalCache()));
  interceptors.add(new ConnectInterceptor(client));
  if (!retryAndFollowUpInterceptor.isForWebSocket()) {
    interceptors.addAll(client.networkInterceptors());
  }
  interceptors.add(new CallServerInterceptor(
      retryAndFollowUpInterceptor.isForWebSocket()));

  Interceptor.Chain chain = new RealInterceptorChain(
      interceptors, 0, originalRequest);
  return chain.proceed(originalRequest);
}

public class RealInterceptorChain{
  private final List<Interceptor> interceptors;
  private final int index;
  private final Request request;

  //省略了一些参数
  public RealInterceptorChain(List<Interceptor> interceptors, int index, Request request) {
   this.interceptors = interceptors;
   this.index = index;
   this.request = request;
 }


  public Response proceed(Request request){
      //代码经过简化
      if (index >= interceptors.size()) throw new AssertionError();

      RealInterceptorChain chain = new RealInterceptorChain(interceptors, index+1, request);
      Response response = interceptor.intercept(chain);
      return response;
  }
}


public class XxxInterceptor extends Interceptor{
  public Response intercept(Chain chain) {
    Request request = chain.request();
    //对request的一些处理...
    return chain.proceed(reqeust);
  }
}

```
可以看出, RealInterceptorChain中的proceed(Request)与XxxInterceptor中的intercept(Chain)相互调用, 形成递归, 最终遍历完interceptors.
