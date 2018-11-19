---
layout: post
author: mrrobot
title: 简单使用ViewDragHelper实现子View随意拖动
---
`ViewDragHelper`:故名思意，是用来帮助我们实现空间拖动的帮助类，其使用起来也是好简单的呢。

ViewDragHelper是用来作用于一个ViewGroup上，从而使得ViewGroup内部的子View可以被拖动的。

先上代码，这里我们拓展了一下LinearLayout：

```
public class MyLinearLayout extends LinearLayout {
    private ViewDragHelper mDragger;
    public MyLinearLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        mDragger=ViewDragHelper.create(this,1.0f,new ViewDragHelper.Callback(){

            @Override
            public boolean tryCaptureView(View child, int pointerId) {
                return true;
            }

            //限制水平移动距离,使其始终在父容器中
            @Override
            public int clampViewPositionHorizontal(View child, int left, int dx) {
                int leftBound=getPaddingLeft();
                int rightBound=getWidth()-child.getWidth()-leftBound;
                return Math.min(Math.max(left,leftBound),rightBound);
            }

            //限制竖直移动距离,使其始终在父容器中
            @Override
            public int clampViewPositionVertical(View child, int top, int dy) {
                int  topBound=getPaddingTop();
                int  bottomBound=getHeight()-child.getHeight()-topBound;
                return Math.min(Math.max(top,topBound),bottomBound);
            }
        });
    }

    @Override
    public boolean onInterceptHoverEvent(MotionEvent event) {
        return mDragger.shouldInterceptTouchEvent(event);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        mDragger.processTouchEvent(event);
        return true;
    }

    public MyLinearLayout(Context context) {
        this(context,null);
    }
}
```

我们使用ViewDragHelper.create(this,1.0f,new ViewDragHelper.Callback())方法来获得ViewDragHelper实例，其中第一个参数就是要与ViewDragHelper关联的ViewGroup，在这里就是我们的LinearLayout，第二个参数是拖拽时的灵敏度，一般传入1.0f即可。第三个参数回调Callback是用来处理拖动位置的，通常我们要重写它的两个方法：clampViewPositionHorizontal（）和clampViewPositionVertical，我们就是通过这两个方法来限制子View在父容器中被拖拽的水平位置和数值位置。注意这两个方法体中我们是如何实现限制子View式中在父容器中得。

接下来我们一定要重写Callback的tryCaptureView（）并返回true,表示允许尝试捕获子View

然后是重写onInterceptHoverEvent（）和onTouchEvent(),其写法基本固定为上面代码中的写法。

当完成这几个步骤后，我们的LinearLayout就基本实现了，当然我们只是实现了最基本的功能，即其内部的字View可以拖拽。

布局文件：

```
<?xml version="1.0" encoding="utf-8"?>
<com.mrrobot.useviewdraghelper.MyLinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    tools:context="com.mrrobot.useviewdraghelper.MainActivity">
    <TextView
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_gravity="center"
        android:gravity="center"
        android:background="@android:color/black"
        android:text="Hello World!"/>
    <TextView
        android:layout_width="50dp"
        android:layout_height="50dp"
        android:layout_gravity="center"
        android:gravity="center"
        android:background="@android:color/holo_blue_light"
        android:text="Hello World!"/>
</com.mrrobot.useviewdraghelper.MyLinearLayout>
```

最终效果：

![](http://cl.ly/2U1l3J3z1X2r/ezgif.com-video-to-gif.gif)
