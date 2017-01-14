---
layout: single
title: 创建和销毁对象
excerpt: "java 静态工厂方法"
author_profile: false
sidebar:
  nav: "java"
categories: java effective-java
---

# Creating and Destorying Objects  
> 本文关注重点:  
1. 何时，以何种方式，创建对象？  
2. 何时，以何种方式，避免创建对象？  
3. 怎样确保对象被及时销毁？  
4. 怎样管理对象销毁之前的一些清理动作？  

## 一、静态工厂方法  
比如下面这个将原始boolean类型数据转换为Boolean对象的例子  
{% highlight java linenos %}
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
{% endhighlight %}

### 静态工厂方法的优点  

1. 与构造方法相比, **静态工厂方法**的名字可以不必和类名一致,因此:  
  - 可以更好的描述返回值  
  - 不会因参数类型相同, 但参数列表的顺序不同而产生使用上的混淆.(因为可以取不同的名字)  
2. 与构造方法相比, **静态工厂方法**不必每次调用时都创建新的对象, 因此可以保证在任何时间, 内存中只有唯一一个对象,称之为**实例控制**(instance-controlled). 实例控制确保了 a.equals(b) 且 a==b, 因此可以使用更高效的 == 来判断两个对象是否是同一个对象.
3. 与构造方法相比, **静态工厂方法**可以return返回类型的子类型. 这种灵活性带来的优点是:  
  - 可以返回一个非public类的对象. 适用于<a href="#interface_based_frameworks"><font color="blue">基于接口的框架(interface-based frameworks)</font></a>的API设计  
  - 可以<a href="#return_different_object"><font color="blue">根据不同的参数类型返回不同的对象.</font></a>  
  - 或者可以在API的不同release版本中返回不同的对象. 提高程序的可维护性和性能.  
  - The class of the object returned by a static factory method need not event exist at the time the class containing the method is written. (The class of the object returned 甚至可以在 the class containing the method 书写时根本不存在). 这种灵活的设计常见于<a href="#service_provider_frameworks"><font color="blue">服务提供者框架(service provider frameworks)</font></a>  
4. <a href="#type_inference"><font color="blue">可以减少创建对象的参数化冗余.</font></a> (JKD_1.8 中已经不存在这个问题了)  

### 静态工厂方法的缺点  
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

## <a href="#build_pattern"><font color="blue">二、Builder模式</font></a>  
> 如果一个类有较多可选参数,可以考虑使用Builder模式.  

1. <a href="#telescoping_constructor_pattern"><font color="blue">重叠构造模式(telescoping constructor pattern)</font></a>, 会因可选参数过多导致创建太多的构造方式, 难以使用.  

2. <a href="#java_beans_pattern"><font color="blue">JavaBean模式(JavaBeans pattern)</font></a>, 虽然可以解决重叠构造模式的弊端, 但是JavaBean模式也导致了对象构建过程的不连续,和使用过程中的不稳定.  

3. Builder模式的缺点就是, 创建对象之前需要先构造一个builder, 对性能会有些许影响.  

## 三、使用private构造方法或者枚举类型强化singleton  
> 单例模式的缺点是难以测试: 除非单例类implements一个可以代表它的类型的接口, 否则无法使用模拟实现的类来代替单例类.  

有以下几种实现singleton的方式:  

1.公有属性方式(public static final field):  

{% highlight java linenos %}
public class Elvis{
  public static final Elvis INSTANCE = new Elivis();
  private Elvis(){ ... }
}
{% endhighlight %}  
优点: 简洁. (性能方面不再占有优势, 因为现在的JVM可以将静态工厂方法也优化为内联方式调用)  
缺点: 可以通过反射创建新的对象. (规避方法: 在构造函数中判断, 如果二次创建就抛出异常)  

2.静态工厂方法:  

{% highlight java linenos %}
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  public static Elvis getInstance(){ return INSTANCE; }
}
{% endhighlight %}
  优点:  
    - 灵活, 可以在不改变API的前提下, 改变为非单例模式(可以改为每一个线程返回一个不同的对象)  
    - 与泛型有关(item 27)  
  注意:  
    - 以上两种单例模式的实现方式, 如果implements Serializable, 需要给每个field添加`transient`限定, 还要增加一个 `readResolve()`方法(Chapter 11)  

3.枚举单例模式(目前最广泛使用的一种单例化方式)  

{% highlight java linenos %}
public enum Elvis {
  INSTANCE;
}
{% endhighlight %}

  优点:  
    - 自动提供序列化  
    - 即使通过反序列化或者反射也无法多次实例化  

## 四、使用private构造方法强化不可实例化对象  
> 私有默认构造方法,这没什么好说的

## 五、避免创建不必要的对象  
1. 关于String  
  - `String s = new String("stringeabc");`如果在一个循环中, 这种方式每次都创建一个String对象.  
  - `String s = "stringabc";`这种方式只会创建一个对象, 并且如果其他用到同样内容的字符串, 可以重用这个对象.  

2. 避免自动装箱  

3. 轻量级对象避免使用对象池. 现在的JVM的对轻量级对象的优化要比使用对象池好的多.  

## 六、消除不再使用的对象的引用  

1.出栈导致的内存泄露  

我们来看下面这段有内存泄露风险的代码:  
{% highlight java linenos %}
public class Stack{
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;

  public Stack(){
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }

  public void push(Object e){
    ensureCapacity();
    elements[size++] = e;
  }

  public Object pop(){
    if(size == 0){
      throw new EmptyStackException();
    }
    return elements[--size];
  }

  private void ensureCapacity(){
    if(elements.length == size){
      elements = Arrays.copyOf(elements, 2*size +1);
    }
  }
}
{% endhighlight %}  
无论你使用何种测试工具, 这段代码都能通过测试. 可是它确实存在内存泄露的风险, 你能找到么?  
如果找不到, 那就继续往下看吧...  
问题出在`pop()`中,正确的出栈姿势是获取对象后, 将elements[size]置空:  
{% highlight java linenos %}
public Object pop(){
  if(size == 0){
    throw new EmptyStackException();
  }
  Object result = elements[--size];
  elements[size] = null;//这是至关重要的一行代码, 没有这行代码,可能发生内存泄露
  return result;
}
{% endhighlight %}

为什么不将elements[size]置空会导致内存泄露呢?  
如果Stack因为存入过多对象导致容量增长, 然后又出栈导致实际使用容量减少, 那么就会剩余很大一部分未使用容量. 未使用容量中又有一部分容量存储了指向之前存入对象的引用. 这就会导致之前存入的对象,虽然已经出栈, 但无法被回收.  

2.缓存导致的内存泄露  

如果把object存入缓存, 却忘记从缓存中释放, 就会导致内存泄露.  
如果情况允许的话, 可以使用<a href="#weak_hash_map"><font color="blue">WeakHashMap</font></a>来管理缓存. 以object作为key存入WeakHashMap, WeakHashMap会在没有外部变量引用object时, 清除此object.  

3.注册listener和callback导致的内存泄露  

如果只register却没有deregister,就会导致内存泄露.  
但如果使用WeakHashMap的话, 可以不用deregister, WeakHashMap会在没有外部引用key时自动清除key.  


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

### <span id="build_pattern">5.Builder模式</span>  
以OkHttpClient为例,下面是OkHttpClient对象的创建方式:  
{% highlight java linenos %}
OkHttpClient client = new OkHttpClient.Builder()
                          .retryOnConnectionFailure(true)
                          .connectTimeout(10,TimeUnit.SECONDS)
                          .build();
{% endhighlight %}  
OkHttpClient builder模式部分代码:  
{% highlight java linenos %}
public class OkHttpClient{

  private OkHttpClient(Builder builder) {
    this.dispatcher = builder.dispatcher;
    this.retryOnConnectionFailure = builder.retryOnConnectionFailure;
    this.connectTimeout = builder.connectTimeout;
  }

  public static final class Builder {
    Dispatcher dispatcher;
    boolean retryOnConnectionFailure;
    int connectTimeout;
    ...//更多字段

    public Builder() {
      //默认值
      dispatcher = new Dispatcher();
      retryOnConnectionFailure = true;
      connectTimeout = 10_000;
    }

    public Builder retryOnConnectionFailure(boolean retryOnConnectionFailure) {
      this.retryOnConnectionFailure = retryOnConnectionFailure;
      return this;
    }

    public Builder connectTimeout(long timeout, TimeUnit unit) {
      if (timeout < 0) throw new IllegalArgumentException("timeout < 0");
      if (unit == null) throw new NullPointerException("unit == null");
      long millis = unit.toMillis(timeout);
      if (millis > Integer.MAX_VALUE) throw new IllegalArgumentException("Timeout too large.");
      if (millis == 0 && timeout > 0) throw new IllegalArgumentException("Timeout too small.");
      connectTimeout = (int) millis;
      return this;
    }

    public OkHttpClient build() {
      return new OkHttpClient(this);
    }

  }
}
{% endhighlight %}  

### <span id="telescoping_constructor_pattern">6.重叠构造模式</span>
{% highlight java linenos %}
public class OkHttpClient{
  private Dispatcher dispatcher;//必选参数
  private boolean retryOnConnectionFailure;//可选参数
  private int connectTimeout;//可选参数

  public OkHttpClient(Dispatcher dispatcher){
    this.dispatcher = dispatcher;
  }

  public OkHttpClient(Dispatcher dispatcher, boolean retryOnConnectionFailure){
    this.dispatcher = dispatcher;
    this.retryOnConnectionFailure = retryOnConnectionFailure;
  }

  public OkHttpClient(Dispatcher dispatcher, boolean retryOnConnectionFailure, int connectTimeout){
    this.dispatcher = dispatcher;
    this.retryOnConnectionFailure = retryOnConnectionFailure;
    this.connectTimeout = connectTimeout;
  }
}
{% endhighlight %}

### <span id="java_beans_pattern">7.JavaBean模式</span>  
这是一个典型的JavaBean模式的使用方法:  
{% highlight java linenos %}
OkHttpClient client = new OkHttpClient();
client.setDispatcher(new Dispatcher);
client.setRetryOnConnectionFailure(true);
client.setConnectTimeout(10_000);
{% endhighlight %}

JavaBean模式也导致了对象构建过程的不连续,和使用过程中的不稳定：  
所谓不连续, 就是在setXXX()之间可能执行其他方法.  
所谓不稳定, 就是在对象被使用的过程中,仍可以通过setXXX()改变对象的属性值.  

### <span id="weak_hash_map">8.WeakHashMap</span>  
> WeakHashMap，此种Map的特点是，当除了自身有对key的引用外，此key没有其他变量引用那么此map会自动丢弃此值  
{% highlight java linenos %}
public class Test {  
    public static void main(String[] args) throws Exception {  
        String a = new String("a");  
        Map weakmap = new WeakHashMap();  
        weakmap.put(a, "aaa");  
        a=null;  

        System.gc();  

        Iterator j = weakmap.entrySet().iterator();  
        while (j.hasNext()) {  
            Map.Entry en = (Map.Entry)j.next();  
            System.out.println("weakmap:"+en.getKey()+":"+en.getValue());  

        }  
    }  
}  
{% endhighlight %}  
上面这段代码,将对象a作为key分别存入WeakHashMap, 然后将a置为null.  
gc之后, WeakHashMap中却什么也没有, 因为除了WeakHashMap自身对a有引用外, 再无其他外部变量引用a, WeakHashMap就会将a回收.  



