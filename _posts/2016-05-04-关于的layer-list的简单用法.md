---
layout: post
author: mrrobot
title: 关于的layer-list的简单用法
---

今天看别人博客时第一次见到了layer-list的用法。layer-list的用途是将多张png图片按照指定的顺序一层层地叠放在一起，这是其最常见的用法。

比如我们有七张png图片，分别为：

![](http://cl.ly/2a0H130f0o1j/eel_mask1.png)
![](http://cl.ly/3X3Z021P0P3f/eel_mask2.png)
![](http://cl.ly/3H3a0S1U0R0X/eel_mask3.png)
![](http://cl.ly/2F2Z0w3Y2W21/eel_mask4.png)
![](http://cl.ly/0B1G221T0933/eel_mask5.png)
![](http://cl.ly/3l2l461K1y0w/eel_mask6.png)
![](http://cl.ly/1T0J2R0K3T3F/eel_mask7.png)

现在我们在项目想要将他们合成一张图片，

在drawable目录下新建一个eel.xml文件：

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:drawable="@drawable/eel_mask1"/>
    <item
        android:drawable="@drawable/eel_mask2"/>
    <item
        android:drawable="@drawable/eel_mask3"/>
    <item
        android:drawable="@drawable/eel_mask4"/>
    <item
        android:drawable="@drawable/eel_mask5"/>
    <item
        android:drawable="@drawable/eel_mask6"/>
    <item
        android:drawable="@drawable/eel_mask7"/>
</layer-list>
```
效果图为：

![](http://cl.ly/0s3m243m102E/snake.tiff)

在UIxml文件中，我们可以像引用正常资源一样引用我们新建的eel.xml文件，就像引用图片资源一样。

在项目中，我们用LayerDrawable来引用这个资源。

当然，这里只是介绍了一下Layer-list最基本的用法，还有很多属性和用法，详情参见各大搜索引擎。
