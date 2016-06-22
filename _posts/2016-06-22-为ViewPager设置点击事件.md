layout: post
title: 为ViewPager设置点击事件
author: mrrobot
---

由于ViewPager要监听用户的手势操作，故直接为ViewPager设置OnClickListener是不起作用的，因为点击事件被ViewPager内部消耗了。因此想要监听ViewPager的点击事件，我们应该利用ViewPager的setOnTouchListener()方法，在该方法中检测点击事件，然后做出响应。具体代码如下：

```
mViewPager.setOnTouchListener(new View.OnTouchListener() {
            float oldX = 0, newX = 0, sens = 5;

            @Override
            public boolean onTouch(View v, MotionEvent event) {
                switch (event.getAction()) {
                    case MotionEvent.ACTION_DOWN:
                        oldX = event.getX();
                        break;

                    case MotionEvent.ACTION_UP:
                        newX = event.getX();
                        if (Math.abs(oldX - newX) < sens) {
							 //do something here
                            return true;
                        }
                        oldX = 0;
                        newX = 0;
                        break;
                }
                return false;
            }
        });
```