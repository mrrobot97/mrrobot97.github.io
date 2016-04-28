---
layout: post
author: mrrobot
title: Android中利用Semaphore进行线程同步
---
> 在Android中，经常出现多个线程调度公共资源的情况，当线程个数大于公共资源的数量时，就必须要对线程的调度进行同步，否则很可能会报错，本文通过简单的从用法上介绍Semaphore来说明如何进行线程之间的同步。

Semaphore的作用其实就是一个计数器，当线程调度资源时，通过调用Semaphore.acquire()方法先从Semaphore处获得一个permit才可以继续执行，否则就会一直wait直到获取到permit或者超时。当Semaphore为0时，任何线程的请求都将wait.而当线程使用完公共资源后，执行Semaphore.release()方法来释放资源，其实质就是对Semaphore进行加一。 

Semaphore mSemaphore=new Semaphore(0);参数为初始的permit的数量，即一次性最多可以获得permit的线程数量。当设置为0时，只有在某个线程中先执行一次Semaphore.release()方法对Semaphore增1，别的线程才可以通过Semaphore.acquire()方法来获取permit。

比如我们有一个公共资源 整形数num，我们希望主线程能够先调用num,然后对num加一，然后子线程才能够调用num的指。我们可以这么做：

	private int num=0;
    private Semaphore mSemaphore=new Semaphore(0);
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //请求permit,若无则一直等待直到permit数量大于0或者超时
                    mSemaphore.acquire();
                    //此处打印出的num应该为1
                    Log.d("YJW","current thread is "+Thread.currentThread().getId()+" and num is "+num);
                    //利用完公共资源后释放permit,相当于Semaphore对执行加1操作
                    mSemaphore.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
        //主线程与子线程异步执行,这里我们确保主线程中的Log先进行执行,然后num+1,然后执行子线程的Log操作
        Log.d("YJW","current thread is "+Thread.currentThread().getId()+" and num is "+num++);
        //只有这里release()之后,Semaphore的permit数量为1，然后子线程的acquire()才会执行成功,然后执行之后的内容,这样就达到了同步线程的目的
        mSemaphore.release();
    }
    
输出结果：

	04-27 21:33:43.208 1663-1663/com.mrrobot.semaphorecase D/YJW: current thread is 1 and num is 0
	04-27 21:33:43.208 1663-1784/com.mrrobot.semaphorecase D/YJW: current thread is 138 and num is 1
	
我们可以看出，的确是thread id 为1的主线程先打印Log,并对num加1，然后子线程才能打印Log。这样就达到了我们线程之间资源同步的目的。

同样的多个permit的Semaphore使用方法与此处一个道理，故不再赘述。