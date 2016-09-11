---
layout: archive
title: 创建和销毁对象——静态工厂方法
excerpt: "java 静态工厂方法"
author_profile: false
sidebar:
  nav: "java"
categories: java effective-java
---

## Creating and Destorying Objects  
> 本文关注重点:  
1. 何时，以何种方式，创建对象？  
2. 何时，以何种方式，避免创建对象？  
3. 怎样确保对象被及时销毁？  
4. 怎样管理对象销毁之前的一些清理动作？  

### 1.静态工厂方法  
比如下面这个将原始boolean类型数据转换为Boolean对象的例子  
{% highlight java linenos %}
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
{% endhighlight %}

#### 静态工厂方法的优点  

1. 与构造方法相比, **静态工厂方法**的名字可以不必和类名一致,因此:  
  - 可以更好的描述返回值  
  - 不会因参数类型相同, 但参数列表的顺序不同而产生使用上的混淆.(因为可以取不同的名字)  
2. 与构造方法相比, **静态工厂方法**不必每次调用时都创建新的对象, 因此可以保证在任何时间, 内存中只有唯一一个对象,称之为**实例控制**(instance-controlled). 实例控制确保了 a.equals(b) 且 a==b, 因此可以使用更高效的 == 来判断两个对象是否是同一个对象.
3. 与构造方法相比, **静态工厂方法**可以return返回类型的子类型. 这种灵活性带来的优点是:  
  - 可以返回一个非public类的对象. 适用于[基于接口的框架(interface-based frameworks)](#interface_based_frameworks)的API设计  
  - 可以[根据不同的参数类型返回不同的对象.](#return_different_object)  
  - 或者可以在API的不同release版本中返回不同的对象. 提高程序的可维护性和性能.  
  - The class of the object returned by a static factory method need not event exist at the time the class containing the method is written. (The class of the object returned 甚至可以在 the class containing the method 书写时根本不存在). 这种灵活的设计常见于[服务提供者框架(service provider frameworks)](#service_provider_frameworks)  
4. [可以减少创建对象的参数化冗余.](#type_inference) (JKD_1.8 中已经不存在这个问题了)  

#### 静态工厂方法的缺点  
1. 因为没有public或者protected构造方法, 静态工厂方法类不能被继承.  
2. 不像构造方法那么容易辨别.不过我们可以通过一下方法来增加识别:  
  - 在类注释中增加静态工厂方法的说明  
  - 遵循一些约定的命名规则:  
    - valueOf  
    - of —— valueOf的简单写法,常用在EnumSet  
    - getInstance  
    - newInstance  
    - getType —— 类似于getInstance, 当工厂方法有多个不同类型的对象返回时  
    - newType —— 类似于newInstance  

## Tips
### <span id="interface_based_frameworks">1.基于接口的框架(interface-based frameworks)</span>  
> 接口不能有静态方法. 因此, 如果一个接口命名为**Type**, 它的静态方法会被放在一个命名为**Types**的不可实例化的类中.  

以java.util.Collections为例:  
{% highlight java linenos %}
public class Collections {

  private Collections() {}  //不可实例化

  /**
   * Returns a list containing the specified number of the specified element.
   * The list cannot be modified. The list is serializable.
   *
   * @param length
   *            the size of the returned list.
   * @param object
   *            the element to be added {@code length} times to a list.
   * @return a list containing {@code length} copies of the element.
   * @throws IllegalArgumentException
   *             when {@code length < 0}.
   */
   //返回Collection的子类
  public static <T> List<T> nCopies(final int length, T object) {
      return new CopiesList<T>(length, object);
  }

  private static final class CopiesList<E> extends AbstractList<E> implements Serializable {
        private static final long serialVersionUID = 2739099268398711800L;
        private final int n;
        private final E element;

        CopiesList(int length, E object) {
            if (length < 0) {
                throw new IllegalArgumentException("length < 0: " + length);
            }
            n = length;
            element = object;
        }

        @Override public boolean contains(Object object) {
            return element == null ? object == null : element.equals(object);
        }

        @Override public int size() {
            return n;
        }

        @Override public E get(int location) {
            if (location >= 0 && location < n) {
                return element;
            }
            throw new IndexOutOfBoundsException();
        }
    }


    /**
     * Returns a type-safe empty, immutable {@link List}.
     *
     * @return an empty {@link List}.
     * @since 1.5
     * @see #EMPTY_LIST
     */
     //返回Collection的子类
    @SuppressWarnings("unchecked")
    public static final <T> List<T> emptyList() {
        return EMPTY_LIST;
    }

    /**
    * An empty immutable instance of {@link List}.
    */
   @SuppressWarnings("unchecked")
   public static final List EMPTY_LIST = new EmptyList();

    private static final class EmptyList extends AbstractList
            implements RandomAccess, Serializable {
        private static final long serialVersionUID = 8842843931221139166L;

        @Override public boolean contains(Object object) {
            return false;
        }

        @Override public int size() {
            return 0;
        }

        @Override public Object get(int location) {
            throw new IndexOutOfBoundsException();
        }

        @Override public Iterator iterator() {
            return EMPTY_ITERATOR;
        }

        private Object readResolve() {
            return Collections.EMPTY_LIST;
        }
    }
}
{% endhighlight %}
Collections中的CopiesList<E>和EmptyList都是Collection接口的实现类. 这两个类都是private, 并通过静态工厂方法对外提供. 像这样的类, 在Collections中有三十几个.  

### <span id="return_different_object">2.根据不同的参数类型返回不同的对象</span>  
以 java.util.EnumSet 为例, 这个类没有public构造方法,只有静态工厂方法: 根据枚举的size返回不同类型的对象. 如果size<=64,返回MiniEnumSet<E>; 否则返回HugeEnumSet<E>.  

### <span id="service_provider_frameworks">3.服务提供者框架(service provider frameworks)</span>  
> 服务提供者框架有3个必要组件和1个可选组件  
1. 服务接口(service interface), which providers implement  
2. 提供者注册API(provider registration API), which the system uses to register implements, giving clients access to them  
3. 服务获得API(service access API), which clients use to obtain an instance of the service  
4. [可选]服务提供者接口(service provider interface), which providers implement to create instances of their service implementation  

下面是服务提供者框架的基本组成
{% highlight java linenos %}
//Service interface
public interface Service{
  ...//服务的一些方法
}

//Service provider interface
public interface Provider{
  Service newService();
}

//Noninstantiable class for service registration and access
public class Services{
  private Services(){}

  //Maps service names to Services
  private static final Map<String, Provider> providers = new ConcurrentHashMap<String, Provider>();
  public static final String DEFAULT_PROVIDER_NAME = "<def>";

  //Provider registration API
  public static void registerDefaultProvider(Provider p){
    registerProvider(DEFAULT_PROVIDER_NAME,p);
  }
  public static void registerProvider(String name,Provider p){
    providers.put(name,p);
  }

  //Service access API
  public static Service newInstance(){
    return newInstance(DEFAULT_PROVIDER_NAME);
  }
  public static Service newInstance(String name){
    Provider p = providers.get(name);
    if(p == null)
      throw new IllegalArgumentException("No provider registered with name: " + name);
    return p.newService();
  }
}
{% endhighlight %}

### <span id="type_inference">4.类型推导减少创建对象时的参数冗余</span>  
比如,创建一个HashMap,需要写两次泛型:  
`Map<String,List<String>> map = new HashMap<String,List<String>>();`  
假如HashMap提供了如下静态工厂方法:  
{% highlight java linenos %}
public static <K,V> HashMap<K,V> newInstance(){
  return new HashMap<K,V>();
}
{% endhighlight %}
那么我们就可以通过下面这个简洁的方法创建HashMap对象:  
`Map<String, List<String>> map = HashMap.newInstance();`  
编译器会自动为我们计算出参数的类型,这称之为**类型推导(type inference)**  
