Zookeeper是一个分布式的开放源码的应用程序协调服务。实现了CP，即一致性和分区容错性，即它提供了高的一致性，那么它的可用性就会比较低。

Zookeeper需要一个leader来领导，一旦它宕掉，那么就只能通过选举来选出一个leader来，这期间服务不可用。

leader的作用：

[5分钟让你了解 ZooKeeper 的功能和原理](https://blog.csdn.net/weijifeng_/article/details/79775738)