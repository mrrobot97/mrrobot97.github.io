---
layout: post
author: mrrobot97
title: ViewPager实现无限轮播
---

>Android自带的ViewPager有翻页功能，支持用户手势滑动，但是不支持循环滚动，因此我们就在PagerAdapter的基础上为其增加循环滚动的功能就行了。

自定义Adapter继承自PagerAdapter，我们需要重写四个方法:

	1.public int getCount()
	2.public Object instantiateItem(ViewGroup container, int position)
	3.public boolean isViewFromObject(View view, Object object)
	4.public void finishUpdate(ViewGroup container)
	
假如我们有N个item需要循环滚动，那么在getCount中就返回2*N，这样让ViewPager以为我们有2N个page需要滚动，因此在滚动到第N页，即逻辑上的最后一页时，还能继续向后滚动，因为此时ViewPager以为后面还有N个page呢~

在instantiateItem需要对position进行对N求模处理，毕竟我们实际上只有N个page，然后生成相应的View并添加到container中去：

```
@Override
    public Object instantiateItem(ViewGroup container, int position) {
        position%=viewCnt;
        ViewGroup parent= (ViewGroup) mViews.get(position).getParent();
        if (parent!=null){
            parent.removeView(mViews.get(position));
        }
        container.addView(mViews.get(position));
        return mViews.get(position);
    }
```

isViewFromObject的写法是固定的:

```
@Override
    public boolean isViewFromObject(View view, Object object) {
        return view==object;
    }
```

然后是finnishUpdate()方法，这个方法会在page切换完成之后调用，我们需要在这里实现page的偷偷替换：

```
@Override
    public void finishUpdate(ViewGroup container) {
        super.finishUpdate(container);
        int position=mViewPager.getCurrentItem();
        if (position==0){
            //first pager,false to cancel translation
            mViewPager.setCurrentItem(viewCnt,false);
        }else if(position==getCount()-1){
            //last pager
            mViewPager.setCurrentItem(viewCnt-1,false);
        }
    }
```
之所以将第1个替换为第N+1个是因为第一个page无法向左滑动，而第N+1个左右滑动都可以，因为它左边是第N个，右面是第N+2个。同理将第2N个替换为第N个，因为第2N个无法向右滑动。

因为替换的前后是同一个page的内容，并且我们的setCurrentItem()的第二个参数为false,即取消过渡动画，立即切换，因此用户不会感觉到page发生了切换。这样我们通过"暗度陈仓"，就实现了ViewPager的循环滚动。

理论上我们还应该重写destroyItem()方法，但在我重写并在其中调用container.removeView(mViews.get(position))之后，发现在滚动的过程中会出现白屏现象，也没找到原因，反而注释这段代码之后正常了，知道原因的话请留言告诉我。

Demo:

![](https://blog-1256554550.cos.ap-beijing.myqcloud.com/Screen%2520Capture%2520on%25202016-09-17%2520at%252015-51-54.gif)
