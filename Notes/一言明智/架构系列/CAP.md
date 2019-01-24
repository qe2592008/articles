CAP指的是Consistency（一致性）、Availability（可用性）、Partition tolerance（分区容错性）。

CAP理论指的是上面的三个特性中最多只能满足两者，也就是无法同时满足三大特性。

有三种情况：

    满足CA：单点集群，扩展性差，保证强一致性
    满足CP：性能不高，保证强一致性
    满足AP：只能保证最终一致性，能保证高可用性