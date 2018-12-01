# TairTree
Tair分布式存储


![](https://i.imgur.com/LJOUEvd.png)


<pre>
Tair是淘宝的一个开源项目，它是一个分布式的key/value结构数据的解决方案。

作为一个分布式系统，Tair由一个中心控制节点（config server）和一些列的
服务节点（data server）组成。
     1）config server
        负责管理所有data server，并维护data server的状态信息；
        为了保证高可用,config server可通过heartbeat以一主一备形式提供服务。

     2）data server
        对外提供各种数据服务，并以心跳的形式将自身状况汇报给config server；
        所有的data server地位都是等价的。
</pre>

<pre>
Tair集群的基本概念
      
      configId:唯一标识一个tair集群，每个集群都有一个对应的configId，对应了集群
               的configserver地址和groupname，业务在初始化tair client的时候需要配置
               configId;

      namespace:又称area，是Tair中分配给应用的一个内存或者持久化存储区域，可以认为应用的
                数据存储在自己的namespace中，同一个集群(configId)中的namespace是唯一
                的。通过引入namespace，我们可以支持不同的应用在同集群中使用相同的key存放
                数据，也就是key相同，但内容不会冲突。一个namespace下如果存放相同的key，
                那么内容就会受到影响，在简单K/V形式下会被覆盖，rdb等待有数据结构的存储
                引擎内容会根据不同的接口发生不同的变化。

      quota配额：对应了每个namespace储存区的大小限制，超过配额后数据将面临最近最少使用的
                淘汰。持久化引擎（ldb）本身没有配额，ldb由于自带了mdb cache，所以也可以
                设置cache的配额，超过配额后，在内置的mdb内部进行淘汰。

      expireTime:数据的过期时间。当超过过期时间后，数据将对应用不可见，不同的存储引擎有
                 不同的策略清理掉过期的数据。
</pre>

![](https://i.imgur.com/TMHtlIM.png)

<pre>
存储引擎
      Tair存储引擎分为持久化存储引擎和非持久化存储引擎

      1）非持久化存储引擎
         可以看成是一个分布式缓存
      2）持久化存储引擎
         持久化的Tair将数据存放于磁盘，为了解决磁盘损坏导致数据丢失,Tair可以配置数据的备
         份数目。Tair自动将一份数据的不同备份存放到不同的主机上，当有主机发生异常，无法
         正常提供服务的时候，其余的备份会继续提供服务。

   Tair主要有下面三种存储引擎：
       1）mdb：定位于cache缓存，类似于memcache。支持K/V存取和prefix操作。
       2）rdb: 定位于cache缓存，采用了redis的内存存储结构，支持K/V,list, hash,set,
               sortedset等数据结构。
       3）ldb：定位于高性能存储，采用了levelDB作为引擎，并可选择内嵌mdb cache加速，这种
               情况下cache与持久化存储的数据一致性由Tair进行维护，支持K/V,prefix等数据
               结构，今后将支持list, hash, set, sortedset等Redis支持的数据结构。
</pre>