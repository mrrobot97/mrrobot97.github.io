---
layout: post
author: mrrobot97
title: 动手写一个高仿微信的滑动返回SwipeBackActivity
---

>本文来源于[这个开源项目](https://github.com/XBeats/and_swipeback)，由于作者只给出了用法和基本原理，因此才有了本文的产生。本文去除了原项目中较琐碎和不必要的一些内容，只实现了基本的Activity滑动返回功能。

先看效果图：

![demo](http://ockr1qfi1.bkt.clouddn.com/swipebackdemo.gif)

效果还是挺不错的。

基本原理：利用Application类的registerActivityLifecycleCallbacks(ActivityLifecycleCallbacks)方法，可以记录全局所有Activity的生命周期，因此我们可以利用这点来存储我们所有的Activity于一个栈中，每次滑动返回时从栈中取出前一个Activtity,然后分离出其中id为`Window.ID_ANDROID_CONTENT`的FrameLayout，这个FrameLayout就是我们setContentView中的那个view的父view,利用这个FrameLayout就可以获取Activity界面显示的View。然后我们监听手势事件，在滑动的时候将前一个Activity的View加载进来并不断更改其偏移量即可。


首先是实现ActivityLifecycleCallbacks接口，并在其中用一个栈存储我们所有的Activity:

```
public class ActivityLifeCycleHelper implements Application.ActivityLifecycleCallbacks {
    private Stack<Activity> mActivities;

    public ActivityLifeCycleHelper(){
        mActivities=new Stack<>();
    }

    @Override
    public void onActivityCreated(Activity activity, Bundle bundle) {
        mActivities.add(activity);
    }

    @Override
    public void onActivityStarted(Activity activity) {

    }

    @Override
    public void onActivityResumed(Activity activity) {

    }

    @Override
    public void onActivityPaused(Activity activity) {

    }

    @Override
    public void onActivityStopped(Activity activity) {

    }

    @Override
    public void onActivitySaveInstanceState(Activity activity, Bundle bundle) {

    }

    @Override
    public void onActivityDestroyed(Activity activity) {
        mActivities.remove(activity);
    }

    public Activity getPreActivity(){
        int size=mActivities.size();
        if(size<2) return null;
        else return mActivities.get(size-2);
    }

    public void removeActivity(Activity activity){
        mActivities.remove(activity);
    }
}
```
看得出这个类很简单，只是在Activity创建的时候加入栈中，销毁的时候移除。
然后就是在Application中调用registerActivityLifecycleCallbacks()方法了:

```
public class MyApplication extends Application {
    public ActivityLifeCycleHelper getHelper() {
        return mHelper;
    }

    private ActivityLifeCycleHelper mHelper;
    @Override
    public void onCreate() {
        super.onCreate();
        mHelper=new ActivityLifeCycleHelper();
        //store all the activities
        registerActivityLifecycleCallbacks(mHelper);
    }
}
```



然后定义一个最基本的SwipeBackActivity,当然要继承自AppCompactActivity，这个类我们要做的就是重写它的dispatchTouchEvent()方法，这是因为我们要监听边界滑动返回事件，肯定要拦截其中的一些触摸事件。

```
public class SwipeBackActivity extends AppCompatActivity {
    private TouchHelepr mTouchHelepr;

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if(mTouchHelepr==null)
            mTouchHelepr=new TouchHelepr(getWindow());
        return super.dispatchTouchEvent(ev)||mTouchHelepr.processTouchEvent(ev);
    }
}

```
这里有一个我们自己写的类TouchHelper，具体的逻辑操作就在这里面实现了。接下来就是重点类TouchHelper了。
首先我们定义三个状态:

```
	private boolean isIdle=true;
    private boolean isSlinding=false;
    private boolean isAnimating=false;
```

 1. isIdle,表示当前为静止状态。
 2. isSliding,表示当前用户手指移动，我们的View随之滑动。
 3. isAnimating，表示用户手指松开，View要么恢复原状，要么移动至最右并消失，这是一个Animation过程，isAnimating=true表示当前处于这种动画过程中。
 
然后是几个成员变量:
 
```
	private Window mWindow;
    private ViewGroup preContentView;
    private ViewGroup curContentView;
    private ViewGroup curView;
    private ViewGroup preView;
    private Activity preActivity; 
    
    //左边触发的宽度
    private int triggerWidth=50;
	//阴影宽度
    private int SHADOW_WIDTH=30;
```

mWindow用于初始化TouchHelper，并且这个window就包含了context,activity等信息。

curContentView、preContentView分别表示当前、前一个Activity中外层的FrameLayout。

curView、preView分别表示当前、前一个Activity的界面View。

然后就是处理手势的代码了：

```
private Context getContext(){
        return mWindow.getContext();
    }

//决定是否拦截事件
    public boolean processTouchEvent(MotionEvent event){
        if(isAnimating) return true;
        float x=event.getRawX();
        switch (event.getAction()){
            case MotionEvent.ACTION_DOWN:
                if(x<=triggerWidth){
                    isIdle=false;
                    isSlinding=true;
                    startSlide();
                    return true;
                }
                break;
            case MotionEvent.ACTION_POINTER_DOWN:
                if(isSlinding) return true;
                break;
            case MotionEvent.ACTION_MOVE:
                if(isSlinding){
                    if(event.getActionIndex()!=0) return true;
                    sliding(x);
                }
                break;
            case MotionEvent.ACTION_CANCEL:
            case MotionEvent.ACTION_UP:
                if(!isSlinding) return false;
                int width=getContext().getResources().getDisplayMetrics().widthPixels;
                isAnimating=true;
                isSlinding=false;
                startAnimating(width/x<=3,x);
                return true;
            default:
                break;
        }
        return false;
    }
```
在函数processTouchEvent()中所有要拦截的地方我们都return true,这样子View就不会受到触摸事件了，其余的则应返回false,表示将触摸事件分发给子View去处理。

其中状态更改的代码比较简单，就不解释了。主要说说其中随着状态更改而进行的几个操作函数:

 1. startSlide()
 2. sliding(x)
 3. startAnimating(width/x<=3,x)
 
startSlide(),顾名思义，开始滑动，先看看代码：

```
private void startSlide() {
        preActivity=((MyApplication)getContext().getApplicationContext()).getHelper().getPreActivity();
        if(preActivity==null) return;
        preContentView=(ViewGroup) preActivity.getWindow().findViewById(Window.ID_ANDROID_CONTENT);
        preView= (ViewGroup) preContentView.getChildAt(0);
        preContentView.removeView(preView);
        curContentView=(ViewGroup) mWindow.findViewById(Window.ID_ANDROID_CONTENT);
        curView= (ViewGroup) curContentView.getChildAt(0);
        preView.setX(-preView.getWidth()/3);
        curContentView.addView(preView,0);
        //        if(mShadowView==null){
//            mShadowView=new ShadowView(getContext());
//        }
//        FrameLayout.LayoutParams params=new FrameLayout.LayoutParams(SHADOW_WIDTH, FrameLayout.LayoutParams.MATCH_PARENT);
//        curContentView.addView(mShadowView,1,params);
//        mShadowView.setX(-SHADOW_WIDTH);
    }
```

在startSlide()中，我们给几个成员变量赋值，并且将preView添加到curContentView中，并赋予其一个初始偏移量。这里要特别注意addView(view,index)中的index参数，index参数越大，代表越靠后绘制。这里添加preView时的index为0，表示最先绘制preView，否则preView会显示在curView的上面，这样就不正确了。注释部分稍后再讲。

然后看看sliding()方法:

```
private void sliding(float rawX) {
        if(preActivity==null) return;
        curView.setX(rawX);
        preView.setX(-preView.getWidth()/3+rawX/3);
        //mShadowView.setX(-SHADOW_WIDTH+rawX);
    }
```
这个函数就简单多了，这是随着用户手指的位置动态地更改curView、preView而已。注释稍后讲。

然后是startAnimating()方法:

```
private void startAnimating(final boolean isFinishing, float x) {
        int width=getContext().getResources().getDisplayMetrics().widthPixels;
        ValueAnimator animator=ValueAnimator.ofFloat(x,isFinishing?width:0);
        animator.setInterpolator(new DecelerateInterpolator());
        animator.setDuration(200);
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator valueAnimator) {
                sliding((Float) valueAnimator.getAnimatedValue());
            }
        });
        animator.addListener(new Animator.AnimatorListener() {
            @Override
            public void onAnimationStart(Animator animator) {

            }

            @Override
            public void onAnimationEnd(Animator animator) {
                doEndWorks(isFinishing);
            }

            @Override
            public void onAnimationCancel(Animator animator) {

            }

            @Override
            public void onAnimationRepeat(Animator animator) {

            }
        });
        animator.start();
    }
```

当用户松开手指时的位置的x坐标小于屏幕宽度的1/3时，恢复原状，否则将preView完全显示，这里利用ValueAnimator来实现动画。注意在动画完成后我们还要做一些收尾工作，就是方法doEndWorks():

```
private void doEndWorks(boolean isFinishing) {
        if(preActivity==null) return;
        if(isFinishing){
            //更改当前activity的底view为preView,防止当前activity finish时的白屏闪烁
            BackView view=new BackView(getContext());
            view.cacheView(preView);
            curContentView.addView(view,0);
        }
        //curContentView.removeView(mShadowView);
        if(curContentView==null||preContentView==null) return;
        curContentView.removeView(preView);
        preContentView.addView(preView);
        if(isFinishing){
            ((Activity)getContext()).finish();
            ((Activity)getContext()).overridePendingTransition(0,0);
        }
        isAnimating=false;
        isSlinding=false;
        isIdle=true;
        preView=null;
        curView=null;
    }
```

收尾工作中我们将状态修正，该移除的View移除，该添加的View添加。若preView完全显示，就finish当前activity,注意还要利用`((Activity)getContext()).overridePendingTransition(0,0)`取消默认的activity更换动画，这样才能实现暗度陈仓的目的。你应该已经看到了这里还有一个BackView，这个BackView其实就是preView的一个副本，我们将BackView添加到curContentView的最底部，覆盖那个白色底部，否则动画完成后会有一个白屏闪烁现象。

```
//用于防止白屏闪烁
class BackView extends View{

    private View mView;
    public BackView(Context context) {
        super(context);
    }

    public void cacheView(View view){
        mView=view;
        invalidate();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if(mView!=null){
            mView.draw(canvas);
            mView=null;
        }
    }
}
```

这样实现的效果就是如下:

![demo](http://ockr1qfi1.bkt.clouddn.com/withouwshadowview.gif)

你应该注意要这个和第一个demo显示的不太一样，因为这里没有阴影效果，体现不出层次感，不够美观，那么接下来我们只需要在添加一点点代码就可以添加这样的一个阴影效果.

```
class ShadowView extends View{

    private Drawable mDrawable;

    public ShadowView(Context context) {
        super(context);
        int[] colors=new int[]{0x00000000, 0x17000000, 0x43000000};
        mDrawable=new GradientDrawable(GradientDrawable.Orientation.LEFT_RIGHT,colors);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mDrawable.setBounds(0,0,getMeasuredWidth(),getMeasuredHeight());
        mDrawable.draw(canvas);
    }
}

```
这个就是要绘制上去的阴影效果，很简单，将前面几个方法中注释的代码部分还原即可。

这样，就完成了我们所有的代码~~~

喜欢的话，去给[原项目](https://github.com/XBeats/and_swipeback)start吧，当然如果你能给我的文章一个喜欢话我会更开心的~