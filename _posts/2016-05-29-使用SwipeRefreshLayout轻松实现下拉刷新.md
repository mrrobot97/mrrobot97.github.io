---
layout: post
author: mrrobot
title: 使用SwipeRefreshLayout轻松实现下拉刷新
---
>是在看别的的源代码时才了解到的这个官方控件，都出了这么久了我居然不知道，真是醉了。再也不用第三方的了。
使用方法：

1. 在布局文件中添加SwipeRefreshLayout.
2. 将ListView或者GridView或者RecyclerView放到SwipeRefreshLayout布局中。
3. 在代码中为SwipeRefreshLayout设置onRefresh监听事件：setOnRefreshListener.
4. 重写onRefresh()方法，添加自己的逻辑代码,注意在数据加载完成后要调用SwipeRefreshLayout的setRefreshing(false)，否则UI将会一直处于刷新状态。

SwipeRefreshLayout还可以设置颜色。

eg:

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="me.mrrobot97.swiperefreshlayout.MainActivity">

    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/swipe_refresh"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ListView
            android:id="@+id/list_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"></ListView>
    </android.support.v4.widget.SwipeRefreshLayout>
</RelativeLayout>

```

```
public class MainActivity extends AppCompatActivity {
    private ListView mListView;
    private SwipeRefreshLayout mSwipeRefresh;
    private ArrayAdapter<String> mAdapter;
    private List<String> data=new ArrayList<>();
    private int count=10;
    private Handler mHandler=new Handler(){
        @Override
        public void handleMessage(Message msg) {
            mAdapter.notifyDataSetChanged();
            mSwipeRefresh.setRefreshing(false);
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initData();
        init();

    }

    private void initData() {
        for (int i=0;i<10;i++){
            data.add("Item "+i);
        }
    }

    private void init() {
        mListView= (ListView) findViewById(R.id.list_view);
        mSwipeRefresh= (SwipeRefreshLayout) findViewById(R.id.swipe_refresh);
        mSwipeRefresh.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
            @Override
            public void onRefresh() {
                data.add("Item "+(count++));

                //成功加载数据后关闭刷新
                mHandler.sendEmptyMessageDelayed(0,1000);

            }
        });
        mAdapter=new ArrayAdapter<String>(this,android.R.layout.simple_list_item_1,data);
        mListView.setAdapter(mAdapter);
    }
}

```

最终效果图：

![](http://i2.buimg.com/2aaaf1a4c1f3d8cb.gif)


