#  升级 Intellij Idea 报错

## 升级Intellij Idea报错

把Intellij IdeaI版本升级到2019.2版本，然后写了一个测试案例运行的时候报错，如下：

> Class JavaLaunchHelper is implemented in both /Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home/bin/java (0x10d19c4c0) and /Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home/jre/lib/libinstrument.dylib (0x10ea194e0). One of the two will be used. Which one is undefined.

## 解决

环境信息如下：
- 系统：macxos
- JDK: jdk1.8.0_121
- Intellij Idea版本： 2019.2

在网上找了一下[帖子](http://stackoverflow.com/questions/43003012/objc3648-class-javalaunchhelper-is-implemented-in-both)，说是jdk版本的bug,只需要：在Intellij Idea中设置：`idea.no.launcher=true`
操作路径：`Help->Edit -> Custom Properties`, 但是没有用。最后升级了JDK版本，解决了。

