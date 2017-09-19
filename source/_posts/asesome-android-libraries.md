---
title: 那些年，让我【为之一振】的库
date: 2017-06-11
tags: 
- Android开发
- 干货
categories: Android开发
thumbnail: https://ooo.0o0.ooo/2017/06/14/5940bc7ae03a8.png
---

这些库你可能都听说过，本文主要介绍让我感受到 `震惊` 的Android开源库，推荐给还没有用的人。

# ButterKnife

大名鼎鼎的ButterKnife，当初我烦死了每次都要重复写的 `TextView tv = (TextView) findViewById(R.id.text)` ，真是扎心了呢。当然还有这个，眼不见心不烦：

```java
view.setOnClickListener (new View.OnClickListener() {
  @Override
  public void onClick (View v) {
    // TODO
  }
});
```

当我发现可以用 `@BindView(R.id.text)` 和 `@OnClick(R.id.view)`实现上述~~恶心~~代码的时候，顿时惊呆了（X。这个库可谓是人人知晓了，非常有名，大幅提升了效率。同时，可以搭配 [android-butterknife-zelezny](https://github.com/avast/android-butterknife-zelezny) AndroidStudio插件，在代码的 `layout资源引用部分（如 R.layout.activity_main）` **打开生成菜单（我比较习惯 `alt+insert`）**，选择 `Generate Butterknife Injections` 随后就很明白啦！



地址：[https://github.com/JakeWharton/butterknife](https://github.com/JakeWharton/butterknife)

推荐一个很棒的教程：[http://dev.qq.com/topic/578753c0c9da73584b025875](http://dev.qq.com/topic/578753c0c9da73584b025875)

-------

# Android-Universal-Image-Loader

先说说我是怎么认识这个库的。以前做一些图片加载相关项目，一直没有想过缓存相关概念，直到发觉 `每次都重新加载，浪费资源`、`加载速度慢` 等问题，才想起了 `缓存`。然后从 `Sorcery图标包`（在此特别感谢Sorcery图标包，让我发现了很多很棒的库子）等项目中发现了这个库，缓存图片、加载图片都做得很简单。同时，也有一些别的图片加载框架，`Glide`、`Picasso` 等。



地址：[https://github.com/nostra13/Android-Universal-Image-Loader](https://github.com/nostra13/Android-Universal-Image-Loader)

-----

# OkHttp

我是从LeanCloud SDK中开始尝试使用OkHttp的，以前的***Connection就不提了啊（逃

一下子才发现网络请求竟如此简单，如我需要一个Get请求，直接定义一个 `Request`，通过 `OkHttpClient` 就拿到了 `Response` ，太简单了！但是我也没有仔细使用过OkHttp，就不多阐述了。到后面还有比它更简单的（



地址：[https://github.com/square/okhttp](https://github.com/square/okhttp)

-----

# RxJava

在此之前，我都是用 `AsyncTask` 做异步操作的，虽然也很不错，但是为了体验一下风靡的RxJava，就试了一下。感觉到代码简洁了不少（与其说简洁，不如说 **看起来更舒服了，也就是`可读性更佳`**）。 我们来一起对比两段代码，其作用是从网络上加载一个图片到ImageView，具体网络请求（图片加载）就不写了：

~~PS: 基本上手打的，有Typo见谅~~

AsyncTask

```Java
public class LoadImageTask extends AsyncTask<String, Integer, Bitmap> {
  private ImageView mImageView;
  public LoadImageTask (ImageView imageView) {
    mImageView = imageView;
  }
  
  @Override
  protected Bitmap doInBackground (String... args) {
    // TODO: Load image from network
    return bitmap;
  }
  
  @Override
  protected void onPostExecute (Bitmap bitmap) {
    mImageView.setImageBitmap(bitmap);
  }
}

// MainActivity -> 调用之
public class MainActivity extends Activity {
  private LoadImageTask mTask;
  @Override
  public void onCreate (Bundle savedInstanceState) {
    super.onCreate (savedInstanceState);
    // Bind Views
    
    // 到这里该执行任务了
    mTask = new LoadImageTask (imageView);
    mTask.execute ("某URL [doge原谅ta]");
  }
  
  // 在onDestroy的时候取消任务
  @Override
  public void onDestroy () {
    if (mTask != null && !mTask.isCancelled()) {
      mTask.cancel(true);
    }
    super.onDestroy();
  }
}
```

再看看RxJava1实现：

```java
public class MainActivity extends Activity {
  private Subscription mLoadSubscription;
  @Override
  public void onCreate (Bundle savedInstanceState) {
    super.onCreate (savedInstanceState);
    // Bind Views
    
    // 创建Observable
    Observable<Bitmap> observable = Observable.create(new Observable.OnSubscribe<Bitmap>() {
            @Override
            public void call(Subscriber<? super Bitmap> subscriber) {
                // TODO: Load image from network
              	subscriber.onNext(bitmap);
              	subscriber.onCompeleted();
            }
        });
    // 到这里该执行任务了
    mLoadSubscription = observable.subscribeOn(Schedulers.io())
      .observeOn(AndroidSchedulers.mainThread())
      .subscribe(new Action1<Bitmap>() {
        @Override
        public void call (Bitmap bitmap) {
          imageView.setImageBitmap(bitmap);
        }
      }, new Action1<Throwable>() {
        @Override
        public void call (Throwable e) {
          // 加载失败回掉，太贴心了
        }
      });
  }
  
  // 在onDestroy的时候取消订阅
  @Override
  public void onDestroy () {
    if (mLoadSubscription != null && !mLoadSubscription.isUnsubscribed()) {
      mLoadSubscription.unsubscribe();
    }
    super.onDestroy();
  }
}
```

当时尤为喜欢 `onNext`、`onCompeleted`、`onError` 和 `subscribeOn`、`observeOn`，觉得真是贴心啊。



地址：[https://github.com/ReactiveX/RxJava](https://github.com/ReactiveX/RxJava)

推荐教程：[https://gank.io/post/560e15be2dca930e00da1083](https://gank.io/post/560e15be2dca930e00da1083)

-----

# Retrofit

没错就是它！以前没有怎么用过RestAPI，直到后来，才发现每个请求都用OkHttp `Request`再 `Response` 太恶心了。然后发现了 Retrofit 这个好东西，简直大爱。 把所有API都写进接口，要什么请求/参数直接 `注解` ，然后 **`retrofit.create()`** 竟完成了上述几乎所有操作！！！~~男人看了沉默，女人看了流泪~~，我当时真心觉得 【还能这么玩】，太厉害了。比如，我们需要一些请求：

```java
public interface APIInterface {
  @GET("xxx")
  Call<结果Bean> getXXX();
  
  @POST("send")
  Call<AAA> send (@Query("param") String param, @Body("body") String body);
}

// 然后造一个APIManager
public class APIManager {
  private APIInterface api;
  public APIManager () {
    Retrofit retrofit = new Retrofit.Builder()
      .baseUrl("每个请求地址前面都会自动加上这个地址，真贴心")
      .build();
    api = retrofit.create(APIInterface.class);
  }
  
  public Call<XXX> getXXX () {
    return api.getXXX();
  }
  
  public Call<AAA> send (String param, String body) {
    return api.send (param, body);
  }
}
```



地址：[https://github.com/square/retrofit](https://github.com/square/retrofit)

Retrofit+RxJava+Gson简直太爽了，吐血推荐：[https://gank.io/post/56e80c2c677659311bed9841](https://gank.io/post/56e80c2c677659311bed9841)

-----

# Gson

并不代表 `Gson` 这一个库，而是 `Json序列化`。 当我还在苦苦地这样写时：

```java
JSONBean bean = new JSONBean();
bean.setValue(jsonObject.getString("value"));
bean.setA(jsonObject.getInt("A"));
// 默哀N行 get
return bean;
```

才得知可以直接 `反序列化Json` ：

```java
public class JSONBean {
  @SerializedName("value")
  private String value;
  @SerializedName("A")
  private int A;
  // getter、setter
}

public JSONBean parse (String json) {
  return new Gson().fromJson(json, JSONBean.class);
}
```

厉害了呢~ 同时还有许多Json序列化库：`Fastjson`、`LoganSquare`等。



链接：[https://github.com/google/gson](https://github.com/google/gson)

推荐一系列教程：你真的会用Gson吗 [http://www.jianshu.com/p/e740196225a4](http://www.jianshu.com/p/e740196225a4)

一个一键生成模型的AS插件，支持多种Json库和Kotlin，非常棒：[RoboPOJOGenerator](https://github.com/robohorse/RoboPOJOGenerator)

----

# AutoValue

我们先来看一个Model：

```java
public class CoolGay {
  private String id;
  private String username;
  private String email;
  private Date registerTime;
  private int replyCount;
  private String description;
  // 省略N个属性
  
  // getter、setter
  public String getId() {
    return this.id;
  }
  
  public void setId (String id) {
    this.id = id;
  }
  // 省略N个getter、setter
}
```

虽然 getter 和 setter 可以用IDE一键生成，但是还是看着很不舒服了吧。然后隆重介绍这个库：AutoValue。 Google出的，在Google的不少APP里面都能见到（详情去看开放源代码许可），讲个笑话，我一直以为这和 `AndroidAuto（开车的）` 有什么关系呢2333。下面把刚才的那个模型用AutoValue生成一番：

```java
@AutoValue
public class CoolGay {
  // 天哪，一行代码就解决了问题？！
  public abstract String id();
  public abstract String username();
  public abstract String email();
  public abstract Date registerTime();
  public abstract int replyCount();
  public abstract String description();
  
  public static CoolGay create(String id, String username, String email, 
                              Date registerTime, int replyCount, String description){
    // 这个 AutoValue_CoolGay 是 AutoValue 自动编译时生成的
    new AutoValue_CoolGay(id, username, email, registerTime, replyCount, String description);
  }
}
```

还有 `equals()/hashcode()/tostring()` 甚至是 `Builder`、`Gson`、`Parcelable（非官方）` 生成，写模型太简单了。



地址：[https://github.com/google/auto](https://github.com/google/auto)

教程推荐：[http://tedyin.me/2016/04/11/auto-value/](http://tedyin.me/2016/04/11/auto-value/)  以及 [https://juejin.im/entry/57d2552d128fe10055150c9c](https://juejin.im/entry/57d2552d128fe10055150c9c)

-----

我目前还没有详细了解它们的源码，以后要仔细学习的。这些库给我们开发提供了很大便利，支持他们，大家一定要写上开放源代码许可，这是对作者的支持。同时忠告一下，一些小项目也没必要非得用这些库，还是 `能不就不`，尽量不重复早轮子，但也不要让程序过于臃肿。