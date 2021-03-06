# 虚拟机的资源隔离 / 利用QEMU Monitor限制内存、I/O和vCPU数

## 资源隔离的思路

虚拟机或者容器的资源隔离：CPU可以用cgroups([DOC](https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt))进行限制，内存(缓存)可以单独开辟，网络可以用Traffic Control([HOWTO](http://linux-ip.net/articles/Traffic-Control-HOWTO/))。

FAST 17上FlashBlox: Achieving Both Performance Isolation and Uniform Lifetime for Virtualized SSDs 这篇paper提出用open-channel SSD将SSD也进行资源隔离，将每个chip/channel/die分给特定的VM/container。

对于IO的限制，QEMU也提供了throttling的方法，如下会写到。

## 接入QEMU Monitor

* 启动QEMU虚拟机时，加入`-curses`启动参数，然后在虚拟机开始启动后按`ESC+2`进入QEMU Monitor

* 类似上边的方法，也可以加入`-vnc :1`启动参数，在VNC界面调出QEMU Monitor

* 还可以选择加入`-qmp unix:./qmp-sock,server,nowait`参数，这时会在当前目录新建一个名为“qmp-sock”的Unix Socket文件，QEMU源码树的`scripts/qmp`文件夹中已经提供了相关的连接脚本，可以直接这样连接：

```bash
/PATH_TO_QEMU_SRCS/scripts/qmp/qmp-shell -H ~/vmimgs/qmp-sock
```
注意其中的`-H`是指以HMP模式启动QEMU Monitor，如果不加是以low-level的QMP模式启动。

然后就会出现欢迎语和以`(qemu)`开头的交互式qmp-shell。


## 内存
* 利用balloon限制内存

开启虚拟机时需要加入参数`-balloon virtio`

在QEMU Monitor中，使用`info balloon`可以查看当前的内存，使用`balloon [mem_size]`可以将当前内存限制在某个值。

在开启虚拟机的时候会用`-m`设置内存，这个是balloon的最大内存值。

* 利用hotpluggable memory来限制内存

?貌似还需要在虚拟机内插拔内存，待续。。

* 一个自动动态通过balloon改变guest内存的补丁，貌似并没有被并入主线：

https://www.linux-kvm.org/page/Projects/auto-ballooning

## 限制I/O

是利用QEMU Monitor的`block_set_io_throttle`命令。基本格式：
```bash
block_set_io_throttle device bps bps_rd bps_wr iops iops_rd iops_wr
```

但是QEMU 2.8 这个功能存在BUG不能用，升级到2.9以上后又好了。

例子：
```bash
# 查询磁盘设备
info block
# 进行I/O限流，比如这里是限制virtio2这个设备的iops为100
block_set_io_throttle virtio2 0 0 0 100 0 0
```

## 插拔vCPU

待续。。。

---


[1] Documentation/QMP, https://wiki.qemu.org/Documentation/QMP

[2] QEMU CPU Hotplug,http://events.linuxfoundation.org/sites/events/files/slides/CPU%20Hot-plug%20support%20in%20QEMU.pdf

[3] Features/CPUHotplug, https://wiki.qemu.org/Features/CPUHotplug

[4] qemu-docs, memory-hotplug, https://github.com/qemu/qemu/blob/master/docs/memory-hotplug.txt

[5] I/O bursts with QEMU 2.6, https://blogs.igalia.com/berto/2016/05/24/io-bursts-with-qemu-2-6/

[6] QEMU DOC, https://qemu.weilnetz.de/doc/qemu-doc.html

[7] docker runtime resources constraint,  https://docs.docker.com/engine/reference/run/#block-io-bandwidth-blkio-constraint