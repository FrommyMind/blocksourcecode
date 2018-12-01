---
title: HBase Region Splitting and Merging
date: 2018-11-28 06:01:03
tags: HBase
---

源文：https://hortonworks.com/blog/apache-hbase-region-splitting-and-merging/

For this post, we take a technical deep-dive into one of the core areas of HBase. Specifically, we will look at how Apache HBase distributes load through regions, and manages region splitting. HBase stores rows of data in tables. Tables are split into chunks of rows called “regions”. Those regions are distributed across the cluster, hosted and made available to client processes by the RegionServer process. A region is a continuous range within the key space, meaning all rows in the table that sort between the region’s start key and end key are stored in the same region. Regions are non-overlapping, i.e. a single row key belongs to exactly one region at any point in time. A region is only served by a single region server at any point in time, which is how HBase guarantees strong consistency within a single row#. Together with the -ROOT- and .META. regions, a table’s regions effectively form a 3 level B-Tree for the purposes of locating a row within a table.

在这篇文章中，我们将深入到HBase的一块核心内容。尤其是我们将要看下Apache Hbase怎么通过regions做负载的，怎么管理region的切分。HBase将数据存储在表中。表被切分为数据块即regions。这些regions被分配在整个集群，可以被客户端程序调用RegionServer来使用。一个region就是一个连续的key空间，意味着在一个表中，所有在region的开始key和结束key之间的数据都被存储在一个region里。Regions是不重叠的，例如，在任何一个时间点上一个row key都只属于一个特定的region，一个region只会存在一个region server上。这样HBase就能保证数据的强一致性。算上-ROOT-和.META这2个region，一个表的region可以有效地通过一个3层B-树获取一行数据在一个表中的位置。
<!-- more -->
A Region in turn, consists of many “Stores”, which correspond to column families. A store contains one memstore and zero or more store files. The data for each column family is stored and accessed separately.
一个Region反过来，由许多列簇的存储组成。一个列簇的存储包含一个内存存储和零到多个存储文件。数据的每一个列簇都被分开存储和使用。

A table typically consists of many regions, which are in turn hosted by many region servers. Thus, regions are the physical mechanism used to distribute the write and query load across region servers. When a table is first created, HBase, by default, will allocate only one region for the table. This means that initially, all requests will go to a single region server, regardless of the number of region servers. This is the primary reason why initial phases of loading data into an empty table cannot utilize the whole capacity of the cluster.
一个表通常由多个region组成，这些region分布在多个region server上。因此，region被分布式的region server在物理上进行读写和查询等操作。当一个表被初始创建时，HBase默认会为其分配一个region。这就意味着，在开始时所有的请求都会发送到一个单独的region servers上，无论还有多少其他的region server。这就在初始加载数据进入一个空表时，不能利用整个集群的性能的主要原因。

## PRE-SPLITTING
---

The reason HBase creates only one region for the table is that it cannot possibly know how to create the split points within the row key space. Making such decisions is based highly on the distribution of the keys in your data. Rather than taking a guess and leaving you to deal with the consequences, HBase does provide you with tools to manage this from the client. With a process called pre-splitting, you can create a table with many regions by supplying the split points at the table creation time. Since pre-splitting will ensure that the initial load is more evenly distributed throughout the cluster, you should always consider using it if you know your key distribution beforehand. However, pre-splitting also has a risk of creating regions, that do not truly distribute the load evenly because of data skew, or in the presence of very hot or large rows. If the initial set of region split points is chosen poorly, you may end up with heterogeneous load distribution, which will in turn limit your clusters performance.
HBase在初始时只创建一个region的原因是它不知道怎么去创建row key空间的分割点。需要根据数据的分布才能做出这个决定。HBase提供了一个工具帮助你从客户端管理它，而不是随便猜测一个值或者留给你去处理。通过一个叫做pre-splitting的程序，一个可以在创建表的时候通过提供切割点来创建多个region。因为pre-splitting将确保在初始加载数据时会更均匀的分布在整个集群。所以如果你只要你的key的分布情况，就应该尽量的考虑使用它。然而，pre-splitting在创建表时也有一个风险，就是由于数据倾斜或者热冷数据或者某些行的数据非常大使它不能完全的平均分布在集群上。如果初始的region切割点设置的不好，可能会产生各种各样的加载，这可能会影响你集群的性能。

There is no short answer for the optimal number of regions for a given load, but you can start with a lower multiple of the number of region servers as number of splits, then let automated splitting take care of the rest.
对于一个load任务使用多少region适合并没有一个很直观的答案，但是你可以在开始时设置较为region server的几倍数，然后随着数据的增加让它自动切分。

One issue with pre-splitting is calculating the split points for the table. You can use the RegionSplitter utility. RegionSplitter creates the split points, by using a pluggable SplitAlgorithm. HexStringSplit and UniformSplit are two predefined algorithms. The former can be used if the row keys have a prefix for hexadecimal strings (like if you are using hashes as prefixes). The latter divides up the key space evenly assuming they are random byte arrays. You can also implement your custom SplitAlgorithm and use it from the RegionSplitter utility.

pre-splitting的一个问题就是计算表的切割点。你可以使用RegionSplitter工具。RegionSplitter使用一个可插拔的算法来创建切割点。HexStringSplit和UniformSplit是2个预定义的算法。当row key有一个16进制字符串的前缀时，可以使用HexStringSplit算法。UniformSplit算法假设key是随机的字节数组，然后去切割key空间。你也可以自己写Split算法并通过RegionSplitter工具来使用它。

```
$ hbase org.apache.hadoop.hbase.util.RegionSplitter test_table HexStringSplit -c 10 -f f1
```
-c 10,指定需要的region是10个，-f 指定列簇, 分隔符是“:”. 这个工具将会创建一个有10个region的名为 “test_table”的表。
```
13/01/18 18:49:32 DEBUG hbase.HRegionInfo: Current INFO from scan results = {NAME => 'test_table,,1358563771069.acc1ad1b7962564fc3a43e5907e8db33.', STARTKEY => '', ENDKEY => '19999999', ENCODED => acc1ad1b7962564fc3a43e5907e8db33,}
13/01/18 18:49:32 DEBUG hbase.HRegionInfo: Current INFO from scan results = {NAME => 'test_table,19999999,1358563771096.37ec12df6bd0078f5573565af415c91b.', STARTKEY => '19999999', ENDKEY => '33333332', ENCODED => 37ec12df6bd0078f5573565af415c91b,}
...
```
If you have split points at hand, you can also use the HBase shell, to create the table with the desired split points.
如果你知道切割点，你也可以使用HBase shell在创建表的时候指定切割点。
```
hbase(main):015:0> create 'test_table', 'f1', SPLITS=> ['a', 'b', 'c']
```
or
```
$ echo -e  "anbnc" >/tmp/splits
hbase(main):015:0> create 'test_table', 'f1', SPLITSFILE=>'/tmp/splits'
```

For optimum load distribution, you should think about your data model, and key distribution for choosing the correct split algorithm or split points. Regardless of the method you chose to create the table with pre determined number of regions, you can now start loading the data into the table, and see that the load is distributed throughout your cluster. You can let automated splitting take over once data ingest starts, and continuously monitor the total number of regions for the table.
为了更好的分布加载数据，你应该思考下你的数据模型和key的分布情况，然后选择合适的分割算法或者分割点。无论你选择哪种方法创建你的表，你可以开始导入数据进入你的表，并查看加载是否分布在你的集群上。当你已经开始导入数据了，你也可以让它自动切割，然后监控表的region的个数。

## AUTO SPLITTING
---
Regardless of whether pre-splitting is used or not, once a region gets to a certain limit, it is automatically split into two regions. If you are using HBase 0.94 (which comes with HDP-1.2), you can configure when HBase decides to split a region, and how it calculates the split points via the pluggable RegionSplitPolicy API. There are a couple predefined region split policies: ConstantSizeRegionSplitPolicy, IncreasingToUpperBoundRegionSplitPolicy, and KeyPrefixRegionSplitPolicy.
无论是否使用pre-splitting，当一个region达到一定的限制，它会自动的分割成2个region。如果使用HBase0.94 （HDP-1.2），你可以配置什么时候HBase决定去切割一个region，和它怎么通过可插拔[RegionSplitPolicy API](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/regionserver/RegionSplitPolicy.html)来计算切割点。已经有一些预定义的切割策略：[ConstantSizeRegionSplitPolicy](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/regionserver/ConstantSizeRegionSplitPolicy.html)，[IncreasingToUpperBoundRegionSplitPolicy](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/regionserver/IncreasingToUpperBoundRegionSplitPolicy.html)和[KeyPrefixRegionSplitPilicy](http://hbase.apache.org/apidocs/org/apache/hadoop/hbase/regionserver/KeyPrefixRegionSplitPolicy.html)。

The first one is the default and only split policy for HBase versions before 0.94. It splits the regions when the total data size for one of the stores (corresponding to a column-family) in the region gets bigger than configured “hbase.hregion.max.filesize”, which has a default value of 10GB. This split policy is ideal in cases, where you are have done pre-splitting, and are interested in getting lower number of regions per region server.
ConstantSizeRegionSplitPolicy是默认的切割策略，并且是HBase0.94之前的唯一策略。当一个region的一个store存储的数据大小大于配置的“hbase.hregion.max.filesize”值时，默认10GB，它就会自动的切割分区。这个切割策略是理想的，你已经预分区你的表，并且每个region server不会有太多的region。

The default split policy for HBase 0.94 and trunk is IncreasingToUpperBoundRegionSplitPolicy, which does more aggressive splitting based on the number of regions hosted in the same region server. The split policy uses the max store file size based on Min (R^2 * “hbase.hregion.memstore.flush.size”, “hbase.hregion.max.filesize”), where R is the number of regions of the same table hosted on the same regionserver. So for example, with the default memstore flush size of 128MB and the default max store size of 10GB, the first region on the region server will be split just after the first flush at 128MB. As number of regions hosted in the region server increases, it will use increasing split sizes: 512MB, 1152MB, 2GB, 3.2GB, 4.6GB, 6.2GB, etc. After reaching 9 regions, the split size will go beyond the configured “hbase.hregion.max.filesize”, at which point, 10GB split size will be used from then on. For both of these algorithms, regardless of when splitting occurs, the split point used is the rowkey that corresponds to the mid point in the “block index” for the largest store file in the largest store.
HBase0.94默认的切割策略变成了IncreasingToUpperBoundRegionSplitPolicy，它会基于每个region server拥有的region个数对数据进行切割。这个切割策略使用最大store文件基于Min (R^2 * “hbase.hregion.memstore.flush.size”, “hbase.hregion.max.filesize”)，R是在一个region server上一个表拥有的region个数。例如，默认的内存存储是128MB，默认的最大store大小是10GB，只有在第一个flush达到128MB时，第一个region才会被切割。随着在region server上的region个数的增长，它将会增加分割大小：512MB, 1152MB, 2GB, 3.2GB, 4.6GB, 6.2GB,等待。当达到9个region时，分割大小将会超过参数 “hbase.hregion.max.filesize”，此时，10GB的切割大小将会被开始使用。对于所有的这些算法，无论何时发生拆分，所使用的拆分点都是对应于“块索引”中最大存储文件的中点的rowkey。

KeyPrefixRegionSplitPolicy is a curious addition to the HBase arsenal. You can configure the length of the prefix for your row keys for grouping them, and this split policy ensures that the regions are not split in the middle of a group of rows having the same prefix. If you have set prefixes for your keys, then you can use this split policy to ensure that rows having the same rowkey prefix always end up in the same region. This grouping of records is sometimes referred to as “Entity Groups” or “Row Groups”. This is a key feature when considering use of the “local transactions” (alternative link) feature in your application design.
KeyPrefixRegionSplitPolicy是一个很好的补充。你可以配置你用来分组的row key的前缀的长度，这个分割策略可以确保有相同的前缀的一组数据不会被从中间切割。如果你设置了你的key的前缀，你可以使用这个切割策略来确保你所有有相同前缀的数据不会被分割到不同的region中。这个分组是指实例组或者行组。当在你的应用设计考虑使用本地处理时这是一个关键的功能。

You can configure the default split policy to be used by setting the configuration “hbase.regionserver.region.split.policy”, or by configuring the table descriptor. For you brave souls, you can also implement your own custom split policy, and plug that in at table creation time, or by modifying an existing table:
你可以使用配置参数“hbase.regionserver.region.split.policy”来配置你的默认切割策略，或者通过配置表的描述信息。你也可以使用你自己的切割策略，并且在你创建表的时候使用它，或者修改已经存在的表。
```
HTableDescriptor tableDesc = new HTableDescriptor("example-table");
tableDesc.setValue(HTableDescriptor.SPLIT_POLICY, AwesomeSplitPolicy.class.getName());
//add columns etc
admin.createTable(tableDesc);
```

If you are doing pre-splitting, and want to manually manage region splits, you can also disable region splits, by setting “hbase.hregion.max.filesize” to a high number and setting the split policy to ConstantSizeRegionSplitPolicy. However, you should use a safeguard value of like 100GB, so that regions does not grow beyond a region server’s capabilities. You can consider disabling automated splitting and rely on the initial set of regions from pre-splitting for example, if you are using uniform hashes for your key prefixes, and you can ensure that the read/write load to each region as well as its size is uniform across the regions in the table.
如果你正在做pre-splitting，并且向要手动的管理你的region切割，你可以通过设置“hbase.hregion.max.filesize”一个比较大的值和设置切割策略为ConstantSizeRegionSplitPolicy来禁用它。然后，你应该使用一个比较安全的值如100GB，这样region不会增长到超过一个region server的容量。你可以考虑禁用自动切割和通过pre-splitting的初始设置，例如，如果你对你的key的前缀使用统一的hash，你可以确保对每个区域的读/写负载以及表中各个region的大小都是统一的。

## FORCED SPLITS
---

HBase also enables clients to force split an online table from the client side. For example, the HBase shell can be used to split all regions of the table, or split a region, optionally by supplying a split point.
```
hbase(main):024:0> split 'b07d0034cbe72cb040ae9cf66300a10c', 'b'
0 row(s) in 0.1620 seconds
```

With careful monitoring of your HBase load distribution, if you see that some regions are getting uneven loads, you may consider manually splitting those regions to even-out the load and improve throughput. Another reason why you might want to do manual splits is when you see that the initial splits for the region turns out to be suboptimal, and you have disabled automated splits. That might happen for example, if the data distribution changes over time.
HBase也提供通过客户端去强制分割在线的表。例如，可以使用HBase shell去分割表的所有region或者一个region，也可以通过提供分割点去切割。
仔细监控你的HBase的加载分布，如果有些region变的不均匀，你需要考虑手分割这些region来提高图吞吐量。为什么你需要手动分割的另外一个原因是当你发现初始的region变的不标准或者你禁用了自分割。例如，你的数据分布随着时间一直在变化。

## HOW REGION SPLITS ARE IMPLEMENTED
---

As write requests are handled by the region server, they accumulate in an in-memory storage system called the “memstore”. Once the memstore fills, its content are written to disk as additional store files. This event is called a “memstore flush”. As store files accumulate, the RegionServer will “compact” them into combined, larger files. After each flush or compaction finishes, a region split request is enqueued if the RegionSplitPolicy decides that the region should be split into two. Since all data files in HBase are immutable, when a split happens, the newly created daughter regions will not rewrite all the data into new files. Instead, they will create  small sym-link like files, named Reference files, which point to either top or bottom part of the parent store file according to the split point. The reference file will be used just like a regular data file, but only half of the records. The region can only be split if there are no more references to the immutable data files of the parent region. Those reference files are cleaned gradually by compactions, so that the region will stop referring to its parents files, and can be split further.
在HBase中，写的需求是由region server来处理的，他们被存储在内存存储系统“memstore”中。当memstore文件满了，它开始将数据写入存储在磁盘的存储文件中。这个过程叫做“memstore flush”。RegionServer将会把这些存储文件合并成大文件。每次flush或者合并完成后，如果RegionSplitPolicy决定这个region应该被分割成2个region，就会触发一个分割的请求。因为在HBase中所有的数据文件都是不可变的，当需要一个分割时，对于一个region的2个子region，他们不是重新将所有的数据写入新的文件，而是会先创建小的类似软连接的叫做引用文件的文件。一个region只有在没有引用指向它的父文件时才可以被分割。这些引用文件会被compaction清除，这样它就不会执行他们的父文件，然后可以被分割。


Although splitting the region is a local decision made at the RegionServer, the split process itself must coordinate with many actors. The RegionServer notifies the Master before and after the split, updates the .META. table so that clients can discover the new daughter regions, and rearranges the directory structure and data files in HDFS. Split is a multi task process. To enable rollback in case of an error, the RegionServer keeps an in-memory journal about the execution state. The steps taken by the RegionServer to execute the split are illustrated by Figure 1. Each step is labeled with its step number. Actions from RegionServers or Master are shown in red, while actions from the clients are show in green.
尽管region分割是被在RegionServer本地处理的问题，但是分割过程要考虑很多动作。RegionServer在分割的前后会通知Master去更新.META.表，这样客户端可以发现新的子region和文件夹结构和HDFS的数据文件。分割是有多个任务的程序。为了再错误时能够回滚，RegionServer将执行的状态保存在内存中。RegionServer执行分割的过程如下图。每一步都标注了号码。RegionServer的操作或者Master操作用红色标注，客户端的操作是绿颜色。
![Alt text](/img/DF2947BB-B48F-4C4E-B27A-C259D57EDA86.jpg)

1. RegionServer decides locally to split the region, and prepares the split. As a first step, it creates a znode in zookeeper under /hbase/region-in-transition/region-name in SPLITTING state.
RegionServe决定在本地分割region，并且开始准备分割。第一步，它首先在zookeeper的/hbase/region-in-transition/region-name下创建一个SPLITTING状态的znode。
2. The Master learns about this znode, since it has a watcher for the parent region-in-transition znode.
因为Master会监控父级znode region-in-transition，所以Master将获取到这个znode。
3. RegionServer creates a sub-directory named “.splits” under the parent’s region directory in HDFS.
RegionServer在HDFS上的父region目录下创建一个名为“.splits”的子文件夹
4. RegionServer closes the parent region, forces a flush of the cache and marks the region as offline in its local data structures. At this point, client requests coming to the parent region will throw NotServingRegionException. The client will retry with some backoff.
RegionServer关闭父region，强制执行一个缓存的flush并且在它本地的数据结构中标志这个region为offline。此时，客户端发送到父region的请求将会抛出NotServingRegionException，客户端重试一些回退。
5. RegionServer create the region directories under .splits directory, for daughter regions A and B, and creates necessary data structures. Then it splits the store files, in the sense that it creates two Reference files per store file in the parent region. Those reference files will point to the parent regions files.
RegionServer在.splits文件夹下为子regionA和B创建region文件夹并创建需要的数据结构。然后开始分割存储文件，此时它会为每一个存储文件创建2个引用文件指向父region。这些引用文件将会指向父region的文件。
6. RegionServer creates the actual region directory in HDFS, and moves the reference files for each daughter.
RegionServe在HDFS上创建真正的region文件夹并移动引用文件到每一个子region下。
7. RegionServer sends a Put request to the .META. table, and sets the parent as offline in the .META. table and adds information about daughter regions. At this point, there won’t be individual entries in .META. for the daughters. Clients will see the parent region is split if they scan .META., but won’t know about the daughters until they appear in .META.. Also, if this Put to .META. succeeds, the parent will be effectively split. If the RegionServer fails before this RPC succeeds, Master and the next region server opening the region will clean dirty state about the region split. After the .META. update, though, the region split will be rolled-forward by Master.
RegionServer发送一个Put请求到.META.表，并在.META.表中将父region设置为offline，并添加子region的信息。客户端通过扫描.META.会发现父region正在做分割，但是并不知道子region知道在.META.表中有他们的信息。如果这个PUT请求成功了，父region会被分割。如果RegionServer在这个RPC成功之前失败了，Master和下一个访问region的region server将会清除region分割的垃圾状态。在更新.META.表后，Master会继续向前处理region的分割。
8. RegionServer opens daughters in parallel to accept writes.
RegionServer打开子region并接受向其中写入数据。
9. RegionServer adds the daughters A and B to .META. together with information that it hosts the regions. After this point, clients can discover the new regions, and issue requests to the new region. Clients cache the .META. entries locally, but when they make requests to the region server or .META., their caches will be invalidated, and they will learn about the new regions from .META..
RegionServer添加子region A和B和他们所在节点的信息到.META.。在这个操作完成后，客户端可以发现新的region，并发送请求到新的region。客户端本地缓存.META.，但是当他们发送请求到region server或者 .META.，他们的缓存将会验证，这时他们会更新他们的 .META.。
10. RegionServer updates znode /hbase/region-in-transition/region-name in zookeeper to state SPLIT, so that the master can learn about it. The balancer can freely re-assign the daughter regions to other region servers if it chooses so.
RegionServer在zookeeper中更新znode /hbase/region-in-transition/region-name的状态为SPLIT，这样master就获取到这个状态。balancer可以将子region重新分配给其他的region server。
11. After the split, meta and HDFS will still contain references to the parent region. Those references will be removed when compactions in daughter regions rewrite the data files. Garbage collection tasks in the master periodically checks whether the daughter regions still refer to parents files.  If not, the parent region will be removed.
在分割后，HDFS仍旧保留这执行父region的引用文件。这些引用在子region重写数据时将会别移除。Master垃圾回收任务也会检查子region是否还有引用文件执行父region，如果没有父region将会被移除。

## REGION MERGES
---

Unlike region splitting, HBase at this point does not provide usable tools for merging regions. Although there are HMerge, and Merge tools, they are not very suited for general usage. There currently is no support for online tables, and auto-merging functionality. However, with issues like OnlineMerge, Master initiated automatic region merges, ZK-based Read/Write locks for table operations, we are working to stabilize region splits and enable better support for region merges. Stay tuned!

CONCLUSION

As you can see, under-the-hood HBase does a lot of housekeeping to manage regions splits and do automated sharding through regions. However, HBase also provides the necessary tools around region management, so that you can manage the splitting process. You can also control precisely when and how region splits are happening via a RegionSplitPolicy.
The number of regions in a table, and how those regions are split are crucial factors in understanding, and tuning your HBase cluster load. If you can estimate your key distribution, you should create the table with pre-splitting to get the optimum initial load performance. You can start with a lower multiple of number of region servers as a starting point for initial number of regions, and let automated splitting take over. If you cannot correctly estimate the initial split points, it is better to just create the table with one region, and start some initial load with automated splitting, and use IncreasingToUpperBoundRegionSplitPolicy. However, keep in mind that, the total number of regions will stabilize over time, and the current set of region split points will be determined from the data that the table has received so far. You may want to monitor the load distribution across the regions at all times, and if the load distribution changes over time, use manual splitting, or set more aggressive region split sizes. Lastly, you can try out the upcoming online merge feature and contribute your use case.

