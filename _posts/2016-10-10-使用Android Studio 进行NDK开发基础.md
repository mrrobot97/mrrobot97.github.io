---
layout: post
author: mrrobot97
title: 使用Android Studio 进行NDK开发基础
---

##配置NDK环境
在SDKManager中下载NDK依赖包以及额外工具，只需要勾选NDK、LLDB、CMAKE：

![](https://developer.android.com/studio/images/projects/ndk-install_2-2_2x.png)

安装完成后，重启AndroidStudio即可自动配置好NDK开发环境。

##JNI调用

###定义本地Native方法

```
public class HelloJNI {
    
    public native String getStringFromC();
}

```

此时getStringFromC()方法会报错，这是由于找不到这个方法，无视之，直接build project。

###生成头文件

build成功后进入app/build/intermediates/classes/debug目录下，使用命令行javah生成HelloJNI中的native方法对应的头文件：

```
javah -jni me.mrrobot97.hellojni.HelloJNI
```
注意包名以及类名都要正确，成功运行后在debug目录下就会生成一个头文件`me_mrrobot97_hellojni_HelloJNI.h`，我们将这个头文件拷贝至AndroidStudio的JNI目录下，没有这个目录就手动创建一个，然后根据这个头文件创建对应的C文件，同样放置于JNI目录下：

```
#include "me_mrrobot97_hellojni_HelloJNI.h"

JNIEXPORT jstring JNICALL Java_me_mrrobot97_hellojni_HelloJNI_getStringFromC
  (JNIEnv * env, jobject obj)
  {
        return (*env)->NewStringUTF(env, "Hello JNI!");
  }
```

###修改gradle
然后修改module级别的build.gradle文件，在defaultConfig中加入如下配置：

```
ndk{
            moduleName "hellojni"
   }
```
这里的moduleName就是我们要生成的.so文件的名字。修改gradle.properties,加入如下：

```
android.useDeprecatedNdk=true
```

然后就可以build了，build后会在app/build/intermediates/ndk/目录下生成对应的.so文件。有了.so文件我们就不需要JNI目录下的C文件了，直接在HelloJNI类中加入:

```
static {
        System.loadLibrary("hellojni");
    }
```
就可以使用我们定义的native方法了。

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        HelloJNI helloJNI=new HelloJNI();
        Log.d("yjw",helloJNI.getStringFromC());
    }
}
```

运行项目，查看log:

```
10-10 05:53:43.958 1811-1811/? D/yjw: Hello JNI!
```

证实我们已经成功调用了.so文件来实现native方法调用了！

上面的方法是针对只有源C文件而无.so文件的使用步骤，如果我们有现成的.so文件，直接调用System.loadLibrary即可。