---
title: 解决ScrollView嵌套ListView造成滑动冲突的问题
tags:
- Android
categories:
- Android
comments: true
date: 2016-02-29 18:26:50
updated:
desc: 解决ScrollView嵌套ListView造成滑动冲突的问题
---

参考链接：
http://blog.csdn.net/hanhailong726188/article/details/46136569

今天重新会看项目中解决ScrollView嵌套ListView造成滑动冲突的办法，终于理解了原理，特此记下，网上有多种解决办法，先只说一种目前项目中所采用的，日后再补齐其他方法。

不过，最好的解决办法应该是：
> 避免嵌套

如果要实现类似效果，可以利用ListView或RecyclerView里的**type**。

现在说下如何解决：

```java
public class ListViewForScrollView extends ListView {

    public ListViewForScrollView(Context context) {
        super(context);
    }

    public ListViewForScrollView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public ListViewForScrollView(Context context, AttributeSet attrs,
            int defStyle) {
        super(context, attrs, defStyle);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int expandSpec = MeasureSpec.makeMeasureSpec(Integer.MAX_VALUE >> 2,
                MeasureSpec.AT_MOST);
        super.onMeasure(widthMeasureSpec, expandSpec);
    }
}
```

解释：
重载`onMeasure()`函数，在测量ListView的高度时，传入最大的临界值高度，需要多高，父控件（ScrollView）就给分配多高，使ListView不需要自己滑动。

参考：

```java
public static class MeasureSpec {
    private static final int MODE_SHIFT = 30;
    private static final int MODE_MASK  = 0x3 << MODE_SHIFT;

    /**
     * Measure specification mode: The parent has not imposed any constraint
     * on the child. It can be whatever size it wants.
     */
    public static final int UNSPECIFIED = 0 << MODE_SHIFT;

    /**
     * Measure specification mode: The parent has determined an exact size
     * for the child. The child is going to be given those bounds regardless
     * of how big it wants to be.
     */
    public static final int EXACTLY     = 1 << MODE_SHIFT;

    /**
     * Measure specification mode: The child can be as large as it wants up
     * to the specified size.
     */
    public static final int AT_MOST     = 2 << MODE_SHIFT;
}
```

### 冲突造成的现象

ListView只会显示一行item，高度只有一行的高度，可以在这个狭小的空间内滚动。

下面做一个实验
布局如下：由于ScrollView只能包含一个子View，所以我们用一个LinearLayout做子View，`android:orientation`设置成vertical，里面包含一个ListView和一个TextView。

```xml
<ScrollView
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/holo_purple">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <ListView
            android:id="@+id/listview"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="@android:color/holo_blue_bright"/>

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="bottom"
            android:background="@android:color/holo_green_light"
            android:text="hahaha"/>
    </LinearLayout>
</ScrollView>
```

Java代码就不放了，ListView很简单的加载20个数字，并显示出来。并且

* 给ScrollView设置了background为紫色
* 给ListView的item view设置了background为红色
* 给TextView设置了background为绿色

一下列出了3中情况，**注意观察右侧滚动条**：

#### 情况1
若 `ListView的高度 + TextView的高度 < 屏幕高度`，则ListView显示一个Item的高度，并且可以在此高度范围内滑动，ScrollView无法滑动。

运行效果如下图：
![](http://7xp4nq.com1.z0.glb.clouddn.com/image/blog/screenshot2.png?imageView/0/w/600/h/600)

#### 情况2
若 `ListView的高度 + TextView的高度 > 屏幕高度`，比如可以将TextView的`android:layout_height`设置成900dp。则ListView显示一个Item的高度，并且不能滑动，ScrollView可以滑动。
![](http://7xp4nq.com1.z0.glb.clouddn.com/image/blog/screenshot1.png?imageView/0/w/600/h/600)

#### 情况3
接情况2，把ListView换成我们自定义的ListView，ListView可完整显示20项，问题完美解决。
运行效果如下图：
![](http://7xp4nq.com1.z0.glb.clouddn.com/image/blog/screenshot3.png?imageView/0/w/600/h/600)

额外思考：这样做，效率如何？

