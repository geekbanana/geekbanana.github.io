---
layout: single
title: 走向灵活软件之路--面向对象的六大原则
excerpt: "设计模式 面向对象 六大原则"
author_profile: false
sidebar:
  nav: "design-patterns"
categories: design-patterns
---

## 1.单一职责原则(Single Responsibility Principle, SRP)
- 一个函数所做的事情应该尽量单一
- 一个类应该是一组相关性很高的函数、数据的封装

假设我们有一个图片加载器:
{% highlight java linenos %}
/**
 * 图片加载类
 */
public class ImageLoader {
    // 图片缓存
    LruCache<String, Bitmap> mImageCache;
    // 线程池,线程数量为CPU的数量
    ExecutorService mExecutorService = Executors.newFixedThreadPool(Runtime.getRuntime()
            .availableProcessors());
    private Handler mUiHandler = new Handler() ;

    public ImageLoader() {
        initImageCache();
    }

    private void initImageCache() {
        // 计算可使用的最大内存
        final int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
        // 取4分之一的可用内存作为缓存
        final int cacheSize = maxMemory / 4;
        mImageCache = new LruCache<String, Bitmap>(cacheSize) {

            @Override
            protected int sizeOf(String key, Bitmap bitmap) {
                return bitmap.getRowBytes() * bitmap.getHeight() / 1024;
            }
        };
    }

    private void updateImageView(final ImageView imageView, final Bitmap bitmap) {
		mUiHandler.post(new Runnable() {

			@Override
			public void run() {
				 imageView.setImageBitmap(bitmap); ;
			}
		});
	}

    public void displayImage(final String url, final ImageView imageView) {
        imageView.setTag(url);
        mExecutorService.submit(new Runnable() {

            @Override
            public void run() {
                Bitmap bitmap = downloadImage(url);
                if (bitmap == null) {
                    return;
                }
                if (imageView.getTag().equals(url)) {
                	updateImageView(imageView, bitmap) ;
                }
                mImageCache.put(url, bitmap);
            }
        });
    }

    public Bitmap downloadImage(String imageUrl) {
        Bitmap bitmap = null;
        try {
            URL url = new URL(imageUrl);
            final HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            bitmap = BitmapFactory.decodeStream(conn.getInputStream());
            conn.disconnect();
        } catch (Exception e) {
            e.printStackTrace();
        }

        return bitmap;
    }
}
{% endhighlight %}
从代码可以看出,这个ImageLoader主要包含三个功能: 图片展示, 图片下载, 图片缓存. 这就不符合我们的**单一职责原则**.
当我们要更换缓存策略或者优化图片下载时, 就需要修改ImageLoader这个类, 这就增大了升级维护的成本.
毕竟谁也不敢保证, 在这揉成一坨的代码中修改缓存和下载, 不会对ImageLoader其他功能造成破坏.
下面我们将图片缓存从ImageLoader中分离出来(图片下载同样也需分离,这里仅以图片缓存为例).

图片缓存ImageCache:
{% highlight java linenos %}
public class ImageCache {
    // 图片缓存
    LruCache<String, Bitmap> mImageCache;

    public ImageCache() {
        initImageCache();
    }

    private void initImageCache() {
        // 计算可使用的最大内存
        final int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
        // 取4分之一的可用内存作为缓存
        final int cacheSize = maxMemory / 4;
        mImageCache = new LruCache<String, Bitmap>(cacheSize) {

            @Override
            protected int sizeOf(String key, Bitmap bitmap) {
                return bitmap.getRowBytes() * bitmap.getHeight() / 1024;
            }
        };
    }

    public void put(String url, Bitmap bitmap) {
        mImageCache.put(url, bitmap) ;
    }

    public Bitmap get(String url) {
        return mImageCache.get(url) ;
    }
}
{% endhighlight %}

图片加载ImageLoader:
{% highlight java linenos %}
public class ImageLoader {
    // 图片缓存
    ImageCache mImageCache = new ImageCache();
    // 线程池,线程数量为CPU的数量
    ExecutorService mExecutorService = Executors.newFixedThreadPool(Runtime.getRuntime()
            .availableProcessors());
    private Handler mUiHandler = new Handler() ;

    public void displayImage(final String url, final ImageView imageView) {
        Bitmap bitmap = mImageCache.get(url);
        if (bitmap != null) {
            imageView.setImageBitmap(bitmap);
            return;
        }

        imageView.setTag(url);
        mExecutorService.submit(new Runnable() {

            @Override
            public void run() {
                Bitmap bitmap = downloadImage(url);
                if (bitmap == null) {
                    return;
                }
                if (imageView.getTag().equals(url)) {
                	updateImageView(imageView, bitmap) ;
                }
                mImageCache.put(url, bitmap);
            }
        });
    }

    private void updateImageView(final ImageView imageView, final Bitmap bitmap) {
		mUiHandler.post(new Runnable() {

			@Override
			public void run() {
				 imageView.setImageBitmap(bitmap); ;
			}
		});
	}

    public Bitmap downloadImage(String imageUrl) {
        Bitmap bitmap = null;
        try {
            URL url = new URL(imageUrl);
            final HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            bitmap = BitmapFactory.decodeStream(conn.getInputStream());
            conn.disconnect();
        } catch (Exception e) {
            e.printStackTrace();
        }

        return bitmap;
    }
}
{% endhighlight %}
这样, ImageCache只做缓存相关的事情, ImageLoader只做显示相关的功能(如果下载也分离出来的话),
这就符合了**单一职责原则**, 此时更换缓存策略, 就无需在ImageLoader中修改了.

## 2.开闭原则（Open Close Principle, OCP）
软件中的对象(类, 模块, 函数)对扩展是开放的,对修改是封闭的.
换句换说, 在软件升级维护的过程中, 应该尽量通过继承, 依赖注入等方式修改, 而不是直接修改原有代码.
还是以ImageLoader为例, 假设我们现在需要一个硬盘缓存DiskCache(那么ImageCache更名为MemoryCache更为贴切):
{% highlight java linenos %}
public class DiskCache {
    static String cacheDir = "sdcard/cache/";

    // 从缓存中获取图片
    public Bitmap get(String url) {
        return BitmapFactory.decodeFile(cacheDir + url);
    }

    // 将图片缓存到内存中
    public void put(String url, Bitmap bmp) {
        FileOutputStream fileOutputStream = null;
        try {
            fileOutputStream = new FileOutputStream(cacheDir + url);
            bmp.compress(CompressFormat.PNG, 100, fileOutputStream);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            CloseUtils.closeQuietly(fileOutputStream);
        }
    }
}
{% endhighlight %}
同时ImageLoader要加入DiskCache:
{% highlight java linenos %}
public class ImageLoader {
    // 内存缓存
    MemoryCache mMemoryCache = new MemoryCache();
    // sd卡缓存
    DiskCache mDiskCache = new DiskCache();

    // 使用sd卡缓存
    boolean isUseDiskCache = false;


    public void displayImage(final String url, final ImageView imageView) {
        Bitmap bmp = null;
        if (isUseDiskCache) {
            bmp = mDiskCache.get(url);
        } else {
            bmp = mMemoryCache.get(url);
        }

        if ( bmp != null ) {
            imageView.setImageBitmap(bmp);
            return;
        }

        //没有缓存, 则提交给线程池下载
    }


    public void useDiskCache(boolean useDiskCache) {
        isUseDiskCache = useDiskCache ;
    }
}
{% endhighlight %}
此时, 我们为了增加DiskCache, 修改了ImageLoader中的代码, 你可能会觉得:"这也没修改多少代码啊! So easy"
假设我们现在又需要一个Memory+Disk的双缓存, 那么我们势必要写一个DoubleCache,
并且再次修改ImageLoader中的代码将其加入. 你可能会觉得这同样没有多少工作量.

那我假设在实际工作中, 你写了一个带内存缓存的ImageLoader. 然后产品经理说不要MemoryCache, 只要DiskCache即可.
你噼里啪啦一顿猛敲, 一个DiskCache闪亮登场, 然后修改ImageLoader代码,将DiskCache加入其中,还没来得及调试, 产品经理又告诉你,还是双缓存体验要好一些.
你二话不说, 低头又是一顿猛敲, 一个DoubleCache完美出炉, 再次修改ImageLoader代码将DoubleCache加入.这次比上一次要好一些, 产品经理给了你调试的时间.
然并卵, 你刚刚调试完毕, 准备喝杯咖啡, 产品经理告诉你:"小马, 我想了一下, 我们这个图片更新频率比较高, 就不要做缓存了..."
想必此时你的心中已经有万只羊驼在奔腾, 但你还是理智的做了一个深呼吸, 默默的低头注释代码...
然后, 正当你沉浸在刚才的愤怒中无法自拔之时, 产品经理有来了:"Sorry, 小马, 为了更好的客户体验, 我觉得需要在没有网络的时候仍然可以给客户展示图片,你还是加上DiskCache吧..."
此时, 你拿起了杯子径直走向了茶水间... 因为你需要喝一杯静静... 否则你体内的洪荒之力将会冲破封印... 后果, 不堪设想...

如果说这些你都能忍, 那么试想这样一种场景. 你觉得自己写的这个ImageLoader还不错, 遂将之发布到开源社区.
网友的需求千变万化, 部分网友希望能使用自定义的缓存策略, 那你又该怎么办?
显然这种方式无法满足自定义缓存的需求. 如果要使用自定义缓存, 就只能去修改你的ImageLoade, 这显然违反了**开闭原则**
要解决这个问题, 我们可以使用**依赖注入**的方式给ImageLoader注入缓存.

下面先看使用依赖注入的UML图:
![OCP](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter01_OCP.png)

所有的缓存都实现ImageCache接口, ImageLoader持有的是ImageCache类型的引用.
这样, 自定义的缓存策略只要实现ImageCache, 然后通过ImageLaoder的setImageCache(ImageCache)
即可在不修改ImageLoader代码的情况下, 实现自定义缓存策略的更换.

注入缓存实现的ImageLoader:
{% highlight java linenos %}
public class ImageLoader {
    // 图片缓存, 默认为内存缓存
    ImageCache mImageCache = new MemoryCache();
    // 线程池,线程数量为CPU的数量
    ExecutorService mExecutorService = Executors.newFixedThreadPool(Runtime.getRuntime()
            .availableProcessors());
    private Handler mUiHandler = new Handler() ;

    //缓存的依赖注入
    public void setImageCache(ImageCache cache) {
        mImageCache = cache;
    }

    public void displayImage(String imageUrl, ImageView imageView) {
        Bitmap bitmap = mImageCache.get(imageUrl);
        if (bitmap != null) {
            imageView.setImageBitmap(bitmap);
            return;
        }

        submitLoadRequest(imageUrl, imageView);
    }

    private void submitLoadRequest(final String imageUrl, final ImageView imageView) {
        imageView.setTag(imageUrl);
        mExecutorService.submit(new Runnable() {

            @Override
            public void run() {
                Bitmap bitmap = downloadImage(imageUrl);
                if (bitmap == null) {
                    return;
                }
                if (imageView.getTag().equals(imageUrl)) {
                    updateImageView(imageView, bitmap);
                }
                mImageCache.put(imageUrl, bitmap);
            }
        });
    }

    private void updateImageView(final ImageView imageView, final Bitmap bitmap) {
		mUiHandler.post(new Runnable() {

			@Override
			public void run() {
				 imageView.setImageBitmap(bitmap); ;
			}
		});
	}

    public Bitmap downloadImage(String imageUrl) {
        Bitmap bitmap = null;
        try {
            URL url = new URL(imageUrl);
            final HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            bitmap = BitmapFactory.decodeStream(conn.getInputStream());
            conn.disconnect();
        } catch (Exception e) {
            e.printStackTrace();
        }

        return bitmap;
    }
}
{% endhighlight %}

## 3. 里氏替换原则(Likvos Substitution Principle, LSP)
所有引用基类的地方,必须能够同名的使用其子类对象.
通俗的讲就是, 所有引用基类的地方, 替换为其子类对象, 可以正常运行, 不会报错.

以Android中Window与View的关系为例
![Window_View](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter01_LSP.png)

代码实现:
{% highlight java linenos %}
/**
 * 简单的窗口类
 * @author mrsimple
 *
 */
public class Window {
	public void show(View child)  {
		// 绘制各种视图
		child.draw();
	}
}

/**
 * 视图基类
 * @author mrsimple
 *
 */
public abstract class View {
	public abstract void draw() ;

    public void measure(int width, int height){
        // 测量视图大小
    }

}
{% endhighlight %}
Window依赖于View, 而View有多个具体实现类, 任何继承自View的类都可以设置给Window的show(View)方法,
这就是**里氏替换原则**

## 4.依赖倒置原理(Dependence Inversion Principle, DIP)
- 高层模块不应依赖低层模块, 两者都应该依赖低层的抽象
- 抽象不应该依赖细节
- 细节应该依赖抽象

解释一下, 所谓抽象是指抽象类或接口. 细节是指抽象的具体实现类.
"高层模块不应依赖低层模块, 两者都应该依赖低层的抽象", 这句话可以这样理解:
类A(高层)调用类B(低层)的方法,就需要拿到B的对象. 但A中不直接持有B类型对象的引用, 而是持有B的接口I类型的引用(I i = new B()).
这样A就不会与B(细节)直接发生依赖关系, 而是与B(细节)的接口I(抽象)发生依赖.
这样做的优点是: 无论把细节换成B, C, 还是D,(B,C,D都实现I), 都无需改动A, 增强了程序的扩展性.

以ImageLoader为例, 我们之所以写为setImageCache(ImageCache)而不是setImageCache(MemoryCache),也不是setImageCache(DiskCache),
这样做的好处是, 不论我们传入的参数是MemoryCache, DiskCache, DoubleCache还是XxxCache, 只要这个XxxCache实现了ImageCache这个接口,
都可以与ImageLoader无缝对接.

## 5.接口隔离原则(Interface Segregation Principle, ISP)
类的依赖关系应该建立在最小的接口上.  
**接口隔离原则**是将庞大的,雍容的接口拆分成更小的接口.

例如
{% highlight java linenos %}
  FileOutputStream fos = null;
  try{
    fos = new FileOutputStream(dir);
    ...
  }cache(IOException e){
    e.printStachTrace();
  }finally{
    if(fos!=null){
      try{
        fos.close();
      }cache(IOException e){
        e.printStackTrace();
      }
    }
  }
{% endhighlight %}

使用流的时候, 通常需要在finally中写一段代码来关闭流.
以FileOutputStream为例, 它的close方法不是继承自OutputStream, 而是实现了Closeable接口.
因此我们就可以写一个通用的工具类来关闭所有的流(而不单单关闭FileOutputStream):

{% highlight java linenos %}
/**
 * 关闭工具类
 */
public abstract class CloseUtils {
    /**
     * 关闭Closeable对象
     *
     * @param closeable
     */
    public static void closeQuietly(Closeable closeable) {
        if (null != closeable) {
            try {
                closeable.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
{% endhighlight %}

以上5个原则, **单一职责原则**, **开闭原则**,  **里氏替换原则**,  **接口隔离原则**,  **依赖倒置原则** 称之为**SOLID**原则.

## 6.迪米特原则(Law of Demeter, LOD)
**迪米特原则**也称**最少知识原则**(Least Knowledge Principle).
一个对象应该对其他对象有最少的了解. 通俗的讲就是, 一个类应该在实现需求的前提下依赖最少的类.

我们以生活中租房这个事件为例, 租户(Tenant)通过中介(Mediator)找到合适的房子(Room). 那我们可以写出如下代码:
{% highlight java linenos %}
/**
 * 房间
 */
public class Room {
    public float area;
    public float price;

    public Room(float area, float price) {
        this.area = area;
        this.price = price;
    }

    @Override
    public String toString() {
        return "Room [area=" + area + ", price=" + price + "]";
    }

}

/**
 * 中介
 */
public class Mediator {
    List<Room> mRooms = new ArrayList<Room>();

    public Mediator() {
        for (int i = 0; i < 5; i++) {
            mRooms.add(new Room(14 + i, (14 + i) * 150));
        }
    }

    public List<Room> getAllRooms() {
        return mRooms;
    }

    public void rentOut(Room room) {
        mRooms.remove(room);
    }
}

/**
 * 租户
 */
public class Tenant {
    public void rentRoom(float roomArea, float roomPrice,  Mediator mediator) {
        List<Room> rooms = mediator.getAllRooms();
        for (Room room : rooms) {
            if (isSuitable(roomArea, roomPrice, room)) {
                System.out.println("租到房间啦! " + room);
                break;
            }
        }
    }

    // 租金要小于等于指定的值,面积要大于等于指定的值
    private boolean isSuitable(float roomArea, float roomPrice, Room room) {
        return room.price <= roomPrice
                && room.area >= roomArea;
    }
}
{% endhighlight %}
从上面代码可以看出, Tenant不仅依赖Mediator, 还依赖Room. 租户只是想租个房间而已,
将所有的检测条件都放到Tenant中就弱化了Mediator的功能. 由下面的UML图也可看出, 这种写法耦合度较高,
当Room有修改时, Tenant也需要修改.
![LOD_高耦合](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter01_LOD.png)

如果我们将租房的检测条件放到Mediator中
{% highlight java linenos %}
/**
 * 中介
 */
public class Mediator {
    List<Room> mRooms = new ArrayList<Room>();

    public Mediator() {
        for (int i = 0; i < 5; i++) {
            mRooms.add(new Room(14 + i, (14 + i) * 150));
        }
    }

    public Room rentOut(float area, float price) {
        for (Room room : mRooms) {
            if (isSuitable(area, price, room)) {
                return room;
            }
        }
        return null;
    }

    private boolean isSuitable(float roomArea, float roomPrice, Room room) {
        return room.price <= roomPrice
                && room.area >= roomArea;
    }
}

/**
 * 租户
 */
public class Tenant {
    public void rentRoom(float roomArea, float roomPrice, Mediator mediator) {
        System.out.println("租到房啦   " + mediator.rentOut(roomArea, roomPrice));
    }
}
{% endhighlight %}
此时的UML图如下
![LOD_低耦合](http://oi63pt0qt.bkt.clouddn.com/asdp_chapter01_LOD_refactor.png)

可以看出, 重构后Tenant之依赖Mediator, 耦合度降低.

## 总结
总结几个字, 一个好的软件应该具有: 高扩展, 高内聚, 低耦合.
