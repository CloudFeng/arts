# 第三周：（2020.06.08~2020.06.14）| 深圳 2020.06.14

本周的文章都是在地铁上看的，之前打算是白天看，晚上回来整理笔记。后面加班回来晚，所以就一直没有整理。所以都堆积到周六日两天，发现整理的时候，不知道如何整理。周六带儿子，晚上居然没心情整理。周日带着儿子去找早教。下午就没陪儿子出去游泳了，在家整理笔记。但整理特别痛苦，看过的知识一点印象都没有，只是零星记得一点命令。花了2个小时才整理出2篇，中间解决一个问题花挺多时间的：

> $ docker exec --privileged  -it d53754a3f26e /bin/bash
root@d53754a3f26e:/# dmesg | grep -i "Out of memory"
dmesg: read kernel buffer failed: Operation not permitted
root@d53754a3f26e:/# sysctl -w kernel.dmesg_restrict=0
sysctl: setting key "kernel.dmesg_restrict": Read-only file system

后面找资料，重新装了一个ubuntu的镜像：`docker run --privileged -ti --name ubuntu_two ubuntu:18.04 /bin/bash`。

作者提到: "你想要做成某件事情，结果应该怎么评估？" 那我购买此专栏以及参加TalkGo读书第一期的目标是啥，如何去衡量呢。学习排查思路，认识志同道合朋友。但太过于宽泛，是否有更为详细或者用数字化衡量呢？

这周需要改进地方，还是每日整理，不要堆积到周六日。周六日的时间已经不属于自己了，需要及时调整。