## 1. memtable

更新先写入到内存中的memtable，memtable容量固定。


memtable 内存存储的结构是一个支持并发的skiplist（读写锁）。

skiplist的每个结点的数据为一条key-value记录，key的结构是：

| key | seq | keyType |

|----------|---------|--------|





- seq：为56bit递增的版本号

- keyType：8bit标记，表明是值还是删除



## 2. SST文件的结构

level0的SST文件是对memtable的dump操作。

SST文件的结构如下：
|  file begin | 

|:----------|

| data block 1 |

| data block 2|

|...   |

| data block N |

| meta block 1 |

|...|

| meta block K |

| metaindex block|

|index block|

|footer|

|**file end**|



### 2.1 data block

包含多个key-value条目（数据部分）和一个block trailer

#### 数据部分

数据部分由KV entries和restarts组成

每个KV entry的存储:

| shared key len(varint)| not shared key len(varint) | value len(varint) |key|value|

|----------|---------|--------|--------|--------|

 

- shared key len： 表示与上一个entry共享的key的长度

- not shared key len：非共享的key的长度，与共享的加起来是key的真正长度



为了避免某处数据损坏，导致后面全部key都无法解析，

每隔若干个KV entry设置一个restart点，该处KV entry不与前面共享key。shared key len等于0。

在KV entries的后面保存restart的信息（restart的offset和restart点的个数）



#### tailer

- type： 一字节，标记block压缩类型

- crc：四字节，检验码

### 2.2 meta block

存储该SST文件的元信息

meta的类型有：

- filter meta block

布隆过滤器

- stat meta block

统计信息

### 2.3 metaindex

用来索引meta block，组织形式为KV，key是meta的名字，value为meta的位置（blockhandle）

blockhandle表示某个block在文件内的offset和block的大小size

例如过滤器的meta的key为`filter.<N>`，可以有多个过滤器

### 2.4 index

用来索引datablock, 组织形式也为KV，

key是每个datablock的最大key（>=该datablock的所有key，小于下一个datablock的）

value是datablock的blockhandle

### 2.5 footer

固定长度40字节，包括

- metaindex的blockhandle

- index 的blockhandle

- padding 

- magic // 8字节小端0xdb4775248b80fb57

## 3. Manifest

存储各个层次的SST文件的名称和对应的key范围，每次compact后会重新创建一个

也可以存储一些全局的meta信息。

## 4. CURRENT

保存当前最新的Manifest文件名
