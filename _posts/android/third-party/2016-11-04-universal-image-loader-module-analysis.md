---
layout: single
title: Universal Image Loader 模块分析
excerpt: "Universal Image Loader 模块分析"
author_profile: false
sidebar:
  nav: "android"
categories: android third-party
---
# Android Universal Image Loader 源码分析   
> 基于 Universal Image Loader 1.9.5   

## 使用方法  
首先, 我们先来看一下它的使用法方法, 大致分为如下四步:  
1.配置ImageLoaderConfiguration(全局配置一次)  
{% highlight java linenos %}
File cacheDir = StorageUtils.getCacheDirectory(context);
ImageLoaderConfiguration config = new ImageLoaderConfiguration.Builder(context)
        .memoryCacheExtraOptions(480, 800) // default = device screen dimensions
        .diskCacheExtraOptions(480, 800, null)
        .taskExecutor(...)
        .taskExecutorForCachedImages(...)
        .threadPoolSize(3) // default
        .threadPriority(Thread.NORM_PRIORITY - 2) // default
        .tasksProcessingOrder(QueueProcessingType.FIFO) // default
        .denyCacheImageMultipleSizesInMemory()
        .memoryCache(new LruMemoryCache(2 * 1024 * 1024))
        .memoryCacheSize(2 * 1024 * 1024)
        .memoryCacheSizePercentage(13) // default
        .diskCache(new UnlimitedDiskCache(cacheDir)) // default
        .diskCacheSize(50 * 1024 * 1024)
        .diskCacheFileCount(100)
        .diskCacheFileNameGenerator(new HashCodeFileNameGenerator()) // default
        .imageDownloader(new BaseImageDownloader(context)) // default
        .imageDecoder(new BaseImageDecoder()) // default
        .defaultDisplayImageOptions(DisplayImageOptions.createSimple()) // default
        .writeDebugLogs()
        .build();
{% endhighlight %}  
2.初始化ImageLoader(全局初始化一次)  
{% highlight java linenos %}
ImageLoader.getInstance().init(config.build());
{% endhighlight %}  
3.配置DisplayImageOptions(可选,局部配置)  
{% highlight java linenos %}
DisplayImageOptions options = new DisplayImageOptions.Builder()
        .showImageOnLoading(R.drawable.ic_stub) // resource or drawable
        .showImageForEmptyUri(R.drawable.ic_empty) // resource or drawable
        .showImageOnFail(R.drawable.ic_error) // resource or drawable
        .resetViewBeforeLoading(false)  // default
        .delayBeforeLoading(1000)
        .cacheInMemory(false) // default
        .cacheOnDisk(false) // default
        .preProcessor(...)
        .postProcessor(...)
        .extraForDownloader(...)
        .considerExifParams(false) // default
        .imageScaleType(ImageScaleType.IN_SAMPLE_POWER_OF_2) // default
        .bitmapConfig(Bitmap.Config.ARGB_8888) // default
        .decodingOptions(...)
        .displayer(new SimpleBitmapDisplayer()) // default
        .handler(new Handler()) // default
        .build();
{% endhighlight %}  
4.加载图片  
下载图片, 转换为Bitmap, 显示在ImageView(或者任何实现ImageAware的类)
{% highlight java linenos %}
imageLoader.displayImage(imageUri, imageView, options, new ImageLoadingListener() {
  @Override
  public void onLoadingStarted(String imageUri, View view) {
      ...
  }
  @Override
  public void onLoadingFailed(String imageUri, View view, FailReason failReason) {
      ...
  }
  @Override
  public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
      ...
  }
  @Override
  public void onLoadingCancelled(String imageUri, View view) {
      ...
  }
}, new ImageLoadingProgressListener() {
  @Override
  public void onProgressUpdate
  (String imageUri, View view, int current, int total) {
      ...
  }
});
{% endhighlight %}  
下载图片, 转换为Bitmap, 并将Bitmap返回到CallBack中  
{% highlight java linenos %}
ImageSize targetSize = new ImageSize(80, 50); // result Bitmap will be fit to this size
imageLoader.loadImage(imageUri, targetSize, options, new SimpleImageLoadingListener() {
    @Override
    public void onLoadingComplete(String imageUri, View view, Bitmap loadedImage) {
        // Do whatever you want with Bitmap
    }
});
{% endhighlight %}  
下载图片, 转换为Bitmap, 并同步返回Bitmap
{% highlight java linenos %}
ImageSize targetSize = new ImageSize(80, 50); // result Bitmap will be fit to this size
Bitmap bmp = imageLoader.loadImageSync(imageUri, targetSize, options);
{% endhighlight %}  
## 下载&展示 任务流  
![UIL任务流](http://odxsluszm.bkt.clouddn.com/UIL_Flow.png)  

## 模块分析  
1. ImageLoaderConfiguration(全局配置)  
2. DisplayImageOptions(局部配置)  
3. ImageLoader  
  显示图片  
4. ProcessAndDisplayImageTask implements Runnable  
  处理并显示图片(已获取Bitmap)  
5. LoadAndDisplayImageTask  implements Runnable  
  加载并显示图片(为获取Bitmap)  
6. DisplayImageTask implements Runnable  
  显示图片(Process 和 Load最终都通过它显示图片)  
7. BitmapDisplayer  
  真正显示图片的类, 不同的子类有不同的功能(如,显示圆角图片,显示图片时附带动画效果)  
8. ImageLoaderEngine  
  三个Excutor:  
    从DiskCache或MemoryCache获取Bitmap的Excetor  
    从网络Bitmap的Excutor  
    任务分发的Excutor(根据情况分发给另外两个Excutor)  
9. ImageDownloader  
  图片下载的分发器:  
  Network获取  
  File获取  
  Assets获取  
  Drawable获取(就是R.mipmap.id)  
  Content获取(未知)  

## 流程图  
![UIL流程图](http://odxsluszm.bkt.clouddn.com/Universal-Image-Loader.png)

