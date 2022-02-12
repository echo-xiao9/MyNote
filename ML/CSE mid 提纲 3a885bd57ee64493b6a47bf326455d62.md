# CSE mid 提纲

# 1. Intro

**14 Properties of Computer Systems**

1. Correctness   2. Latency   3. Throughput   

4. Scalability(当系统scale变化很大，是否还有很好的性能） 

5. Utilization (系统的使用率)  6. Performance Isolation  系统之间的隔离（就算在一台host) 

7. Energy Efficienc   8. Consistency（数据的一致性，比如高铁票两张X ）   *9. Fault Tolerance   10. Security   11. Privacy   

12. Trust   13. Compatibility （兼容性）   14. Usability(是否用户友好）

**M.A.L.H**

Modularity

– Split up system
– Consider separately

– "Divide-and-conquer" technique

Abstraction
– Interface/Hiding
– Avoid propagation of effects

– Limiting the impact of faults

Layering
– Gradually build up capabilities

Hierarchy
– Reduce connections
– Divide-and-conquer

# 2. Scalability

**Distributed system**: a collection of independent/autonomous connected through a **communication network working** together **hosts** to perform a service

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled.png)

**结构：**

1. 每次从db找太慢 → cache → 分布式caching server 
    1. memcache 的client要知道那一台cache放了data.
        
        使用consistent hashing
        
2. 网络请求太多 
    1. 负载均衡：把请求分发给不同的服务器
    2. CDN：买地域服务器，缓存这个地区访问的文件
3. 数据库成为瓶颈
    1. 读写分离： primary-backup replication( 带来consistency的问题，记得后面解决方案！）
    2. 表太大，把单张表分成多张，consistency问题
4. 分布式文件系统，向外提供统一的文件接口
5. 使用不一样的server来提供不同应用：micro-servers, 通过消息队列来交流 Kafka
6. 分布式计算：处理复杂的请求，离线计算。热搜，金融欺诈
7. 分布式框架：批处理、图处理、机器学习

### 分布式系统的挑战

极其容易出现：fault( active and nonactive)→ error → failure

不可靠的网络：

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%201.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%202.png)

### 分布式问题解决方案

**保证Availability：**

1. 冗余 Redundancy （增加replicas) → challenge: consistency
    - RAID (Redundant Array of Inexpensive Disks)
    - Primary-backup replication
    - Replicated state machine
    
    解决： 一致写（3个写2个交集。。） primary
    
2. 重试 retry
    1. 重启→ 要保证consistency，这样就要求stateless, 或者说有些丢与consistency的要求没有这么高，比如Google search

**处理network partition: 部分区域网断了**

### CAP

- 以下三点不可同时满足：
    1. Consistency：所有节点同时看到同样的数据
    2. Availability：每个请求均响应，不管是响应成功还是失败
    3. Partition Tolerance：信息丢失和部分系统故障，系统继续跑，无法预估，因为网络难count on
- 两种选择：
    1. AP：保证用户体验，不一致没关系，e.g.买书
    2. CP：保证数据一致，拒绝用户访问，e.g.火车票、金融
    

# 3. Inode-based file system

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%203.png)

## **L1: Block layer**

Superblock contains  metadata:

- Size of the blocks
    
    block size 是软件可以调整的，硬件是4K，如果太小了，读取很慢，如果太大了，block可能没存满（block是最小单位），可能会被浪费
    
- Number of free blocks
    
    bitmap
    
    一个4KB的bitmap 可以存 4K*8 个block 就对应了4K*8*4K=128M 的磁盘空间。其实是可以对应比较大的。
    
- A list of free blocks
- Other metadata of the file system (including inode info)

## **L2: File Layer → inode**

```cpp
struct inode
	integer block_nums[N]
	integer size
	integer type // directory or normal file or symbolic link!
	integer refcnt
```

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%204.png)

## L3: inode Number Layer

inode table: 存了所有inode， inode number就是inode在这个里的序号。然后会记录每个inode是否空。一个block存4个inode.

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%205.png)

## L4: File Name Layer

Mapping 在directory 里面做name和inode number 里面的对应。

[directory](https://www.notion.so/51ec02962bbf41e1b7045df3528146b2)

procedure name_to_inode(string filename, integer dir)-> integer
return LOOKUP(dir, filename)

## L5: Path Name Layer

目的是层次化的命名 Structured naming: E.g. "projects/paper"

procedure PATH_TO_INODE_NUMBER(string path, integer dir)

LINK： 快捷方式 LINK("Mail/inbox/new-assignment", "assignment")

UNLINK： 移除binding

- 不可以有指向directory的link 因为会出现circle
- 在directory里面的. 和.. 不需要知道名字，就用这两个对应inode number就可以了。
- 当refcnt=0 这个文件就可以被删除了。

RENAME 应该是一个atomic action

1 UNLINK(to_name)
2 LINK(from_name, to_name)
3 UNLINK(from_name)

12 之间crash → to_name的文件lost

23之间crash → 两个nam                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             e指向同一个inode number

## L6: Absolute Path Name Layer

![? why  no sharing](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%206.png)

? why  no sharing

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%207.png)

## L7: Symbolic Link Layer

soft link (symbolic link) 做法就是再新建一个inode指向这个文件。hardlink本质就是在目录文件里面加一条。size是很小的

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%208.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%209.png)

ls -il

如果对于symbolic link :  cd /into/a/symlink/, cd.. 返回的是之前的目录，而不是上级目录，这是bash的优化，否则使用cd -P 可以进入上级目录。

 

# 4. File System API

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2010.png)

read before write, 因为每次是一整块写入的。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2011.png)

？ 先写数据，再inode, 再bitmap

对应？

FAT file system 

文件就拿链表把block链接起来，File metadata: name & size

!看一下回放GFS

### FAT

# 5. Remote Procedure Call (RPC)

目标：To make distributed computing look more like centralized computing. It should appear to be the programmer that a normal call is taking place. 简化网络call ,分布式像本地调用

A message may contain:
– Service ID (e.g., function ID)
– Service parameter (e.g., function parameter)
– Using marshal / unmarshal

A stub in distributed computing is a piece of code that converts parameters passed between client and server during a remote procedure call (RPC).分布式计算中的存根是一段代码，用于在远程过程调用(RPC)期间转换客户机和服务器之间传递的参数。

RPC uses stubs to avoid handling argument encoding/decoding and
send/receiving messages, message transports, etc

### Encoding/decod = Marshal / Unmarshal in RPC

就是序列化

使用RPC传递需要注意correctness，使用压缩算法， evolvable(前后兼容）

向前兼容：前面的版本可以保留对于旧版本的解释，这样就可以decode旧版本

向后兼容：旧版本可以省略新版本新加的东西。

RPC的帮助：

1. RPC simplifies programming w/ an interface similar to local function call. 简化编程，就像调用本地接口
2. RPC uses stubs to avoid handling argument encoding/decoding and
send/receiving messages, etc.  对收发消息更方便了（stab?)
– Ensure correctness & efficiency

![？ w/rpc是什么](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2012.png)

？ w/rpc是什么

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2013.png)

### RPC处理failure

可能的情况

1. Request may have been lost (e.g., someone unplugged a network cable).
2. Request may be waiting in a queue and will be delivered later (e.g., the
network or the recipient is overloaded).
3. The remote node may have failed (e.g., it crashed or it was powered down)
4. The remote node may have temporarily stopped responding (e.g., it is
experiencing a long garbage collection pause)
5. The remote node may have executed the function, but the response has
been lost on the network (e.g., a network switch has been misconfigured).
6. The remote node may have executed the function, but the response has
been delayed and will be delivered later (e.g., the network or your own
machine is overloaded).

尽量使得at least once → exactly-once

1. 设计函数为幂等的
2. 不能保证幂等，服务器端记录transaction id, 如果id是一样的，就说明做过了，就发送一个响应就好，不用再做一次。但是这里会有一个问题，什么时候删除这些id记录，如果服务器重启了怎么保证id还在。(服务器是有状态的）
    1. 所以需要记录在硬盘里面，并且存储时间需要经验值推断。

# 6. Distributed file system NFS & GFS

virtual file system (VFS)

nfs client每次read, nfs server会像从来没有遇到这个文件一样，打开，读，关闭这个文件。client每次发过来的请求的信息都是完备的。所以就算中间server挂了，对application是没有影响的。并且由于发过来的是offset+ count，这样每次都从offset开始读，server不需要记录curser的位置。因此是无状态的。at least once. rpc client 到server的readrpc是无状态的。但是application端是有状态的，所有的state是由client来记录curser的位置。写操作也是一样的。close 不用调用NFS server， 因为早就close了，但是在NFS client里面的file table 要删掉directory。

### NFS（Network File System)

Design Goals (by Sun, 1980 s, designed for workstations)

- Any machine can be a client or server
- Support **diskless** workstations
- **Heterogeneous(不均一的）** system must be supported
    - Different HW, OS, underlying file system
- Access **transparency**
    - Use remote access model
- Recovery from failure
    - Stateless, UDP, client retries
- High performance
    - Use caching and read-ahead

read(handle, offset, count) 从offset 开始读，读count个

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2014.png)

READ_RPC 是幂等的，因为它有一个offset 和count，每次都调用seek找到位置

WRITE_RPC 也是幂等的，会覆盖同一个位置。

但是client是不是幂等的，有状态（curser的位置）

![1. inode有namespace ，属于特定的文件系统，一个server上可能有好多文件系统 2. server可能会对应的iNode number的文件被删除，又被赋值给别人了，所以需要对比版本 ](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2015.png)

1. inode有namespace ，属于特定的文件系统，一个server上可能有好多文件系统 2. server可能会对应的iNode number的文件被删除，又被赋值给别人了，所以需要对比版本 

![fh里面不传文件名，就是因为可能会有重命名的问题。但是如果传了inode是没有关系的。](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/%E6%88%AA%E5%B1%8F2021-09-30_%E4%B8%8A%E5%8D%8810.42.41.png)

fh里面不传文件名，就是因为可能会有重命名的问题。但是如果传了inode是没有关系的。

![会返回告诉fh过期了这么做是和Unix不同，但是这是一个tradeoff, 因为NFS server并不知道谁打开了文件，不然得加状态。](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2016.png)

会返回告诉fh过期了这么做是和Unix不同，但是这是一个tradeoff, 因为NFS server并不知道谁打开了文件，不然得加状态。

为了分布式保证性能+cache, cache+分布式需要保证一致性，这里有两种保证标准：1. 对所有读写都是最新的 2. 在close和open的时候保证是最新的。如果有人中间改了，就需要告诉所有人（这样服务器就有状态了）或者加锁

![截屏2021-09-30 上午11.08.02.png](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/%E6%88%AA%E5%B1%8F2021-09-30_%E4%B8%8A%E5%8D%8811.08.02.png)

**Problem with NFS**
File consistency
Assumes synchronized clock (a global clock)

- We have mentioned in lecture 02 , its challenging

Locking cannot work

- Separate lock manager added (stateful)

No reference counting of open files (stateless server)

- You can delete a file opened by yourself/others

And many more ...

# 7. **GFS** *The Google File System*

背景： 1. 文件大 2. 很多append操作（很大的数据） 3. 大量进程同时append一个文件 4. 存储时间久

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2017.png)

![截屏2021-09-30 上午11.27.45.png](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/%E6%88%AA%E5%B1%8F2021-09-30_%E4%B8%8A%E5%8D%8811.27.45.png)

### 读操作

![截屏2021-09-30 上午11.30.26.png](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/%E6%88%AA%E5%B1%8F2021-09-30_%E4%B8%8A%E5%8D%8811.30.26.png)

![截屏2021-09-30 上午11.33.55.png](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/%E6%88%AA%E5%B1%8F2021-09-30_%E4%B8%8A%E5%8D%8811.33.55.png)

### 写操作

首先client找到一个离自己最近的，让他成为master, 然后master找一个没有挂的（容错）变成primary， 给一个一定时间的租约，primary也可以申请延长。之后只有primary才有资格写这个chunck, 其他两个备份听primary怎么写

![截屏2021-09-30 上午11.33.59.png](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/%E6%88%AA%E5%B1%8F2021-09-30_%E4%B8%8A%E5%8D%8811.33.59.png)

![截屏2021-09-30 上午11.36.16.png](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/%E6%88%AA%E5%B1%8F2021-09-30_%E4%B8%8A%E5%8D%8811.36.16.png)

![截屏2021-09-30 上午11.36.23.png](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/%E6%88%AA%E5%B1%8F2021-09-30_%E4%B8%8A%E5%8D%8811.36.23.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2018.png)

最重要点：meta data和data的分离

master 先告诉data放在哪个trunck server上。

meta data必须保证consistency，所以meta data放在一台机器里面，而且是够的，因为元数据不需要大量的修改:1. 文件都是大文件，所以数据大，处理一个文件所需要的时间久，所以不会频繁的打开一个文件。 2. append only ,

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2019.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2020.png)

# 8. Batch Processing Systems & MapReduce

分布式计算challenge

1. Sending data to/from nodes
2. Coordinating among nodes
3. Recovering from node failure
4. Optimizing for locality

### 完整流程

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2021.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2022.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2023.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2024.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2025.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2026.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2027.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2028.png)

![截屏2021-10-09 上午10.33.44.png](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/%E6%88%AA%E5%B1%8F2021-10-09_%E4%B8%8A%E5%8D%8810.33.44.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2029.png)

map: length的映射

mapper: 收到数据，生成part发给reducer

特定的单词发给特定的reducer.eg. 所有的apple发给reducer1

方式： 如果有20个reducer, 有20个文件，每个文件里面有对于每台reducer哪些word分过去。每个mapper写20个文件，都分别喂给20个reducer。 reducer 接受每个mapper的东西再最后组合起来

word count:

举例：分类水果，mapper遇到一个水果，丢给对应这个水果的reducer, 然后reducer 来数一数我一共收到了多少个水果。所以mapper是直接面对输入，而reducer是输入KV 对，然后做整合。

## map reduce 如何解决分布式系统的问题

核心问题

1. machine failure 容错
    
    ![截屏2021-10-09 上午11.08.22.png](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/%E6%88%AA%E5%B1%8F2021-10-09_%E4%B8%8A%E5%8D%8811.08.22.png)
    

master 节点的状态跟新会写到GFS 里面。

因为master只有一台出错的概率小。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2030.png)

![master 状态](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2031.png)

master 状态

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2032.png)

1. Skipping bad record 
    
    如果有两台机器都崩了，就可能数据本身就有问题或者第三方库有问题，所以就跳过了。
    
    思路：在一个batch的工作环境里，很有可能是到最后好几个小时之后给报错。硬件出错可能是偶然的，重启一下，但是软件出错是长久的可能每次都会错，这时候就可以采用绕过。
    

## Locality

如果mapper所有都网络读的话很慢。但是有了GFS， 直接把mapreduce 的worker运行在chunk server,因为它一般做网络IO，但是CPU是空着的。所以这样物理上整合在了一起，逻辑上是分开的。

每个任务过来都会fork一个master, master里面有GFS lib,他知道mapper 需要喂什么文件，然后中间文件生成也是需要放到cs, worker的输入文件，中间文件都刚好放到这个机器上。

![IMG_8AEC4A74B9FB-1.jpeg](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/IMG_8AEC4A74B9FB-1.jpeg)

## Refinement: redundant execution

慢的机器各有各的原因，各种稀奇古怪的bug。

reducer 得等mapper做完，有的机器很慢，所以就先做完的帮助还没有做完的mapper做。做的慢的人可能就已经出错了。

### map reduce 的重大意义

在几千几万的机器上，做分布式系统，可以容错完成事情。是一件工程上的重大改进，在容错方面，给大家很多启示。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2033.png)

### 缺陷

1. 现在已经不用了，它不能适合所有的work,比如不能做pagerank,时间太久.
2. 前后衔接不好
3. 如何做sharding, 由应用的语义决定，无法自动化

## Dryad: Distributed Data-Parallel Programs from Sequential Building Blocks

把操作变成数据流，节点：计算，边：通信，节点：多个输入，多个输出。

Computations are expressed as a graph

- Vertices are computations
- Edges are communication channels
- Each vertex has several input and output edges

Dryad Job = Directed Acyclic Graph (DAG)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2034.png)

![3disk是GFS 3备份](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2035.png)

3disk是GFS 3备份

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2036.png)

继承了MapReduce的容错机制，比如节点挂了，再跑一遍。但是数据的流MapReduce是记在磁盘里，这个如果挂了，让上家在跑一次，递归上去，这是一个tradeoff. 但是可能挂的概率不高，比起写磁盘更搞笑。同时这个编程也更灵活了。结构更简化了

总结：

Sacrifice some architectural **simplicity** compared with MapReduce system design
Provide more **flexible** abstract to developers expressing their code as a **DAG**

# 9.  MapReduce to Graph

目的：request handling

## PageRank

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2037.png)

简化版 alpha =0

![第一轮迭代](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2038.png)

第一轮迭代

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2039.png)

图计算不用MapReduce： 存储太大、时间都耗费在了数据再各个层的传输上面，大量硬盘和内存的读写，效率很低。

于是改用了MPI / Pthreads,它是一个线程库，MPI是一个多机版的pthread

### Pthread and MPI

但是他们的parallel代码非常难维护，原因：他们把page rank 之类的算法和具体机器实现耦合的太紧了。

### 期望

我们希望得到一个抽象的，松耦合的编程框架，没有太多的底层细节。

**Friendly programming model**

– Focus on application’s logic, instead of system-level details
– Many programmers have little system background

**High performance**

– High utilization of computing resource
– Less network transfer

**Fault tolerance**

于是出现了google 的Pregel, 因为Google之前就拿了很多便宜的小型机做分布式集群，积累了很多经验。

# BSP and Pregel

message-passing

消息驱动,可以收发消息

shared memory 

可以看到很多信息，调用别人的信息和调用本机的信息一样，就调用函数不用考虑是本地还是别的机子。

顺序：算→ 传 → 等

如果没有barrier, 状态很难确定，可能一半新一半旧，一半旧的又有一半更旧。。

可以联想流水线的寄存器隔开

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2040.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2041.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2042.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2043.png)

### Pregel 的page rank程序实现

**应用程序员代码(上图）**

## fault tolerant

Failure detection: master pings each worker periodically via heartbeat

- Similar in MapReduce

Failure handling

- Checkpointing: at the beginning of a superstep, checkpoint the graph data to a persistent storage (e.g., GFS)，
    - 相当于每次存到git里面，一旦发生了错误就从里面再拿出之前版本的，但是这里有到底多久记录一次的问题
- After failure, can recovery from checkpoint & re-compute
    - For either worker or master
- The frequencies of checkpointing is determined by the mean time to failure
    - Can measure MTTF（Meantime to failure) in advance
        - 就是记录出错时间，每次大概多久会出错，就是大概在meantime to failure 之前去记录一下。

和mapreduce 的改进：减少了中间数据的磁盘读写。前者是要存磁盘读写很多，但是这里的中间状态直接变成message发出去，这里会有网络操作，比磁盘操作简单。后来问为什么mapreduce不要把东西都存成消息呢？好像说这样对容错什么的有了新的挑战...总之就是把数据不经过磁盘了，直接走网路。

问题：message的发送还是不够简单、计算冗余（8个次→36）、如果有人做的特别慢，有人拖后腿，由于是synchronize，如果没有饱和式救援的方式（快的人帮忙）就会拖后腿了。

为了解决这些问题，出现了shared memory

## Shared memory and GraphLab

每个人从自己的scope里面拉数据来，算自己的迭代，如果没有收敛，通知邻居算一下，如果收敛了，邻居要算一下，让邻居放到reschedule 的表格里面，如果要做并发，就要拆分成几个子图，但是拆开的节点的拓扑就破坏了，因此拆开的边的属于scope的外面的点要做一个ghost，完全拷贝数据，但是是只读的。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2044.png)

那么最后在更新，中间断开的节点的时候就有了sequential consistency的问题。

### sequential consistency

问题：出现了正确性无法保证的问题。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2045.png)

最简单的方法: full consistency。加锁。

Pregel把mapreduce 的写磁盘减少了，变成等的时候同步；使用shared memory 就是把Pregel的同步变成异步，这样就加速了。

# 10.Single-node key-value store/storage

总结：

这节课讲了KVSTore的存储，首先是存在内存，然后要到硬盘里面就遇到很多问题，最重要的就是平衡1.尽量满足磁盘的顺序读写 2.维护顺序，满足查找。

1. 每个KV都是一个文件→ 浪费
2. 多个数据合并放在磁盘→ 大量随机读写
3. 针对的数据：log structure file( append-only records)。在LSM tree的实现就很好的体现了这个tradeoff，首先在memtable就整合了很多update的操作，减少了硬盘的读写。其次在Level0 是每个sstable 顺序append这样就满足了顺序读写。部分有序：在整合的时候也是顺序写这些新的sstable. 然而在L1及以上每一层之间是有序的，这样就满足了顺序查找的要求。 和同样在磁盘读写很出名的B+ tree比起来，B+是全部有序，这样在写操作的时候就要定位到那个地方写，这样就出现了大量的随机读写很慢了。
4. 不用大文件，排序什么的有很大问题，fixsize/ trunck→ 小文件， 保证了文件之内(L0)或者文件和文件之间排序就做了tradeoff.
    
    L1之后有趣的是merge的时候顺便排序了，这样就减少了读写的次数。
    

# 11. Fault-tolerance: Crash Consistency

在更新文件需要至少多少次写操作？

1. 更新inode: size& pointer  
2. 更新bitmap  → 一个磁盘块被两个人指向，读写
3. 写data
有8种出错的可能。

写的顺序data, bitmap, inode

一次操作若干次磁盘写，中间状态会暴露出来。

## Sync Metadata Update + fsck

删除的顺序：先删除inode，再改bitmap.

如果 iNode 和bitmap 不一致，bitmap删除了，inode没删.那就让bitmap恢复到和inode一致， 符合All or nothing. 

1. 如果写了data, inode， 没有写bitmap. 那别人以为这个block是free的，指向它，会有一个磁盘块有两个文件指向。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2046.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2047.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2048.png)

Issues with Synchronous Write —— slow on normal operation & recovery

### write-back cache

用一个文件记录小文件的改写，一次刷到硬盘

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2049.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2050.png)

## log

Journaling Drawbacks

这样所有的数据都要写两次，先写block再写journal. 因为数据不是一样重要的，metadata最重要。所以我们journal只写metadata.

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2051.png)

![如果直接删除journal但是metadata已经写入了，就会有一致性的问题](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2052.png)

如果直接删除journal但是metadata已经写入了，就会有一致性的问题

这节课的内容里，第一个flush可以用hash(checksum)删除，第二个不能删除.

1. 写data, jounral of metadata到磁盘cache-》 flush(可省略） -》commit ( data 和 journal hash一下写到Jm里面）-》写Jc-》写metadata

# 12. Atomicity: All-or-Nothing

1. Synchronous meta-data update + fsck 
2. Logging (ext 3/4), and many others（shadow write)

解决方法1： shadow write

![解决问题，写到临时文件里面，这样所有的写操作都写到临时文件，如果断电了，就可以恢复。这样要求rename是atomic](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2053.png)

解决问题，写到临时文件里面，这样所有的写操作都写到临时文件，如果断电了，就可以恢复。这样要求rename是atomic

磁盘支撑：写4K的sector是原子的。

1. 磁盘有足够的能量可以写4K
2. 小电容保证数据继续写下去
3. 浪费一点容量：每次写3个4K

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2054.png)

![思维：通用问题→ 具体问题 ](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/IMG_244D52C66287-1.jpeg)

思维：通用问题→ 具体问题 

永远不要写一个只有孤本的文件，而是应该写一个它的copy

## shadow copy

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2055.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2056.png)

保证rename的原子性：硬件小电容/ RAID 

### RAID 顺序写三个磁盘

缺陷：

Works well for a single file
****Hard to **generalize to multiple files** or directories，Have to place all files in a single directory, or rename subdirs

Requires copying the entire file for any (small) change
****Only one operation can happen at a time
****Only works for operations that happen on a single computer, single disk

问题： shadow copy太依赖于文件系统了。不能跨机器。而且如果要改很小的，需要复制一整个。只能用于单机的，单磁盘文件系统。就有了下面的通用的log， 对于任何操作都可以用。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2057.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2058.png)

# Redo Log & Undo Log

日志什么时候可以删掉：日志的内容被写回到改写的地方。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2059.png)

如何利用log恢复

Read the log and recover states according to its content Rules:

1. Travel from end to start
2. Mark all transaction's log record w/o CMT log and append ABORT log
3. UNDO ABORT logs from end to start
4. REDO CMT logs from start to end （发现commit, 就从后往前一条一条redo）

log recover的种类

只做redo的recover， 当checkpoint的时候，如果没有commit， 就不写硬盘。

只有undo的recover，说明磁盘上的是最新的，那就是每次写都写到磁盘上。这样的应用不多。

REDO-only logging  Logging rules

- Append REDO log record before flushing state modification
- Uncommitted transactions can not flush state (w/o UNDO)
    
    E.g., achieved via caching the state modification to the memory
    

Recovery rules

- ABORT all log records w/o CMT log
- REDO all committed transactions

vb c容错和atomicity 有关系，allornothing 是他们的交集

好处：

1. 可以支持all or nothing

1. 每次都是append， 每次都是顺序写，所以对于磁盘的性能友好。

![T5 就是nothing](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2060.png)

T5 就是nothing

数据在log和home都有，如果log更新，redo。如果磁盘更新,但是log还没有与commit，undo,它是loser，T4的情况。

## shadow copy

MTTF: mean time to failure 

MTTR: mean time to repair 

MTBF: mean time between failure 

MTBF = MTTF + MTTR

$$
\begin{aligned}&M T T F=\frac{1}{N} \sum_{i=1}^{N} T T F_{i} \\&M T T R=\frac{1}{N} \sum_{i=1}^{N} T T R_{i}\end{aligned}
$$

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2061.png)

# 13. **Atomicity: Before-or-After & Transaction**

## race condittion

定义：两个或以上的主体，同时访问一个共享变量，至少有一个是写操作。

**Race Condition is Hard to Control**

**Heisenbugs** 

加了printf, bug不见了。

Solution-1: DMT (Deterministic Multi-Threading)

如果跑一次是正确的，就记下，就算有bug,但是不触发我们看不见。但是需要把调度的过程记下，变成确定性的。但是这个是很难保证的，但是在程序运行 的时候，中断和cache 是不确定的，这样使得每一次运行都是不一样的。
Solution-2: Record and Replay
录制下有bug 的代码是过程一步一步怎么样的，然后放映，找怎么出现的。

## **Transactions**

在并发，有很多断电因素的环境很好
**Transactions handle failure atomicity & race conditions
transaction 提供的重要特性：ACID**

**A:Atomicity 可以用log机制，shadow copy**

**C:Consistency 是应用自己定义的语义，比如在转账的时候钱的总数是一样的。**用户态的语义，给定一些不变量怎么保证不变，是算法层面的

分布式的consistency 是保证每个republica是一样的。

**I:Isolation**

两个同时进行的transaction效果要是像隔离的一样，没有相互干扰。

有很多级别，before-after是最简单的

**D:Durability**

Once a transaction is committed, it schanges(e.g.,writes) must durably 
stored to a persistent storage。（银行小票打出来了就是保证了）

不会导致数据的丢失。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2062.png)

**Ideal isolation level: serializability**

**Review: Why serializability is ideal?**

程序员擅长写单线程的程序，但是执行的时候希望是多线程，有了isolation 和consistency， 因为每次经过tx这个transaction ,两个状态是consistent的，而且transaction是isolate的，所以只要每个transaction是保证serialization， 就能保证全局是serilizable

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2063.png)

这两个有很多的指令运行顺序。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2064.png)

![如果100% T2在T1z之前要么看到0，0，要么看到20，30，但是中间看到了0，30，这样算不算正确呢？如果我们只care最后状态，OK。所以看下面有几种serializability](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2065.png)

如果100% T2在T1z之前要么看到0，0，要么看到20，30，但是中间看到了0，30，这样算不算正确呢？如果我们只care最后状态，OK。所以看下面有几种serializability

Final-state serializability: 其实用户并不关心每一条指令之间的顺序，但是关心结果之间的顺序。只要调度的结果和顺序执行的结果一样就可以了。

view: 在进程中间的read的结果和顺序执行一样。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2066.png)

**Conflict Serializability**

所有冲突的transaction顺序 和某一种顺序排列的冲突顺序一致。

调度的有很多种，只看冲突的部分。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2067.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2068.png)

例子：

## **Conflict Graph**

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2069.png)

![√](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2070.png)

√

![X 有环](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2071.png)

X 有环

![t1, 早于T3，但是两个有conflict,所以也有冲突。 ？？ right???](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2072.png)

t1, 早于T3，但是两个有conflict,所以也有冲突。 ？？ right???

serializability 的包含关系,对比Conflict & View serializatbility

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2073.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2074.png)

**Simple fine-grained lock： 锁的是数据**

要读写数据加锁，之后放开

**Simple fine-grained lock： 锁的是数据**

要读写数据加锁，之后放开

### **Solution#3: Two-phase locking**

two phase: lock phase & unlock phase

lock point：数据访问完了，lock拿完了。

一旦开始放锁就不能再拿锁。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2075.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2076.png)

**证明：two-phase locking 不会产出环**

假设有环，如下，任取一个开始执行，若x1先执行，则它拿到了xk和x2 的锁，执行完毕x2才能拿到锁，然后x3,x4,...xk 这样执行完毕。但是这时候由图xk→ x1, 也就是在xk 和x1的conflict中， xk 必须先于x1调度，于是违背了要求。于是反面得证。

注意这里的有环是有向的有环。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2077.png)

## **Optimizations: Read-write Locks**

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2078.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2079.png)

在某个物体的写锁被获得前，随便读。

## **Optimizations: Read-write Locks**

**Lock compatibility modes**

– Multiple-reader,single-writerprotocol
• Any number of readers is safe
• **Only one writer, waits for all reader to finish**
**• Suitable for applications with a lot of reading,因为reader之间没有竞争**
• A writer may be delayed indefinitely
• Can release reader-locks after lock point (may before commit)

DeadLock

乐观：死锁很少发生，要是发生了我们再解决。

悲观：经常发生，我们要设计让死锁绝对不发生。排序，这样就不会两个交错的拿了

执行到一半的时候该锁的范围增大了，我们需要使用rangelock

现实中被ignore了。

**Can two-phase locking discussed above truly satisfy serializability?**

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2080.png)

因为锁的时候只会锁我们访问过的变量，没有办法保证新的增的

这样就要使用Phantom lock或者range lock

但是这个是two phase lock 的问题吗？

是理论没问题，但是是一个实践上的问题，理论上比如有100行，应该每行都要加锁，但是实际上只有SS的那个加锁了，如果他运行的时候有人把一个改成了SS，那这个就没有被锁住了。

# 14.**Transaction: OCC**

**Methods for resolving the deadlock is not ideal**

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2081.png)

1. 需要提前知道拿什么锁维护一个序列 2. 维护谁拿了什么锁检测死锁，cost很大

所以我们能不能开始就不加锁。也就是用乐观的方法。

OCC：**Optimistic concurrency control**

Phase 1: Concurrent local processing

- Reads data into a read set
- Buffers writes into a write set

Phase 2: Validation in critical section

- Validates whether serializability is guaranteed:
- Has any data in the read set been modified?
    
    readset的数据有没有人修改
    

Phase 3: Commit the results in critical section or abort

- Aborts: aborts the transaction if validation fails
- Commits: installs the write set and commits the transaction， 如果没有人改就最快速度写回，但是中间万一有人呢？因为是乐观的

如果刚刚validation之后被人改了不行了。我们要保证第二步骤和第三步是一个critical section ,还是加锁（两个phase一起）

和two phase lock 的区别：  phase 2和phase 3运行时间很短， two phase lock 的逻辑计算部分是要加锁，它没有加锁。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2082.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2083.png)

readset 不用加锁，**只要给write set加锁**。假设T1 在验证后写之前read set的内容被T2写了，那相当于先做T1, 再做T2.

如果是validate的时候读到的值是改了两次，但是改回一样的，也不影响。

只要有锁，还是有**deadlock**, 比如「x,y」两个进程同时写，解决方法： 给writeset 的写顺序排序。2pl的transaction拿锁排序是由代码逻辑决定的，但是这里可以直接把变量排序就OK了。

两个线程的先后顺序影响结果，但是这个是应用负责的。

**OCC Advantages**

**Simple validation, which can achieve high throughput and scalability at low contention—— validation很简化，所以throughput会很高。**但是有一个适用的条件：contention必须很低**(数据的重复小，比如T1访问x,y,z, T2访问A,B,C），这样可能导致更多的abort.**

**Phase 1: Concurrent local processing**
– Reads data into a read set
– Buffers writes into a write set

这个是local的，不需要任何的锁优化

**Phase 2: Validation in critical section short at lo**

- Validates whether serializability is guaranteed:
- Has any data in the read set been modified?

**Phase 3: Commit the results in critical section or abort**

- Aborts: aborts the transaction if validation fails
- Commits: installs the write set and commits the transaction

**OCC's Problem:有可能False Aborts**
Some transactions aborted by OCC could have been allowed to commit without causing an unserializable schedule

Especially for TXs with a lot of reads

- E.g., read a lot of items → large read set → easier to abort(buffer 越大，越容易abort）
- May abort even under low-contention

![这里没有形成conflict但是被abort了，这个就是false abort](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2084.png)

这里没有形成conflict但是被abort了，这个就是false abort

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2085.png)

2pL 不会重复，就是在最后等锁。OCC一旦线程多了虽然等锁的时间很长，但是有大量的重复，严重的时候可能做一次，需要10次的重做。

所以可以做hybrid.(OCC + 2PL)

# 硬件支持Hardware transactional memory（RTM）

如果用软件写 A=READ(X), 不写A=X 不能维护readset

如果硬件维护，一旦发现transaction里面有load操作

cache coherence

restricted 

![a写成了20，要发一个广播，告诉所有人我改了a， 然后其他人都重新拿一下a. 基于systemcall](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2086.png)

a写成了20，要发一个广播，告诉所有人我改了a， 然后其他人都重新拿一下a. 基于systemcall

硬件一旦广播到自己发现readset被别人改了，就立即abort，然后跳到下面图的else.

![这个是软件的方法](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2087.png)

这个是软件的方法

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2088.png)

**drawback of RTM**

**limited working set**

如果write set 在cache放不下了，也会直接abort

同时不能跑太长，如果有中断就abort.

硬件实现transactional memory 是大势所趋，虽然有一些drawback，但是写代码很方便。

**Can we avoid locking/validation
for reads?**

### MVCC Multi-version Concurrency Control

occ 和 2pl都有缺陷，要使用lock 的根本原因是数据只有一份，别人只要读，读到的就是最新的。于是我们使用多个copy.

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2089.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2090.png)

读的时候只要一致就行，不要求读到最新的。

## **Snapshot Isolation : SI**

A popular multi-version concurrency control (MVCC) scheme Transactions:

- WRITEs a local buffer (similar to OCC)
- READs a "snapshot" of entire data image
- COMMIT only if no **write-write** conflict
- Install new versions of data

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2091.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2092.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2093.png)

如果都是读，就只要保证数据是一致的就行，旧一点没关系，就像我们自己做备份，读的时候有人写的话就不读最新的就好。

snapshot 并不能保证序列化，只能保证每一个snapshot的数据一致性，这个和git是一样的。之后有冲突需要用户fix conflict.

**Summary**
Transactions provide ACID properties
OCC \& 2PL are basic protocols to provide serializability

- Problem of 2PL: locking \& deadlock
- Problem of OCC: False abort \& livelock

Hardware transactional memory (HTM)

- Advanced CPU features inspired by ACID properties of TXs
- Commercial implementation of HTM uses OCC
- Leads to several drawbacks

The cost of concurrency control can be reduced w / relaxed isolation level

- Snapshot Isolation (require global counter, not serializable)

# 15. Distributed Transaction

问题：

然而引入了网络，就出现了lost、delay、 duplicated 问题，可以用RPC， 做at least once(不断发） at most once( 幂等做多次的效果等价于一次、有状态，记录（有很多记多久的问题））。

网络中的两难问题：到底是请求在发的时候丢了，还是已经做了但是ack丢了。

但是对于automicity（all or nothing) 两者都不不够，在分布式的情况下要满足任意一个节点的crash问题。

tentatively 临时的

如果另一个server没有commit，但是一个已经commit了，从逻辑上讲它也不应该commit。所以要有 2phase commit

### **Two-phase Commit**

tentatively 临时的

![只有最上层可以commit， 多层就下面的都是 tentatively  commit，上层收集完大家都commit 了，就是可以了](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2094.png)

只有最上层可以commit， 多层就下面的都是 tentatively  commit，上层收集完大家都commit 了，就是可以了

![最后N-Z 挂了，重启看一下log，发现有prepare，就问一下coordinator . 或者coodinator不断重发。如果发消息丢了，at least once, 重发](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2095.png)

最后N-Z 挂了，重启看一下log，发现有prepare，就问一下coordinator . 或者coodinator不断重发。如果发消息丢了，at least once, 重发

coordinate commit point 是OK之前， write ahead log. 就是ATM 的小票不然已经给别人OK了，但是自己没有写commit,那就不行了，告诉别人OK之前自己一定要commit。

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2096.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2097.png)

1. lost prepare 丢包, cor 重发
2. N-Z crash cor可以知道，因为心跳没有了，就给另外一个发abort.

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2098.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%2099.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20100.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20101.png)

有多个机器，没法同时写commit， 但是我们可以有一个coordinator 它commit就是真正的commit。这就是一个问题的划归，同样的思想也体现在atomicity 的转移使用shadow copy.

# **Replication Consistency**

冗余是容错的唯一方法, 有两个A-M server

一旦冗余，就一定有consistency的问题。注意这里的consistency是两个备份之间的consistency， 而ACID的consistency是事务的before-after, 中间不能被人改。

**Why Replication?**

For performance

- Higher throughput: replicas can serve concurrently（负载均衡）
- Lower latency: cache is also a form of replication,cache是上下层的replication

For fault tolerance 

- Maintain availability even if some replicas fail 3备份

悲观和乐观复习：

deadlock （悲观是排序，乐观是检测，并且打破环）

transaction before-after(occ 是乐观到时候abort，2pl保证就是拍好队的），

consistency（replication)乐观允许发生inconsistency， 我可以检测和纠正，悲观：不允许发生inconsitency

## 乐观悲观备份

Optimistic Replication

- Tolerate inconsistency, and fix things up later
- Works well when out-of-sync replicas are acceptable

Pessimistic Replication

- Ensure strong consistency between replicas
- Needed when out-of-sync replicas can cause serious problems

Optimistic Replication

记录时间，要是有冲突就用时间来对一下fix 允许冲突

**Consistency of Distributed Files**

时间在分布式系统很常用来确定新旧。

**Time Measuring**

石英振动

time period = count / frequency

电脑关机了，有电容继续供电往前走。但是我们还是需要同步来纠正误差。

**Clock Synchronizing**

**Synchronizing a clock over the internet: NTP**

永远不要写fixed timer, 每秒钟触发一次的函数XX

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20102.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20103.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20104.png)

这么调整时间的问题：如果是调慢，不可以突然把localttime 设置成比之前小，因为可能应用程序员需要写（currTime-beforeTime) 万一是一个unsigned,就可能溢出。所以我们就每次都让时钟走的慢一点，一直等到当前时间校准了为止。

**File Reconciliation with Timestamps**

如果分布式系统，两台机器上都改了同一个文件，比较新旧Mtime是没有意义的。因为是各自的修改。

所以我们用vector timeStamp.

时间向量，记录了两台机器的时间戳，

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20105.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20106.png)

Pessimistic replication(RSM & Paxos)

### **Quorum**

Qr + Qw > N replicas  → Qr和Qw 至少有一个是overlap的，所以一定有新的值

![2+2>3](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20107.png)

2+2>3

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20108.png)

Qr=2 是说，所有replica 都要写，是说有两个人反应了就往下走了。这样更快，这样写的latency就低. 对读友好的话是读操作比较多，对写友好是写操作比较多合适。

一般写很少，读很多，因为写比较慢。

使用到的分布式的原则： majority have the truth , 大部分人是对的。

**Quorum's Limitations**

Provide no before-or-after or all-or-nothing

- If reading \& writing requests come from a site - Easy...
- If reading from multiple sites, writing from one site
    - Need to maintain a version number at the writing site
- If writing from multiple sites
    - Need a protocol providing a distributed sequencer

Another complicating consideration

- Performance maximization
    
    ![初始值和一样的输入可以保证。 顺序一样很难保证。 deterministic 不一定保证比如锁，不同的CPU调度，以及可能有随机数。就算一样的代码和机器，都可能不一样的执行，尤其在并发环境下。如果我们都避开了这些，可能会是deterministic， 这个至少是可以控制的。但是same order是和网络相关，根植于分布式的。在这里client 需要知道request1 和2 怎么确定顺序。](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20109.png)
    
    初始值和一样的输入可以保证。 顺序一样很难保证。 deterministic 不一定保证比如锁，不同的CPU调度，以及可能有随机数。就算一样的代码和机器，都可能不一样的执行，尤其在并发环境下。如果我们都避开了这些，可能会是deterministic， 这个至少是可以控制的。但是same order是和网络相关，根植于分布式的。在这里client 需要知道request1 和2 怎么确定顺序。
    

# 16. **RSM & PAXOS
Consistency across multiple machines**

**Handling Network Partitions**

**Problem: replicas can become inconsistent**

– Issue:clients' requests to different servers can arrive in different order

**Replicated State Machines (RSM)**

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20110.png)

分布式里面有一个顺序是很难的？

**Primary/Backup Model**

分布式里面不能有顺序，就搞一个coordinator 来把分布式化成单一模式。同时为了保证输出的一样的，就选一个primary做事情，其他做primary。

如果primary 挂了，就用backup当primary

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20111.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20112.png)

coordinator不能只有一个否则成为瓶颈，所以多个，但是primary只有一个。如果网络断了，一个corr 认为正常，但是另外一个选了新的backup 当pri，这样就有两个pri,乱了。所以加一个view server来记录谁是pri, server ,有点像euraka

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20113.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20114.png)

### view server

![S1 和 S2 会定期发心跳，看谁挂了？？ view server不会成为瓶颈吗？感觉像是瓶颈转移了，还是因为事情做了分工？](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20115.png)

S1 和 S2 会定期发心跳，看谁挂了？？ view server不会成为瓶颈吗？感觉像是瓶颈转移了，还是因为事情做了分工？

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20116.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20117.png)

before S2 knows it's primary, it will reject any requests from clients
就是为了防止两个primary的情况。

![多了新规则： 如果没有收到backup的回复，就会任务失败，reject所有的请求。](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20118.png)

多了新规则： 如果没有收到backup的回复，就会任务失败，reject所有的请求。

![比如这个时候S2变成了pri, 就不会响应S1， S1 就得不到响应，就拒绝C的请求。这个时候对于S2， 没有backup, 也得不到响应，这样也是可以的，只要primary 只有一个就可以了。这样就保证了consistency。 如果S1和S2 都crash掉了就不行了。在这段时间， S2 肩负了所有的事情，他需要找一个S3 当一个backup, 或者等S1恢复链接了，变成backup.](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20119.png)

比如这个时候S2变成了pri, 就不会响应S1， S1 就得不到响应，就拒绝C的请求。这个时候对于S2， 没有backup, 也得不到响应，这样也是可以的，只要primary 只有一个就可以了。这样就保证了consistency。 如果S1和S2 都crash掉了就不行了。在这段时间， S2 肩负了所有的事情，他需要找一个S3 当一个backup, 或者等S1恢复链接了，变成backup.

primary S1→ S2 的转换点： S2 收到了VS的任命。

但是View server 又变成了瓶颈，但是陷入了套娃，所以我们引入了**Paxos**

## **Paxos** Distributed consensus  mechanism

No guaranteed termination 不一定能讨论出结果

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20120.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20121.png)

一个人可能是第一轮的proposer, 第二轮的acceptor. 难点是几轮会嵌套，并发。因为网络的丢包等等问题。

commit: 和2 Phase commit像，只是把所有人同意，变成了majority同意。

majority： 和quorum 像， 重点在于majority只有一个

问题

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20122.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20123.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20124.png)

![Untitled](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/Untitled%20125.png)

当majority 收到了< accept, foo> 并且记入自己的日志，就是commit point

pexis 有趣的点是：commit point  是一个多台机器决定的，是一个没有人知道的一个点，和2Phase commit不一样， 他是coordinator的commit 决定的。这样才能不依赖于一个点。

所以一旦有一个leader被选出来了，一定是被majority同意的。

# 试卷

```
void wait (unique_lock<mutex>& lck, Predicate pred);
void wait (unique_lock<mutex>& lck);
```

当前线程(应该已经锁定了lock的互斥锁)的执行被阻塞，直到得到通知。 在阻塞线程的时刻，该函数自动调用lock .unlock()，允许其他被锁定的线程继续。 一旦收到通知(由其他线程显式地通知)，该函数将解除阻塞并调用lock .lock()，使lck保持与调用该函数时相同的状态。然后函数返回(注意最后一个互斥锁可能会在返回之前再次阻塞线程)。       

                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             

MTTF: mean time to failure 

MTTR: mean time to repair 

MTBF: mean time between failure 

MTBF = MTTF + MTTR

![3421637145398_.pic.jpg](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/3421637145398_.pic.jpg)

![3431637145398_.pic.jpg](CSE%20mid%20%E6%8F%90%E7%BA%B2%203a885bd57ee64493b6a47bf326455d62/3431637145398_.pic.jpg)

$$
M T T F_{\text {supermodule }}=\frac{M T T F_{\text {replica }}}{N} \times \frac{M T T F_{\text {replica }}}{(N-1) \times M T T R_{\text {replica }}}=\frac{\left(M T T F_{\text {replica }}\right)^{2}}{(N*(N-1)) \times M T T R_{\text {replica }}}
$$