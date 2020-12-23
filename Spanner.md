



# Spanner

---

## Introduction

 	Spanner is more like a multi-version database than a key-value store.

## Implementation

 1. Organization

     	1. <img src="/Users/parker/Library/Application Support/typora-user-images/image-20201207203006580.png" alt="image-20201207203006580" style="zoom: 150%;" />

 2. Software Stack

     1. ![image-20201208180853650](/Users/parker/Library/Application Support/typora-user-images/image-20201208180853650.png)

        Spanner is more like a multi-version database than a key-value store.

     2. directory

          1. <img src="/Users/parker/Library/Application Support/typora-user-images/image-20201207203024548.png" alt="image-20201207203024548" style="zoom:150%;" />

     3. ## Colossus

          1. Colossus是一个通过将元数据存储在一个将元数据存储在另一个将元数据存储在Google Chubby上的文件系统上的文件系统而获得高可扩展性的分布式文件系统。
           2. ![image-20201209112812433](/Users/parker/Library/Application Support/typora-user-images/image-20201209112812433.png)
           3. Big Table最大的特性之一是采用了LSM树固化所有修改，[LSM树](https://en.wikipedia.org/wiki/Log-structured_merge-tree)将一切修改转化为追加日志的形式，这意味着在这种数据结构中，随机写操作几乎不存在。而对于GFS/Colossus，顺序写操作仅在需要新创建数据块的时候联系Master/CFS，其余时候只需和Chunk Server/D Server做数据交换；同时，LSM树在上层的修改合并会让创建数据块的操作变得更加不频繁，从而进一步降低Big Table联系Master的频率。该特性导致的直接结果是，即使Master/CFS整体失去响应，其上的Big Table仍然能够正常读写相当长一段时间。根据前文提到的每层CFS对数据量的压缩率，越向下层的BigTable/CFS，访问频率越是以上万倍的速度衰减。因此，即使Chubby整体宕机，整个Colossus集群依然能运行极长时间，而这个时间差对于Google检测到Chubby失败而采取回复措施已经绰绰有余

 3. Directory

      1. Movedir的实现没有用单事务，这样就可以避免操作大数据块时阻塞同时发生的读写操作。Movedir的做法是，注册一个开始移动数据的事件，然后进行后台异步操作。当数据移动到只剩最后一小部分的时候，它会使用一个事务来做原子操作，同时移动那一小部分数据并更新这2个Paxos组的metadata。
      2. <img src="/Users/parker/Library/Application Support/typora-user-images/image-20201207203116053.png" alt="image-20201207203116053" style="zoom:150%;" />

 4. Data Model

     	1. interleave in 交错输入
          	2. <img src="/Users/parker/Library/Application Support/typora-user-images/image-20201207203614785.png" alt="image-20201207203614785" style="zoom:150%;" />

## True Time

