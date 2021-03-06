## 23 | 基础篇：Linux 文件系统是怎么工作的？

### 笔记

文件系统是对存储设备上的文件，进行组织管理的一种机制。而 Linux 在各种文件系统实现上，又抽象了一层虚拟文件系统 VFS，它定义了一组，所有文件系统都支持的，数据结构和标准接口。

VFS 内部又通过目录项、索引节点、逻辑块以及超级块等数据结构，来管理文件。

- 目录项，记录了文件的名字，以及文件与其他目录项之间的目录关系。
- 索引节点，记录了文件的元数据。
- 逻辑块，是由连续磁盘扇区构成的最小读写单元，用来存储文件数据。
- 超级块，用来记录文件系统整体的状态，如索引节点和逻辑块的使用情况等。

其中，目录项是一个内存缓存；而超级块、索引节点和逻辑块，都是存储在磁盘中的持久化数据。

【应用程序  -> VFS -> 文件系统 -> 存储设备】

### 发现面试题
  无
### 专栏金句
### 发现专栏精选留言

## 24&25 | 基础篇：Linux 磁盘I/O是怎么工作的

### 笔记

磁盘类型：机械与固态；
三层架构：文件层 -> 通用层 -> 设备层

### 发现面试题
1. 请简单说一下，把磁盘引入服务器，有哪些不同的架构。

### 专栏金句
1. 不要孤立地去比较某一指标，而要结合读写比例、I/O 类型（随机还是连续）以及 I/O 的大小，综合来分析。
2. 在基准测试时，一定要注意根据应用程序 I/O 的特点，来具体评估指标。
3. 注意结合读写比例、I/O 类型以及 I/O 大小等，进行综合分析。

### 发现专栏精选留言

无论机械磁盘，还是固态磁盘，相同磁盘的随机 I/O 都要比连续 I/O 慢很多。为什么？
	- 对机械磁盘来说，由于随机 I/O 需要更多的磁头寻道和盘片旋转，它的性能自然要比连续 I/O 慢
	- 对固态磁盘来说，虽然它的随机性能比机械硬盘好很多，但同样存在“先擦除再写入”的限制。随机读写会导致大量的垃圾回收，
  所以相对应的，随机 I/O 的性能比起连续 I/O 来，也还是差了很多。
	- 连续 I/O 还可以通过预读的方式，来减少 I/O 请求的次数


## 26|案例篇：如何找出狂打日志的“内鬼”？

### 笔记

排查思路
1. 观察 CPU 和内存的使用情况： `top`,按1切换到每个CPU的使用情况;确定是否 iowait很高（wa指标）；
2. 若是，观察 I/O 的使用情况：`iostat -x -d 1 #-d表示显示I/O性能指标，-x表示显示扩展统计（即所有I/O指标）`
   - 超慢的响应时间(w_await)和特长的请求队列长度(aqu-sz) ?
3. 查看每个线程的IO情况：`pidstat -d 1`,找到最耗IO的线程号
4. 观察系统调用情况,读写文件必须通过系统调用完成:`strace -p pid`
5. 查看进程打开文件(目录、块设备、动态库、网络套接字等)列表:`lsof -p ${pid}`

### 发现面试题

无

### 专栏金句

无

### 发现专栏精选留言

无


## 27|案例篇：为什么我的磁盘I/O延迟很高？

### 笔记

记录一下排查思路：
1. 看一下磁盘的整体情况：`df`
2. 观察 CPU 和内存的使用情况：`top`
3. 看CPU使用率情况：`ps aux | grep ${app_name}`
4. 看IO使用情况:`iostat -d -x 1`
5. 观察进程的 I/O 情况:`pidstat -d 1`
6. 查看进程读写文件情况：`strace -p ${pid}` 或者 `strace -p 12280 2>&1 | grep 'write|read'`
7. 跟踪内核中文件的读写情况，并输出线程 ID（TID）、读写大小、读写类型以及文件名称:`filetop -C`, 注意：filetop 是bbc包中的软件
8. 查看进程的关联的文件：`ps -efT | grep ${pid}`
9. 动态跟踪内核中的 open 系统调用:`opensnoop`
10. 查看应用程序的源码,修复
11. 验证
### 发现面试题

无

### 专栏金句

当你发现性能工具的输出无法解释时，最好返回去想想，是不是分析中漏掉了什么线索，或者去翻翻工具手册，看看是不是某些默认选项导致的。

### 发现专栏精选留言


## 28| 案例篇：一个SQL查询要15秒，这是怎么回事？

### 笔记

差不多的排查思路， 略

### 发现面试题
无
### 专栏金句

在性能分析过程中，能够综合多个指标，并结合系统的工作原理进行分析，对解释性能现象通常会有意想不到的帮助。

### 发现专栏精选留言


## 29| 案例篇：Redis响应严重延迟，如何解决？

### 笔记

差不多的排查思路， 略

### 发现面试题
### 专栏金句
### 发现专栏精选留言

## 30| 套路篇：如何迅速分析出系统I/O的瓶颈在哪里？

### 笔记

	- 先用 iostat 发现磁盘 I/O 性能瓶颈；
	- 再借助 pidstat ，定位出导致瓶颈的进程；
	- 随后分析进程的 I/O 行为；
	- 最后，结合应用程序的原理，分析这些 I/O 的来源


### 发现面试题
### 专栏金句
### 发现专栏精选留言

@黄智荣
一次io性能问题
数据写es，运行一段时间后，发现写入很慢，查io时发现，读的io很高，写的io很少，很奇怪只写数据还没查询，读的io使用率基本接近100%。
用iotop定位到es一些写的线程，将线程id转成16进制，用jstack打印出es的堆栈信息，查出16进制的线程号的堆栈。发现原来是es会跟据doc id查数据，然后选择更新或新插入。es数据量大时，会占用了很多读的io.
后面写es就不传id，让es自动生成。解决了问题。

@我来也

其实查找其他方面的问题也都是这样啊.一步一步缩小范围.
首先,确定有没有瓶颈产生,或者有哪方面的瓶颈.
其次,看是谁导致的.
再次,是谁操作什么导致的.
最后,结合实际,给出解决方案.

## 31| 套路篇：磁盘 I/O 性能优化的几个思路

### 笔记
### 发现面试题
### 专栏金句
### 发现专栏精选留言

## 32| 答疑（四）：阻塞、非阻塞 I/O 与同步、异步 I/O 的区别和联系

### 笔记
### 发现面试题
### 专栏金句

仅仅是不恰当的选项，都可能会导致性能工具“犯错”，呈现这种看起来不合逻辑的结果。

### 发现专栏精选留言