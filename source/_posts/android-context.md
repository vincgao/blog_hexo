---
title: Android Context笔记和Application误用
tags:
- Android
categories:
- Android
comments: true
date: 2016-02-29 18:11:19
updated:
desc: Android Context笔记和Application误用
---

今天看到郭霖blog有关[Android Context完全解析](http://blog.csdn.net/guolin_blog/article/details/47028975)的文章，感觉醍醐灌顶，特此记下笔记。

# Context
Context的继承结构：
![](http://7xp4nq.com1.z0.glb.clouddn.com/image/blog/20151022212109519.png)
所以，Context一共有Application、Activity和Service三种类型，因此一个应用程序中Context数量有：

```
Context数量 = Activity数量 + Service数量 + 1
```

`getApplication()`只有在Activity和Service中才能调用的到。
在一些其它的场景，比如BroadcastReceiver中也想获得Application的实例时，可以借助`getApplicationContext()`方法。`getApplicationContext()`方法的作用域会更广一些，任何一个Context的实例，只要调用`getApplicationContext()`方法都可以拿到我们的Application对象。
例：

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        MyApplication myApp = (MyApplication) getApplication();
        Log.d("TAG", "getApplication is " + myApp);
        Context appContext = getApplicationContext();
        Log.d("TAG", "getApplicationContext is " + appContext);
    }
}
```

```java
public class MyReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        MyApplication myApp = (MyApplication) context.getApplicationContext();
        Log.d("TAG", "myApp is " + myApp);
    }
}
```

# 使用Application的问题
Android developer API文档中关于[Application](http://developer.android.com/intl/zh-cn/reference/android/app/Application.html)的介绍中写到：

> There is normally no need to subclass Application. In most situation, static singletons can provide the same functionality in a more modular way. If your singleton needs a global context (for example to register broadcast receivers), the function to retrieve it can be given a Context which internally uses Context.getApplicationContext() when first constructing the singleton.

Application中方法的执行顺序如下图所示：

![](http://7xp4nq.com1.z0.glb.clouddn.com/image/blog/20151108174114045.png)

Application中在`onCreate()`方法里去初始化各种全局的变量数据是一种比较推荐的做法。


```java
public class MyApplication extends Application {

    private static MyApplication app;

    public static MyApplication getInstance() {
        if (app == null) {
            app = new MyApplication();
        }
        return app;
    }
}
```
这种写法是大错特错！因为Application属于系统组件，系统组件的实例是要由系统来去创建的，如果这里自己去new一个MyApplication的实例，它就只是一个普通的Java对象而已，而不具备任何Context的能力。
谨记，Application全局只有一个，它本身就已经是单例了，无需再用单例模式去为它做多重实例保护了，提供一个获取MyApplication实例的方法，比较标准的写法：

```java
public class MyApplication extends Application {

    private static MyApplication app;

    public static MyApplication getInstance() {
        return app;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        app = this;
    }
}
```
`getInstance()`方法可以照常提供，但是里面不要做任何逻辑判断，直接返回app对象即可。在onCreate()方法中我们将app对象赋值成this，this就是当前Application的实例，那么app也就是当前Application的实例了。


