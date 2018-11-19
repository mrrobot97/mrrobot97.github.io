---
layout: post
title: 使用ItemTouchHelper实现RecyclerView的拖动排序、滑动删除
author: mrrobot
---

>Google为我们提供的RecyclerView真的是非常的好用，高度可定制化，这次我们利用ItemTouchHelper来实现对RecyclerView的拖动排序以及滑动删除。

先看看效果图:

![效果图](https://blog-1256554550.cos.ap-beijing.myqcloud.com/demo.gif)

效果还是不错的，如果再结合自定义的ItemAnimator,可以实现很漂亮的花样。

首先我们要定义一个类实现``ItemTouchHelper.Callback``接口，然后实现其中的几个方法:

	1.public boolean isLongPressDragEnabled()
	2.public boolean isItemViewSwipeEnabled()
	3.public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder)
	4.public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target)
	5.public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction)
	
	
其中isLongPressDragEnable是决定是否可以通过长按Item来对其排序，isItemViewSwipeEnable决定是否可以滑动Item，只需要在需要实现的功能里面返回true即可，这里我们两个都返回true

```
@Override
    public boolean isLongPressDragEnabled() {
        return true;
    }

    @Override
    public boolean isItemViewSwipeEnabled() {
        return true;
    }
```

然后就是重写onMove和onSwiped方法了，在这两个方法里面我们可以获取需要改变的item的位置，然后我们需要将这些位置通知给我们的Adapter，做法是定义一个接口：

```
public interface OnDragListener {

    void onDrag(int from,int to);

    void onDismiss(int pos);
}
```

让MainActivity实现这个接口，然后重写这两个方法，并通知Adapter数据更新:

```
@Override
    public void onDrag(int from, int to) {
        Collections.swap(mList,from,to);
        mAdapter.notifyItemMoved(from,to);
    }

    @Override
    public void onDismiss(int pos) {
        mList.remove(pos);
        mAdapter.notifyItemRemoved(pos);
    }
```

onMove以及onSwiped方法如下:

```
@Override
    public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        int dragFlags=ItemTouchHelper.DOWN|ItemTouchHelper.UP|ItemTouchHelper.LEFT|ItemTouchHelper.RIGHT;
        int swipeFlags=ItemTouchHelper.LEFT|ItemTouchHelper.RIGHT;

        return makeMovementFlags(dragFlags,swipeFlags);
    }

    @Override
    public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
        mListener.onDrag(viewHolder.getAdapterPosition(),target.getAdapterPosition());
        return true;
    }
```

makeMovementFlags()是用来控制滑动以及拖动的方向的，用法如上所示。

最后就是在MainActivity中将我们的ItemTouchHelper.Callback与RecyclerView绑定:

```
ItemTouchHelper.Callback callback=new MyItemTouchHelper(this);
        ItemTouchHelper helper=new ItemTouchHelper(callback);
        helper.attachToRecyclerView(mRecyclerView);
```

这样所有的工作就做完了。
完整的项目源码地址[在此](https://github.com/mrrobot97/DragableRecyclerView)
