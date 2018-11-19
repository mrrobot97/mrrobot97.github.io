---
layout: post
author: mrrobot97
title: 使用MonkeyRunner进行Android应用自动化测试
---

>MonkeyRunner是谷歌官方SDK tools 中为我们提供的一个程序自动化测试工具，需要开发者编写相应的python自动测试脚本，开发者可以自定义测试流程，功能很全面，可以模拟各种事件。

monkeyrunner位于SDK/tools/monkeyrunner,这是一个可执行脚本，我们编写自己的测试脚本然后用monkeyrunner运行即可。

谷歌提供了一个python库com.android.monkeyrunner，便于我们编写测试脚本。看一个脚本demo:
monkeyrunner.py 

```
from com.android.monkeyrunner import MonkeyRunner, MonkeyDevice, MonkeyImage


device = MonkeyRunner.waitForConnection()

package = 'me.mrrobot97.swipeback'

activity = 'me.mrrobot97.swipeback.MainActivity'

runComponent = package + '/' + activity

device.startActivity(component=runComponent)

MonkeyRunner.sleep(1)

for i in range(0, 5):
	device.touch(720, 1280, 'DOWN_AND_UP')
	MonkeyRunner.sleep(0.5)

for i in range(0, 5):
	device.drag((1, 1280), (720, 1280), 0.5, 50)
	MonkeyRunner.sleep(0.7)

img=device.takeSnapshot()

img.writeToFile('screen.png','png')

print 'finished'


```

三个module分别为MonkeyRunner,MonkeyDevice,MonkeyImage，每一个都包含了许多与APP交互的API，可以在网上查到相应的文档。

终端运行:

```
 monkeyrunner monkeyrunner.py 
```

即可在虚拟机或真机中观察到效果:

![Demo](https://blog-1256554550.cos.ap-beijing.myqcloud.com/auto-test.gif)

关于MonkeyRunner具体的用法请上网查API文档。
