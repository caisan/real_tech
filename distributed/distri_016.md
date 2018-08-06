# 文件系统的一致性问题 (crash consistency)

在存储系统中都会存在(数据库、文件系统、dedup系统 ...)，即系统遭遇断电、
崩溃等情况时，相关联的数据没有全部持久化导致的不一致。

根本原因是存储系统中底层的一次I/O，无法保证上层的一次
请求的相关联的数据的原子写入，所以需要上层的机制保证一
致性。 (~~相关联的数据一般指metadata和data，如文件系统中的inode和用户数据~~） 

| 数据块 | inode块 |bitmap块|一致性|备注|
|--------|--------|-------|----|---|
|N|N|N|一致|相当于什么也没干，FS一致，虽然写没有成功|
|N|N|F|不一致（bitmap和实际占用空间）|若bitmap记录了分配了新空间，可能导致“空间泄露”（道理类似内存泄漏），|
|N|F|N|不一致（inode指针和实际存储位置）|这时inode指向了并未更新的数据块，后续的读操作会返回一堆“垃圾”数据|
|N|F|F|不一致|同上，可能读到“垃圾”数据|
|  F  |    N    |    N |一致|虽然写请求没有成功，但因为bitmap和inode都没变，写入数据块只是相当于白写了，没有引起不一致|
|F|N|F|不一致|由于inode没有更新，可能读不到更新的数据块|
|F|F|N|不一致（bitmap空闲空间和实际所用空间）|未更新的bitmap可能导致再次申请时覆盖有效数据|
|F|F|F|一致|请求正常完成了|

上表整理自OSTEP笔记[1]，假设了一个有data和inode、bitmap两种metadata的文件系统。多种metadata之间的一致性通常最麻烦：如上表的inode和bitmap，他们之间存在相同冗余信息（bitmap可以从inode推导出，但是这个推导是要遍历所有inode的，bitmap的作用就是用冗余的信息换取性能），并且由于并非在一个磁盘块，无法原子地同时更新，所以如果掉电时只有两者之一成功更新了，那么它们之间相同的信息便存在了不一致。metadata和data之间也存在不一致的情况：如上表中，若两种metadata都更新好了，但是data写到一半掉电了，那么下次开机后根据metadata读data时就会读到坏的数据，因此可以称为不一致。

这就需要额外的手段保证一致性，常用的方法有: WAL(Write ahead logging, 也叫journaling), 
CoW(Copy-on-Write, 也叫shadow paging), log-structuring, 
soft updates。当然对于一个对任何事要求都很低的人来说，后一种情况（metadata和data的不一致）可能并不算不一致，因为文件系统没有被破坏，还会正常工作，只是被FS服务的用户或应用得到了错误的数据，谁叫他把电线拔了呢😂。因此对一致性的定义、对一致性强弱的要求也是因人而异，因系统设计目标而异。

比如，ext4便提供了3种logging模式：journal, ordered(default), writeback，这三种方法对一致性的强度依次减弱，journal是把所有data、metadata先进行logging；ordered是用ordered write的方法保证data和metadata的一致性，用logging保证不同类别metadata之间的一致性，ordered write指但是先写data完成，再写metadata的顺序，data因此也不用进行logging；writeback则不管data和metadata的先后顺序，data也不写log，可能刚刚提到的要求很低的人和对性能要求更高的人才会用这个参数吧。

```
摘自kernel文档[2]
data=journal		All data are committed into the journal prior to 
                        being written into the main file system.  Enabling
			this mode will disable delayed allocation and
			O_DIRECT support.

data=ordered	(*)	All data are forced directly out to the main file
			system prior to its metadata being committed to the
			journal.

data=writeback		Data ordering is not preserved, data may be written
			into the main file system after its metadata has been
			committed to the journal.
```



---
[1] http://blog.jcix.top/2018-02-24/ostep_note3/#71_Crash_Consistency

[2] https://www.kernel.org/doc/Documentation/filesystems/ext4.txt

## Journaling

本wiki的[[Log-Structured、Copy-on-Write和Redirect-on-Write|distri_005]]条目介绍了logging(journaling)和log-structing的区别。

Journaling 即 Write-ahead logging，以Ext3文件系统为例，其log机制和数据库中所说的redo log类似[1]。先写log，再写数据，如果发生crash，恢复时redo即可。除了对metadata存两遍的ext3默认的Ordered模式外，还支持Journal模式(data和metadata都存两遍)和Writeback模式(只有metadata写两遍，不同于Ordered模式的是，第二遍写完时并不保证data已经落盘，而Ordered模式中，要等数据块落盘之后再将metadata落盘)。

---
[1] https://pdfs.semanticscholar.org/presentation/2d01/803c1c7c21f3e7c4b1a2230b2e78a00d0897.pdf

## Log-structuring

其特点是追加写。所有的写操作都变成了顺序写，带来的后果是顺序读操作可能都变成随机读了。

## Copy-on-write (shadow paging)

## Soft Updates