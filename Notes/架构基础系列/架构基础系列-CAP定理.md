# 架构基础系列-CAP定理


CAP:
- Consistency（一致性）：数据一致性
    - 强一致性
    - 弱一致性
    - 最终一致性
- Availability（可用性）：服务可用性
- Partition Tolerance(分区容错性)：发生故障时，还能提供正常服务的能力

一般我们都是抱着AP，舍弃C，只保证最终一致性即可。
涉及金额变动的一般保证CA，舍弃P，发生故障的情况下宁可停止服务。


BASE：
- Basically Available（基本可用）：故障时保留部分核心可用，舍弃部分非核心可用
- soft state（软状态）：允许系统有中间状态，不影响系统的整体可用性
- eventual consistency（最终一致性）：数据经过一段时间，最终保持一致




降级
限流