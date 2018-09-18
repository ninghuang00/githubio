---
title: redis入门
categories:
  - 数据库
tags:
  - redis
  - 缓存
date: 2018-09-09 20:04:43
---
 这是摘要
 <!-- more -->

## redis应用场景
> 参考地址:http://blog.51cto.com/yw666/1910451

1. 负载均衡中,缓存session实现session共享
当使用负载均衡技术将用户请求分发到不同的服务器上时,就可能引起session一致性问题.
解决方案:
	1. 使用ip_hash技术将请求精确定位到确定的服务器,缺点就是服务器宕机,则会话丢失
	2. 会话共享复制:在服务器之间复制共享会话,缺点就是只能在相同的中间件之间进行(比如tomcat-tomcat)
	3. 使用cacheDB存取session信息，应用服务器接受新请求将session信息保存在cacheDB中，当应用服务器发生故障时，调度器会遍历寻找可用节点，分发请求，当应用服务器发现session不在本机内存时，则去cacheDB中查找，如果找到则复制到本机，这样实现session共享和高可用。
{%asset_img redis实现会话缓存.png redis实现会话缓存%}	

## redis内存模型
> 参考地址:https://mp.weixin.qq.com/s?__biz=MzI4NTA1MDEwNg==&mid=2650768913&idx=1&sn=df5d7dbe8122b832d3b398f94989c61f&scene=21#wechat_redirect

下图是执行`set hello word`涉及的数据模型
{% asset_img 数据模型.png 数据模型%}
* dictEntry(24字节)
Redis是Key-Value数据库，因此对每个键值对都会有一个dictEntry，里面存储了指向Key和Value的指针；next指向下一个dictEntry，与本Key-Value无关。
* Key(即SDS len+free+9 字节)
图中右上角可见，Key（“hello”）并不是直接以字符串存储，而是存储在SDS结构中。
* SDS(简单动态字符串)
Redis没有直接使用C字符串(即以空字符‘\0’结尾的字符数组)作为默认的字符串表示，而是使用了SDS。SDS是简单动态字符串(Simple Dynamic String)的缩写。
结构如下:
```c 
struct sdshdr {
    int len; //记录buf中已经使用的长度
    int free; //记录buf中未使用的长度
    char buf[]; //buf数组中使用'\0'作为结尾,'\0'不计算在len和free中
};
```
* redisObject(16字节)
Value(“world”)既不是直接以字符串存储，也不是像Key一样直接存储在SDS中，而是存储在redisObject中。实际上，不论Value是5种类型的哪一种，都是通过RedisObject来存储的；而RedisObject中的type字段指明了Value对象的类型，ptr字段则指向对象所在的地址。不过可以看出，字符串对象虽然经过了RedisObject的包装，但仍然需要通过SDS存储。
RedisObject的定义:
```c 
typedef struct redisObject {
　　unsigned type:4; //表示对象的类型,目前包括REDIS_STRING等五种
　　unsigned encoding:4; //表示对象的内部编码
　　unsigned lru:REDIS_LRU_BITS; //记录的是对象最后一次被访问的时间
　　int refcount; //记录对象被引用的次数
　　void *ptr; //指向具体数据的指针
} robj;
```
* jemalloc
无论是DictEntry对象，还是RedisObject、SDS对象，都需要**内存分配器**（如jemalloc）分配内存进行存储。以DictEntry对象为例，有3个指针组成，在64位机器下占24个字节，jemalloc会为它分配32字节大小的内存单元。

### Redis的5种对象类型
1. 字符串(REDIS_STRING)
	1. 简介
	字符串是最基础的类型，因为所有的键都是字符串类型，且字符串之外的其他几种复杂类型的元素也是字符串。字符串长度不能超过512MB。
	2. 内部编码:
		1. int:
		8个字节的长整型。字符串值是整型时，这个值使用long整型表示。
		2. embstr:
		字符串小于等于39字节时使用,由于使用时只分配一次内存(所以redisObject和SDS连续),
		在修改值的时候就需要重新分配两者内存,因此实现为只读,在修改时直接转换成raw再修改,无论是否超过39字节
		3. raw:
		大于39字节的字符串,会分别为SDS和RedisObject分配内存
2. 哈希(REDIS_HASH)
	1. 简介
	哈希作为一种数据结构，不仅与字符串、列表、集合、有序结合并列，是Redis对外提供的5种对象类型的一种，也是Redis作为Key-Value数据库所使用的数据结构。为了说明的方便，在本文后面当使用“内层的哈希”时，代表的是Redis对外提供的5种对象类型的一种；使用“外层的哈希”代指Redis作为Key-Value数据库所使用的数据结构。
	2. 内部编码
	内层的哈希使用的内部编码可以是压缩列表（ziplist）和哈希表（hashtable）两种；Redis的外层的哈希则只使用了hashtable。
	使用压缩列表的条件和REDIS_LIST相同,不满足则使用哈希表
{% asset_img hashtable结构.png %}
3. 列表(REDIS_LIST)
	1. 简介
	列表（list）用来存储多个有序的字符串，每个字符串称为元素；一个列表可以存储2^32-1个元素。Redis中的列表支持两端插入和弹出，并可以获得指定位置（或范围）的元素，可以充当数组、队列、栈等。
	2. 内部编码
		1. 压缩列表(ziplist)
		使用条件
			1. 列表中元素个数小于512
			2. 所有元素字符串对象大小小于64字节
		只能从压缩列表转换成双端列表,不能反向.其中，单个字符串不能超过64字节，是为了便于统一分配每个节点的长度。这里的64字节是指字符串的长度，不包括SDS结构，因为压缩列表使用连续、定长内存块存储字符串，不需要SDS结构指明长度。	 
		2. 双端列表(linkedlist)
		压缩列表是Redis为了节约内存而开发的，是由一系列特殊编码的连续内存块（而不是像双端链表一样每个节点是指针）组成的顺序型数据结构；与双端链表相比，压缩列表可以节省内存空间，但是进行修改或增删操作时，复杂度较高，因此当节点数量较少时，可以使用压缩列表。但是节点数量多时，还是使用双端链表划算。压缩列表不仅用于实现列表，也用于实现哈希、有序列表，使用非常广泛。

4. 集合(REDIS_SET)
	1. 简介
	集合（set）与列表类似，都是用来保存多个字符串，但集合与列表有两点不同：集合中的元素是无序的，因此不能通过索引来操作元素；集合中的元素不能有重复。一个集合中最多可以存储2^32-1个元素，除了支持常规的增删改查，Redis还支持多个集合取交集、并集、差集。
	2. 内部编码
		1. 哈希表
		2. 整数集合
		整数集合适用于集合所有元素都是整数且集合元素数量较小的时候，与哈希表相比，整数集合的优势在于集中存储，节省空间；同时，虽然对于元素的操作复杂度也由O(n)变为了O(1)，但由于集合数量较少，因此操作的时间并没有明显劣势。
		只有集合中元素数量小于512且都为整数值时使用
		结构定义:
```c 
typedef struct intset{
    uint32_t encoding; //决定contents数组中的元素的类型
    uint32_t length; //元素个数
    int8_t contents[];
} intset;
```		
5. 有序集合(REDIS_ZSET)
	1. 简介
	有序集合与集合一样，元素都不能重复。但与集合不同的是，有序集合中的元素是有顺序的。与列表使用索引下标作为排序依据不同，有序集合为每个元素设置一个分数（score）作为排序依据。
	2. 内部编码
		1. 压缩列表
		集合中成员小于128且所有成员小于64字节时使用
		2. 跳表(skiplist)
		跳跃表是一种有序数据结构，通过在每个节点中维持多个指向其它节点的指针，从而达到快速访问节点的目的。除了跳跃表，实现有序数据结构的另一种典型实现是平衡树；大多数情况下，跳跃表的效率可以和平衡树媲美，且跳跃表实现比平衡树简单很多，因此Redis中选用跳跃表代替平衡树。跳跃表支持平均O(logN)、最坏O(N)的复杂点进行节点查找，并支持顺序操作。Redis的跳跃表实现由zskiplist和zskiplistNode两个结构组成：前者用于保存跳跃表信息（如头结点、尾节点、长度等），后者用于表示跳跃表节点。

## 估算redis内存使用量
> 参考地址:https://mp.weixin.qq.com/s?__biz=MzI4NTA1MDEwNg==&mid=2650768913&idx=1&sn=df5d7dbe8122b832d3b398f94989c61f&scene=21#wechat_redirect

假设有90000个键值对，每个key的长度是7个字节，每个value的长度也是7个字节（且key和value都不是整数）。
下面来估算这90000个键值对所占用的空间。
在估算占据空间之前，首先可以判定字符串类型使用的编码方式：embstr。90000个键值对占据的内存空间主要可以分为两部分：
**一部分是90000个dictEntry占据的空间；一部分是键值对所需要的bucket空间。**
每个dictEntry占据的空间包括：
* 一个dictEntry，24字节，jemalloc会分配32字节的内存块。
* 一个key，7字节，所以SDS(key)需要7+9=16个字节，jemalloc会分配16字节的内存块。
* 一个RedisObject，16字节，jemalloc会分配16字节的内存块。
* 一个value，7字节，所以SDS(value)需要7+9=16个字节，jemalloc会分配16字节的内存块。

综上，一个dictEntry需要32+16+16+16=80个字节。
bucket空间：bucket数组的大小为大于90000的最小的2^n，是131072，每个bucket元素为8字节（因为64位系统中指针大小为8字节）。
因此，可以估算出这90000个键值对占据的内存大小为：90000*80 + 131072*8 = 8248576。
验证代码:
```java
public class RedisTest {
　　public static Jedis jedis = new Jedis("localhost", 6379);
　　public static void main(String[] args) throws Exception{
　　　　Long m1 = Long.valueOf(getMemory());
　　　　insertData();
　　　　Long m2 = Long.valueOf(getMemory());
　　　　System.out.println(m2 - m1);
　　}
　　public static void insertData(){
　　　　for(int i = 10000; i < 100000; i++){
　　　　　　jedis.set("aa" + i, "aa" + i); //key和value长度都是7字节，且不是整数
　　　　}
　　}
　　public static String getMemory(){
　　　　String memoryAllLine = jedis.info("memory");
　　　　String usedMemoryLine = memoryAllLine.split("\r\n")[1];
　　　　String memory = usedMemoryLine.substring(usedMemoryLine.indexOf(':') + 1);
　　　　return memory;
　　}
}
```
作为对比将key和value的长度由7字节增加到8字节，则对应的SDS变为17个字节，jemalloc会分配32个字节，因此每个dictEntry占用的字节数也由80字节变为112字节。此时估算这90000个键值对占据内存大小为：90000*112 + 131072*8 = 11128576。
```java
public static void insertData(){

　　for(int i = 10000; i < 100000; i++){

　　　　jedis.set("aaa" + i, "aaa" + i); //key和value长度都是8字节，且不是整数

　　}

}
```

## redis高可用
> 参考地址:https://mp.weixin.qq.com/s?__biz=MzI4NTA1MDEwNg==&mid=2650769300&idx=1&sn=49a11efa1a6ee605fceaddf240a55c40&scene=21#wechat_redirect

### 持久化
最简单的高可用方案,将数据备份到硬盘,保证进程退出后数据不会丢失
1. RDB持久化(redis database)
手动或者自动将当前进程中的数据生成快照存到硬盘中
优缺点
	1. 优点:
		* RDB文件体积小,网络传输速度快,适合做全量复制
		* 恢复速度比AOF快很多
		* 对redis性能影响小
	2. 缺点:
		* 实时性低,兼容性差
2. AOF持久化(append only file)
将redis每次执行的写命令记录到单独的日志文件(类似MySQL的binlog),实时性更好,更流行
与RDB持久化相对应，AOF的优点在于支持秒级持久化、兼容性好，缺点是文件大、恢复速度慢、对性能影响大。

3. 持久化策略选择
在多数情况下，我们都会配置主从环境，slave的存在既可以实现数据的热备，也可以进行读写分离分担Redis读请求，以及在master宕掉后继续提供服务。在这种情况下，一种可行的做法是：
	* master：
	完全关闭持久化（包括RDB和AOF），这样可以让master的性能达到最好；
	* slave：
	关闭RDB，开启AOF（如果对数据安全要求不高，开启RDB关闭AOF也可以），并定时对持久化文件进行备份（如备份到其他文件夹，并标记好备份的时间）；然后关闭AOF的自动重写，然后添加定时任务，在每天Redis闲时（如凌晨12点）调用bgrewriteaof。

### 复制
复制主要实现了数据的多机备份以及对于读操作的负载均衡和简单的故障恢复。哨兵和集群都是在复制的基础上实现的高可用.
缺陷是:
* 故障恢复无法自动化
* 写操作无法负载均衡、存储能力受到单机的限制。

### 哨兵
在复制的基础上，哨兵实现了自动化的故障恢复。缺陷是写操作无法负载均衡、存储能力受到单机的限制。

### 集群
通过集群，Redis解决了写操作无法负载均衡以及存储能力受到单机限制的问题，实现了较为完善的高可用方案。

## redis主从同步
> 参考地址:https://juejin.im/entry/5b90a96a5188255c48348e15

### 主从同步的作用
* 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
* 故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复，但实际上是一种服务的冗余。
* 负载均衡：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
* 高可用基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

### 步骤
1. 启动两个redis实例,此时两个都默认是master节点
2. 选择其中之一作为slave节点执行`slaveof 192.168.0.230 6379`,想要断开复制只要在从节点上执行`slaveof no one`

### 原理
1. 建立连接阶段
	1. 保存主节点信息
	2. 建立socket连接
	3. 发送ping命令,根据返回的信息判断socket连接是否可用
	4. 身份验证
	5. 发送从节点信息,主节点保存
2. 数据同步阶段
	1. 从节点向主节点发送psync命令,开始同步.此时主从节点互为客户端,同步根据主从节点状态分为两种:
		1. 全量复制
			1. 从节点或者主节点判断出无法进行部分复制的时候,使用全量复制
			2. 主节点接收到全量复制命令后,执行bgsave,在后台生成RDB文件,并使用一个缓存区保存从现在开始的所有写命令
			3. 从节点接收到RDB文件后,将现有旧数据删除,载入RDB文件,将数据库恢复到主节点执行bgsave时的状态,
			4. 主节点将缓存区中的写命令发送给从节点执行,更新到最新状态
		2. 部分复制
			1. 复制偏移量
			主节点和从节点分别维护一个offset,用来表示主节点复制到从节点的数据量,通过对比offset可以得知主从节点中的数据是否一致,不一致的**数据存储位置**记录在复制积压缓冲区中.
			2. 复制积压缓冲区
			复制积压缓冲区是由主节点维护的一个默认大小1M的FIFO队列,备份主节点命令传播阶段的写命令,以及每个字节对应的复制偏移量
			3. 服务器运行ID
			`info server | grep run_id`节点初次复制时,主节点会将自己的run_id发给从节点,用于断线重连的时候判断之前是否有过连接,有过连接的可能可以使用部分复制,否则只能使用全量复制

3. 命令传播阶段
数据同步阶段完成后，主从节点进入命令传播阶段。在这个阶段主节点将自己执行的写命令发送给从节点，从节点接收命令并执行，从而保证主从节点数据的一致性.此时主从节点还要维持心跳机制：PING和REPLCONF ACK。心跳机制对于主从复制的超时判断、数据安全等有作用。

## SpringBoot集成redis
















