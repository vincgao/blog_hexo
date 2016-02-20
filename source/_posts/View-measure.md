---
title: Android Layout绘制
---


   > 最近对View进行了一番研究，主要参考了一下几篇blog，多数为汇总而来，之后不定期删改文章，加深理解，算是我对Android View的一些笔记吧

参考链接：

* http://www.codekk.com/blogs/detail/54cfab086c4761e5001b253f
* http://blog.csdn.net/guolin_blog/article/details/16330267
* http://blog.csdn.net/singwhatiwanna/article/details/38426471

# View 树的绘图流程
当 Activity 接收到焦点的时候，它会被请求绘制布局，该请求由 Android framework 处理。绘制是从根节点开始，对布局树进行 measure 和 draw。整个 View 树的绘图流程在ViewRoot.java类的`performTraversals()`函数展开，该函数所做的工作可简单概况为是否需要重新计算视图大小(measure)、是否需要重新安置视图的位置(layout)、以及是否需要重绘(draw)，流程图如下：
![](http://7xp4nq.com1.z0.glb.clouddn.com/view_mechanism_flow.png)

每一个视图的绘制过程都必须经历三个最主要的阶段：

* onMeasure()
* onLayout()
* onDraw()

<img src="http://7xp4nq.com1.z0.glb.clouddn.com/android.png" height="600" align=center></img>

# 我的理解
* measure最重要的成果是在 view 的 `onMeasure()` 里通过`setMeasuredDimension()`将mMeasuredWidth和mMeasuredHeight赋值。此过程后可调用 `getMeasureWidth()` `getMeasureHeight()`。
* layout过程，得到mLeft、mTop、mBottom、mRight。另外，所有的ViewGroup类都应重写onLayout。
* draw对视图进行绘制，拢共分6个步骤。
* mMeasuredWidth、mMeasuredHeight、mLeft、mTop、mBottom、mRight均是View类的成员变量。

# 一、onMeasure()

从整体上看Measure和Layout两个步骤的执行：
![](http://7xp4nq.com1.z0.glb.clouddn.com/measure_layout.png)

树的遍历是有序的，由父视图到子视图，每一个ViewGroup负责测绘它所有的子视图，而最底层的View会负责测绘自身。

### measure过程做了什么
measure过程由`measure(int, int)`方法发起，从上到下有序测量View，在 measure 过程的最后，每个视图存储了自己的尺寸大小和测量规格。

measure 过程会为一个 View 及所有子节点的 mMeasuredWidth 和 mMeasuredHeight 变量赋值，该值可以通过 `getMeasuredWidth()`和`getMeasuredHeight()`方法获得。而且这两个值必须在父视图约束范围之内，这样才可以保证所有的父视图都接收所有子视图的测量。如果子视图对于 Measure 得到的大小不满意的时候，父视图会介入并设置测量规则进行第二次 measure。比如，父视图可以先根据未给定的 dimension 去测量每一个子视图，如果最终子视图的未约束尺寸太大或者太小的时候，父视图就会使用一个确切的大小再次对子视图进行 measure。

### measure过程传递尺寸的两个类

* ViewGroup.LayoutParams （View 自身的布局参数）
* MeasureSpecs 类（**父视图对子视图的测量要求**）

**ViewGroup.LayoutParams**
这个类我们很常见，就是用来指定视图的高度和宽度等参数。对于每个视图的 height 和 width，有以下选择：

* 具体值
* MATCH_PARENT 表示子视图希望和父视图一样大(不包含 padding 值)
* WRAP_CONTENT 表示视图为正好能包裹其内容大小(包含 padding 值)

ViewGroup 的子类有其对应的 ViewGroup.LayoutParams 的子类。比如 RelativeLayout 拥有的 ViewGroup.LayoutParams 的子类 RelativeLayoutParams。
有时我们需要使用 `view.getLayoutParams()` 方法获取一个视图 LayoutParams，然后进行强转，但由于不知道其具体类型，可能会导致强转错误。其实该方法得到的就是其所在父视图类型的 LayoutParams，比如 View 的父控件为 RelativeLayout，那么得到的 LayoutParams 类型就为 RelativeLayoutParams。

**MeasureSpecs**
测量规格，包含测量要求和尺寸的信息，有三种模式:

* UNSPECIFIED
父视图不对子视图有任何约束，它可以达到所期望的任意尺寸。比如 ListView、ScrollView，一般自定义 View 中用不到。

* EXACTLY
父视图为子视图指定一个确切的尺寸，而且无论子视图期望多大，它都必须在该指定大小的边界内，对应的属性为 **match_parent 或具体值**，比如 100dp，父控件可以通过MeasureSpec.getSize(measureSpec)直接得到子控件的尺寸。

* AT\_MOST
父视图为子视图指定一个最大尺寸。子视图必须确保它自己所有子视图可以适应在该尺寸范围内，对应的属性为 **wrap_content**，这种模式下，父控件无法确定子 View 的尺寸，只能由子控件自己根据需求去计算自己的尺寸，这种模式就是我们自定义视图需要实现测量逻辑的情况。

MeasureSpec 代表一个 32 位 int 值，高 2 位代表 SpecMode，低 30 位代表 SpecSize。
MeasureSpec 内部的一些常量的定义：

```java
private static final int MODE_SHIFT = 30;
private static final int MODE_MASK = 0x3 << MODE_SHIFT;
public static final int UNSPECIFIED = 0 << MODE_SHIFT;
public static final int EXACTLY = 1 << MODE_SHIFT;
public static final int AT_MOST = 2 << MODE_SHIFT;

public static int makeMeasureSpec(int size, int mode) {
    if (sUseBrokenMakeMeasureSpec) {
        return size + mode;
    } else {
        return (size & ~MODE_MASK) | (mode & MODE_MASK);
    }
}

public static int getMode(int measureSpec) {
    return (measureSpec & MODE_MASK);
}

public static int getSize(int measureSpec) {
    return (measureSpec & ~MODE_MASK);
}
```


### measure 核心方法

1. `measure(int widthMeasureSpec, int heightMeasureSpec)`
该方法定义在View.java类中，为 final 类型，不可被复写，但 measure 调用链最终会回调 View/ViewGroup 对象的 `onMeasure()`方法，因此自定义视图时，只需要复写 `onMeasure()` 方法即可。

2. `onMeasure(int widthMeasureSpec, int heightMeasureSpec)`
该方法就是我们自定义视图中实现测量逻辑的方法，该方法的参数是父视图对子视图的 width 和 height 的测量要求。在我们自身的自定义视图中，要做的就是根据该 widthMeasureSpec 和 heightMeasureSpec 计算视图的 width 和 height，不同的模式处理方式不同。

3. `setMeasuredDimension()`
测量阶段终极方法，在 `onMeasure(int widthMeasureSpec, int heightMeasureSpec)` 方法中调用，将计算得到的尺寸，传递给该方法，测量阶段即结束。**该方法也是必须要调用的方法，否则会报异常。**在我们在自定义视图的时候，不需要关心系统复杂的 Measure 过程，只需调用`setMeasuredDimension()`设置根据 MeasureSpec 计算得到的尺寸即可，你可以参考 ViewPagerIndicator 的 onMeasure 方法。

ViewGroup 的 `measureChildren（int widthMeasureSpec, int heightMeasureSpec)` 方法对复合 View 的 Measure 流程图：
![](http://7xp4nq.com1.z0.glb.clouddn.com/measurechildflow.png)

### getWidth()和getMeasureWidth()的区别

* `getMeasureWidth()`方法在`measure()`过程结束后可以获取到，`getWidth()`方法要在`layout()`过程结束后才能获取到
* `getMeasureWidth()`方法中的值是通过`setMeasuredDimension()`方法来进行设置的，而`getWidth()`方法中的值则是通过视图右边的坐标减去左边的坐标计算出来的。

```java
public final int getMeasuredWidth() {
   return mMeasuredWidth & MEASURED_SIZE_MASK;
}
```

```java
public final int getWidth() {
   return mRight - mLeft;
}
```

# 二、onLayout()
### layout过程做了什么
首先要明确的是，子视图的具体位置都是相对于父视图而言的。在 layout 过程中，子视图会调用`getMeasuredWidth()`和`getMeasuredHeight()`方法获取到 measure 过程得到的 mMeasuredWidth 和 mMeasuredHeight，作为自己的 width 和 height。ViewRoot的`performTraversals()`方法会在measure结束后继续执行，并调用View的`layout()`方法来执行此过程，如下所示：

```java
host.layout(0, 0, host.getMeasuredWidth(), host.getMeasuredHeight());
```
然后调用每一个子视图的`layout(l, t, r, b)`函数，来确定每个子视图在父视图中的位置(mLeft、mTop、mBottom、mRight)。

View中的`onLayout()`方法是一个空方法，因为`onLayout()`过程是为了确定视图在布局中所在的位置，而这个操作应该是由布局来完成的，即父视图决定子视图的显示位置。
ViewGroup中的`onLayout()`方法是一个抽象方法。**所有ViewGroup的子类都必须重写这个方法，LinearLayout、RelativeLayout都需要进行重写，然后在内部按照各自的规则对子视图进行布局。**

在ViewGroup中
无`measure()` `onMeasure()` 、`draw()` `onDraw()`
有`layout()`
`onLayout()`方法为abstract方法

```java
@Override
protected abstract void onLayout(boolean changed, int l, int t, int r, int b);
```
View中有`measure()` `layout()`  `draw()`

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
   setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
           getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

```java
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
}
```

```java
protected void onDraw(Canvas canvas) {
}
```

# 三、onDraw()
### draw过程做了什么
measure和layout的过程都结束后，ViewRoot中的代码会继续执行并创建出一个Canvas对象，然后调用View的`draw()`方法来执行具体的绘制工作。`draw()`方法内部的绘制过程总共可以分为六步，其中第二步和第五步在一般情况下很少用到。

### 与 draw 过程相关的函数

* `View.draw(Canvas canvas)` 由于 ViewGroup 并没有复写此方法，因此，**所有的视图最终都是调用 View 的 draw 方法进行绘制的**。在自定义的视图中，也不应该复写该方法，而是复写` onDraw(Canvas) `方法进行绘制，如果自定义的视图确实要复写该方法，那么请先调用 `super.draw(canvas)`完成系统的绘制，然后再进行自定义的绘制。

* `View.onDraw()` View 的`onDraw（Canvas）`默认是空实现，自定义绘制过程需要复写的方法，绘制自身的内容。

* `dispatchDraw()` 发起对子视图的绘制。View 中默认是空实现，ViewGroup 复写了`dispatchDraw()`来对其子视图进行绘制。该方法我们不用去管，自定义的 ViewGroup 不应该对`dispatchDraw()`进行复写。


```java
/*
* Draw traversal performs several drawing steps which must be executed
* in the appropriate order:
*
*      1. Draw the background
*      2. If necessary, save the canvas' layers to prepare for fading
*      3. Draw view's content
*      4. Draw children
*      5. If necessary, draw the fading edges and restore layers
*      6. Draw decorations (scrollbars for instance)
*/
```
绘制流程图：
![](http://7xp4nq.com1.z0.glb.clouddn.com/draw_method_flow.png)

不管是Button也好，TextView也好，任何一个视图都是有滚动条的，只是一般情况下我们都没有让它显示出来而已。

View是不会帮我们绘制内容部分的，因此需要每个视图根据想要展示的内容来自行绘制。TextView、ImageView等类都有重写`onDraw()`，如果需要复写该方法，请记得先调用父类的方法。(类比所有ViewGroup的子类都必须重写`onLayout()`)，并且在里面执行了相当不少的绘制逻辑。绘制的方式主要是借助Canvas这个类，它会作为参数传入到`onDraw()`方法中，供给每个视图使用。Canvas这个类的用法非常丰富，基本可以把它当成一块画布，在上面绘制任意的东西。





