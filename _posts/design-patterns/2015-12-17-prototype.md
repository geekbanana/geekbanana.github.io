---
layout: single
title: 3.原型模式
excerpt: "设计模式 原型模式"
author_profile: false
sidebar:
  nav: "design-patterns"
categories: design-patterns
---
## 1.原型模式介绍  
原型二字表明了该模式有一个样板实例, 用户从这个样板对象中复制出一个内部属性一致的对象,
这个过程就是我们俗称的"克隆"

## 2.原型模式使用场景
1. 类初始化需要消耗非常多的资源, 这些资源包括CPU资源, 硬件资源等, 通过原型拷贝避免这些消耗.  
2. 通过new产生一个对象需要非常繁琐的数据准备或访问权限  
3. 一个对象需要提供给其他对象访问, 而且各个调用者可能都需要修改其值时, 可以考虑使用原型模式
拷贝多个对象供调用者使用, 即保护性拷贝.  

注意:  
只有通过new构造对象较为耗时或者成本较高时, 通过clone方法才能够获得效率上的提升.  
通过clone拷贝对象时并不会执行构造函数,如果在构造函数中需要一些特殊的初始化操作类型,
需注意构造函数不会执行的问题.  

## 3.浅拷贝(影子拷贝)  
Word大家一定都用过, 我们在修改一篇重要文档时, 通常先拷贝一个副本, 然后在副本上进行修改.
现在我们假设类WordDocument就是一篇文档, 文档中中有文字mText和图片mImages, 可以进行clone  

```java
public class WordDocument implements Cloneable {
	// 文本
	private String mText;
	// 图片名列表
	private List<String> mImages = new ArrayList<String>();

	public WordDocument() {
		System.out.println("++++++++++ WordDocument 构造函数 ++++++++++");
	}

	@Override
	protected WordDocument clone() {
		try {
			WordDocument doc = (WordDocument) super.clone();
			doc.mText = this.mText;
			doc.mImages = this.mImages;
			return doc;
		} catch (Exception e) {
		}
		return null;
	}

	public String getText() {
		return mText;
	}

	public void setText(String mText) {
		this.mText = mText;
	}

	public List<String> getImages() {
		return mImages;
	}

	public void addImage(String img) {
		this.mImages.add(img);
	}

	/**
	 * 打印文档内容
	 */
	public void showDocument() {
		System.out.println("----------- Word Content Start -----------");
		System.out.println("Text : " + mText);
		System.out.println("Images List: ");
		for (String imgName : mImages) {
			System.out.println("image name : " + imgName);
		}
		System.out.println("----------- Word Content End -----------");
	}
}
```

下面我们模拟一下文档的新建, 拷贝, 修改等操作  

```java
public class Client {
	public static void main(String[] args) {
		// 1、构建文档对象
		WordDocument originDoc = new WordDocument();
		// 2、编辑文档,添加图片等
		originDoc.setText("这是一篇文档");
		originDoc.addImage("图片 1");
		originDoc.addImage("图片 2");
		originDoc.addImage("图片 3");
		originDoc.showDocument();

		// 以原始文档为原型,拷贝一份副本
		WordDocument doc2 = originDoc.clone();
		doc2.showDocument();
		// 修改文档副本,不会影响原始文档
		doc2.setText("这是修改过的 Doc2 文本");
		doc2.showDocument();
		originDoc.showDocument();
	}
}
```

输入内容如下:  

```
// originDoc
++++++++++ WordDocument 构造函数 ++++++++++
----------- Word Content Start -----------
Text : 这是一篇文档
Images List:
image name : 图片1
image name : 图片2
image name : 图片3
----------- Word Content End -----------

// doc2
----------- Word Content Start -----------
Text : 这是一篇文档
Images List:
image name : 图片1
image name : 图片2
image name : 图片3
----------- Word Content End -----------

// 修改后的doc2
----------- Word Content Start -----------
Text : 这是修改过的doc2文本
Images List:
image name : 图片1
image name : 图片2
image name : 图片3
----------- Word Content End -----------

// originDoc
----------- Word Content Start -----------
Text : 这是一篇文档
Images List:
image name : 图片1
image name : 图片2
image name : 图片3
----------- Word Content End -----------
```

目前看来, 原文档(originDoc)可clone出来的文档(doc2)是两个独立的文档, 各自的修改不会影响到另一方.  
但是当我们做如下操作时:

```java
public class Client {
	public static void main(String[] args) {
		// 1、构建文档对象
		WordDocument originDoc = new WordDocument();
		// 2、编辑文档,添加图片等
		originDoc.setText("这是一篇文档");
		originDoc.addImage("图片 1");
		originDoc.addImage("图片 2");
		originDoc.addImage("图片 3");
		originDoc.showDocument();

		// 以原始文档为原型,拷贝一份副本
		WordDocument doc2 = originDoc.clone();
		doc2.showDocument();
		// 修改文档副本,不会影响原始文档
		doc2.setText("这是修改过的 Doc2 文本");
    doc2.addImage("多了张图片有木有.jpg")
		doc2.showDocument();
		originDoc.showDocument();
	}
}
```

此时输入内容如下:
输入内容如下:  
```
// originDoc
++++++++++ WordDocument 构造函数 ++++++++++
----------- Word Content Start -----------
Text : 这是一篇文档
Images List:
image name : 图片1
image name : 图片2
image name : 图片3
----------- Word Content End -----------

// doc2
----------- Word Content Start -----------
Text : 这是一篇文档
Images List:
image name : 图片1
image name : 图片2
image name : 图片3
----------- Word Content End -----------

// 修改后的doc2
----------- Word Content Start -----------
Text : 这是修改过的doc2文本
Images List:
image name : 图片1
image name : 图片2
image name : 图片3
iamge name : 多了张图片有木有.jpg
----------- Word Content End -----------

// originDoc
----------- Word Content Start -----------
Text : 这是一篇文档
Images List:
image name : 图片1
image name : 图片2
image name : 图片3
iamge name : 多了张图片有木有.jpg
----------- Word Content End -----------
```
奇怪的现象出现了: 修改了doc2的mText没有影响到originDoc, 给doc2多加了一张图片却导致originDoc也多了一张图片.
这是为什么呢?  
这是因为我们在clone()方法中通过`doc.mImages = this.mImages`把originDoc的mImages的引用赋给了doc2的mImages,
两个mImages指向的是同一块地址空间, 所以才会导致上述结果.  
这种拷贝我们称之为**浅拷贝(影子拷贝)**, 要想doc2.mImages的修改不影响到originDoc.mImages, 那我们可以通过**深拷贝**来完成  

## 4.深拷贝  

**深拷贝**只需将**浅拷贝**中的`doc.mImages = this.mImages`改为`doc.mImages = (ArrayList<String>) this.mImages.clone()`,
完整的深拷贝clone代码如下:
```java
@Override
protected WordDocument clone() {
  try {
    WordDocument doc = (WordDocument) super.clone();
    doc.mText = this.mText;

    //doc.mImages = this.mImages;//  <-- 浅拷贝的写法

    doc.mImages = (ArrayList<String>) this.mImages.clone();//  <-- 深拷贝的写法
    return doc;
  } catch (Exception e) {
  }
  return null;
}
```

## 5.拷贝的应用--保护性拷贝  
假设我们负责一个应用的登录模块设计. 登录成功后, 势必要保存一些用户信息. 而为了保证数据的一致性,
这些数据不能直接在本地修改, 必须将修改的数据提交到服务器, 再由服务器返回最新的用户信息.
很快我们写出如下代码:  
```java
/**
 * 用户实体类
 */
 public class User{
	 public int age;
	 public String name;
	 public String phoneNum;
	 public Address address;
 }

 /**
  * 地址
	*/
public class Address{
	public String city;
	public String district;//区
	public String street;

	public Address(String city,String district, String street){
		this.city = city;
		this.district = district;
		this.street = street;
	}
}

//登录接口
public Interface Login{
	void login();
}

//登录实现
public class LoginImpl implements Login{
	public void login(){
		//登录到服务器, 获取用户信息
		User loginedUser = new User();

		//将服务器返回的完整信息设置给loginedUser对象
		loginedUser.age = 22;
		loginedUser.name = "cavalry"
		loginedUser.address = new address("广州市","天河区","黄埔大道");

		//将登录后的用户信息设置到LoginSession中
		LoginSession.getInstance().setLoginedUser(loginedUser);
	}
}

//登录Session
public class LoginSession{
	static volatile LoginSession sLoginSession = null;
	private User loginedUser;
	private LoginSession(){}

	public static LoginSession getInstance(){
		if(sLoginSession==null){
			synchronized(LoginSession.class){
				if(sLoginSession==null){
					sLoginSession = new LoginSession();
				}
			}
		}
	}

	//设置已登录的用户信息, 包权限, 不对外开放
	void setLoginedUser(User user){
		loginedUser = user;
	}

	public User getLoginedUser(){
		return loginedUser;
	}

}
```
为了保证用户信息不被随意修改, 我们将LoginSession放在与登录同一级包下, 并将setLoginedUser(User)
的访问权限设置为包访问权限. 但是这样就可以确保用户信息不被随意修改么? 如果有人这样做呢
`LoginSession.getLoginedUser().address.city = "北京";` . 因为Java值传递的原因,
这样同样可以直接在本地修改用户信息.  此时, 我们可以通过**保护性拷贝**来解决这个问题
```java
/**
 * 用户实体类
 */
public class User implements Cloneable{
	public User clone(){
		User user = null;
		try{
			user = (User) super.clone();
		}catch (CloneNotSupportedException e){
			e.printStackTrace();
		}
		return user;
	}
	// 代码省略
}

//并且在LoginSession中做如下修改:
public User getLoginedUser(){
	return loginedUser.clone();
	//return loginedUser;
}
```
这样, 客户端只能通过setLoginedUser(User)来修改用户信息. 而setLoginedUser(User)为包访问权限, 只有与
LoginSession在同一个包下的类才可以使用.
