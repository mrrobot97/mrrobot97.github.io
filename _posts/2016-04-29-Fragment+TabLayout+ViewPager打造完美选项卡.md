---
layout: post
author: mrrobot
title: Fragment+TabLayout+ViewPager打造完美选项卡
---
>2015年Google IO大会上谷歌发布了Android Design Support Library，包含了许多新特性，其中整合了很多之前的开源第三方空间，其中TabLayout使得选项卡的实现变得非常简单。

####TabLayout: compile 'com.android.support:design:23.1.1'
####ViewPager: compile 'com.android.support:support-v4:23.1.1'



首先自定义我们自己的Fragment，其布局很简单，每个Fragment中仅仅显示一个TextView以区别是第几个Fragment：

	public class MyFragment extends Fragment {
    private String mTitle;
    private static final String KEY="title";

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        Bundle bundle=getArguments();
        mTitle=bundle.getString(KEY);
        TextView tv=new TextView(getActivity());
        tv.setText(mTitle);
        tv.setGravity(Gravity.CENTER);
        return tv;
    }

    public static MyFragment newInstance(String title){
        MyFragment mf=new MyFragment();
        Bundle bundle=new Bundle();
        bundle.putString(KEY,title);
        mf.setArguments(bundle);
        return mf;
    }
	}

然后是编写我们的布局文件，也很简单，仅仅包含一个TabLayout和一个ViewPager：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="match_parent">

   <android.support.design.widget.TabLayout
       android:id="@+id/tablayout"
       android:layout_width="match_parent"
       android:layout_height="wrap_content"
       app:tabSelectedTextColor="#FF4081"
       app:tabTextColor="#000"
       app:tabMinWidth="100dp">

   </android.support.design.widget.TabLayout>

   <android.support.v4.view.ViewPager
       android:id="@+id/view_pager"
       android:layout_width="match_parent"
       android:layout_weight="1"
       android:layout_height="0dp">


   </android.support.v4.view.ViewPager>
</LinearLayout>
```

接下来编写MainActivity:

```
package com.mrrobot.viewpagertest;

import android.os.Bundle;
import android.support.design.widget.TabLayout;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentActivity;
import android.support.v4.app.FragmentPagerAdapter;
import android.support.v4.view.ViewPager;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends FragmentActivity {
    private ViewPager mViewPager;
    private List<MyFragment> mFragments=new ArrayList<>();
    private FragmentPagerAdapter mAdapter;
    private TabLayout mTabLayout;
    private List<String> mTitles=new ArrayList<>();
    //控制选项卡的数量
    private static int VIEW_COUNT=6;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initViews();
        initDatas();
    }

    private void initDatas() {
        for (int i=0;i<VIEW_COUNT;i++){
            mTitles.add("text"+i);
            mFragments.add(MyFragment.newInstance(mTitles.get(i)));
        }
        mAdapter=new FragmentPagerAdapter(getSupportFragmentManager()) {
            @Override
            public Fragment getItem(int position) {
                return mFragments.get(position);
            }

            @Override
            public int getCount() {
                return mFragments.size();
            }
            //必须手动重写getPageTiele(),不然tab选项卡无法获取title
            @Override
            public CharSequence getPageTitle(int position) {
                return mTitles.get(position);
            }
        };
        mViewPager.setAdapter(mAdapter);
        for (int i=0;i<VIEW_COUNT;i++){
            mTabLayout.addTab(mTabLayout.newTab());
        }
        //设置Tab可以滚动,适合在选项卡比较多,一个屏幕无法显示完全的情况下使用
        mTabLayout.setTabMode(TabLayout.MODE_SCROLLABLE);
        //将ViewPager与TabLayout联系起来,即实现联动
        mTabLayout.setupWithViewPager(mViewPager);
        //mViewPager.setOnPageChangeListener(new TabLayout.TabLayoutOnPageChangeListener(mTabLayout));
    }

    private void initViews() {
        mViewPager= (ViewPager) findViewById(R.id.view_pager);
        mTabLayout= (TabLayout) findViewById(R.id.tablayout);
    }
}
```

这里我们一共设置了六个选项卡，当然不能在一个屏幕上全部显示出来，所以我们将TabLayout的TabMode设置为TabLayout.MODE_SCROLLABLE，并且将TabLayout与ViewPager通过mTabLayout.setupWithViewPager(mViewPager)联系起来，这样我们的TabLayout就能够跟随我们的ViewPager联动了。并且当点击TabLayout的某一个Tab时，ViewPager的Page可以自动切换。

注意我们在layout文件中为TabLayout设置了一个属性：

	app:tabMinWidth="100dp"

这是为了防止当选项卡的数量过多时，选项卡全部挤在一个屏幕上，导致每个选项卡的宽度都非常小，十分不美观。

当然当你的选项卡数量较少时，可以直接设置TabLayout的TabMode为TabLayout.MODE_FIXED.

注意在FragmentPagerAdapter中我们要手动重写函数getPageTitle，不然TabLayout的各个Tab是无法获取Title，即TabLayout上没有任何文字。

最终效果图 ：

![](http://cl.ly/3E3n2c1m0I1U/ezgif.com-video-to-gif.gif)

	