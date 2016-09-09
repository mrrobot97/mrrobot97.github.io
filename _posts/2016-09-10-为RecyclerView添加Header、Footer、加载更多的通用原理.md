---
layout: post
author: mrrobot
title: 为RecyclerView添加Header、Footer、加载更多的通用原理
---

>RecyclerView固然好用，但是官方并没有为其提供像ListView的addHeaderView()、addFooterView()等方法，固我们需要自己来实现。本文以一个简单例子介绍其通用原理，对题干中的三项都试用。

方法就是：继承RecyclerView.Adapter，然后在其内部进行自己的逻辑实现。这里以在RecyclerView的尾部添加一个加载更多的FooterView为例进行介绍。

继承RecyclerView.Adapter，则需要重写三个方法：

1.onCreateViewHolder(ViewGroup parent, int viewType)

2.onBindViewHolder(RecyclerView.ViewHolder holder, int position)

3.getItemCount()

注意到在onCreateViewHolder方法中有一个参数viewType,我们就是利用这个参数进行改造。首先声明两个ViewType:

	private static final int VIEW_TYPE_NORMAL=0;

    private static final int VIEW_TYPE_BUTTON=1;
    
    
    
    
既然有两种viewType，自然就需要有两个布局文件，一个对应我们普通item的layout,一个对应我们的footerview的layout.同理有两个ViewHolder

```
public static class ViewHolder extends RecyclerView.ViewHolder{
        public TextView mTextView;

        public ViewHolder(View itemView) {
            super(itemView);
            mTextView= (TextView) itemView.findViewById(R.id.text_view);
        }
    }

    public  class LoadMoreHolder extends RecyclerView.ViewHolder{
        private Button mButton;
        private RelativeLayout mLayout;
        public LoadMoreHolder(View itemView) {
            super(itemView);
            mLayout= (RelativeLayout) itemView.findViewById(R.id.relative_layout);
            mButton= (Button) itemView.findViewById(R.id.load_more_bt);
            mButton.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View view) {
                    if (mListener!=null){
                        mListener.loadMore();
                    }
                }
            });
        }
    }
```

这里我们必须主动重写一个函数getItemViewType(int position)，我们需要根据position来决定我们的viewType，这里我添加的是footerview，所以实现逻辑是这样的：

```
@Override
    public int getItemViewType(int position) {
        if (position<mData.size()){
            return VIEW_TYPE_NORMAL;
        }
        return VIEW_TYPE_BUTTON;
    }
```
然后是onCreateViewHolder，在这里根据viewType来生成不同的view以及ViewHolder

```
@Override
    public RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        if (viewType==VIEW_TYPE_NORMAL){
            View view= LayoutInflater.from(mContext).inflate(R.layout.item_layout,null);
            ViewHolder holder=new ViewHolder(view);
            return holder;
        }else{
            View view=LayoutInflater.from(mContext).inflate(R.layout.load_more_button_layout,null);
            LoadMoreHolder holder=new LoadMoreHolder(view);
            return holder;
        }

    }
```

然后是onBindViewHolder

```
@Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
        if (position<mData.size()){
            ((ViewHolder)holder).mTextView.setText(mData.get(position));
        }
    }
```
在getItemCount函数中注意要+1，因为我们手动为RecyclerView添加了一个footerview

到这里为止，我们的Adapter已经能正常为LinearLayoutManager样式的RecyclerView工作了，
若还想要在GridLayoutManager和StaggeredLayoutManager时正常使用，还需重写下面两个函数：

```
@Override
    public void onAttachedToRecyclerView(RecyclerView recyclerView) {
        super.onAttachedToRecyclerView(recyclerView);
        RecyclerView.LayoutManager manager=recyclerView.getLayoutManager();
        if (manager instanceof GridLayoutManager){
            final RecyclerView.LayoutManager finalManager = manager;
            ((GridLayoutManager) manager).setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
                @Override
                public int getSpanSize(int position) {
                    return position<mData.size()?((GridLayoutManager) finalManager).getSpanCount():1;
                }
            });
        }
    }

    @Override
    public void onViewAttachedToWindow(RecyclerView.ViewHolder holder) {
        super.onViewAttachedToWindow(holder);
        if (holder instanceof LoadMoreHolder){
            ViewGroup.LayoutParams lp=((LoadMoreHolder) holder).mLayout.getLayoutParams();
            if (lp!=null&&lp instanceof StaggeredGridLayoutManager.LayoutParams){
                ((StaggeredGridLayoutManager.LayoutParams)lp).setFullSpan(true);
            }
        }
    }
```

应该能看懂，不解释了。

样例全部代码:
[Github样例代码](https://github.com/mrrobot97/LoadMoreRecyclerView)
