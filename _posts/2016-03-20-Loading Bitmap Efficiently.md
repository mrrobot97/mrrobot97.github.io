---
layout: post
title: Loading Bitmap Efficiently
---

#### Android系统为每个程序都分配了一定的内存，当我们的程序加载比较大的bitmap时十分容易出现OOM(Out Of Memory)的错误，本篇文章就讲述如何高效地加载一个Bitmap.

文章分析来源于Google官方给的Training [Displaying Bitmaps Efficiently](http://developer.android.com/training/building-graphics.html)

首先我们获取要加载的图片的宽与高，方法如下：

```
BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;
BitmapFactory.decodeResource(getResources(), R.id.myimage, options);
int imageHeight = options.outHeight;
int imageWidth = options.outWidth;
String imageType = options.outMimeType;


```

然后通过比较图片实际的宽高与我们需要显示在屏幕上显示的大小，调整BitmapFactory.Options的inSampleSize,注意通过设置options.inJustDecodeBounds=true，我们可以至加载图片的大小和类型，而不会真正地讲图片加载到内存中。

```
public static int calculateInSampleSize(
            BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;

    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) > reqHeight
                && (halfWidth / inSampleSize) > reqWidth) {
            inSampleSize *= 2;
        }
    }

    return inSampleSize;
}
```

options.inSampleSize是用来调整图片的压缩比例的，比如inSampleSize=2，就讲图片的width和height分别设置为原图片的一半，这样压缩后的图片的体积就只有原来的1/4了。

最后当我们获取到合适的inSampleSize后再将options.inJustDecodeBounds设置为true，就可以真正地将压缩后的图片加载到内存中去了。

```
public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
        int reqWidth, int reqHeight) {

    // First decode with inJustDecodeBounds=true to check dimensions
    final BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeResource(res, resId, options);

    // Calculate inSampleSize
    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

    // Decode bitmap with inSampleSize set
    options.inJustDecodeBounds = false;
    return BitmapFactory.decodeResource(res, resId, options);
}
```