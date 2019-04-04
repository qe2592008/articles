# SpringCloud基础系列-Eureka
## 一、概述
### 1.1 简介
我们常说的Eureka其实是Spring Cloud Eureka，是Spring Cloud Netflix微服务套件中的服务注册发现组件，是对Netflix中的Eureka的二次封装，使其与Spring Boot微服务开发无缝融合。
> Netflix与Spring Cloud关系：
>
> Netflix是一家做视频的网站，可以这么说该网站上的美剧应该是最火的。
> 
> Netflix是一家没有CTO的公司，正是这样的组织架构能使产品与技术无缝的沟通，从而能快速迭代出更优秀的产品。在当时软件敏捷开发中，Netflix的更新速度不亚于当年的微信后台变更，虽然微信比Netflix迟发展，但是当年微信的灰度发布和敏捷开发应该算是业界最猛的。
>
> Netflix由于做视频的原因，访问量非常的大，从而促使其技术快速的发展在背后支撑着，也正是如此，Netflix开始把整体的系统往微服务上迁移。
>
> Netflix的微服务做的不是最早的，但是确是最大规模的在生产级别微服务的尝试。也正是这种大规模的生产级别尝试，在服务器运维上依托AWS云。当然AWS云同样受益于Netflix的大规模业务不断的壮大。
>
> Netflix的微服务大规模的应用，在技术上毫无保留的把一整套微服务架构核心技术栈开源了出来，叫做Netflix OSS，也正是如此，在技术上依靠开源社区的力量不断的壮大。
>
> Spring Cloud是构建微服务的核心，而Spring Cloud是基于Spring Boot来开发的。
> 
> Pivotal在Netflix开源的一整套核心技术产品线的同时，做了一系列的封装，就变成了Spring Cloud；虽然Spring Cloud到现在为止不只有Netflix提供的方案可以集成，还有很多方案，但Netflix是最成熟的。

### 1.2 结构
Eureka作为服务发现组件包括两个部件：
- Eureka服务器
- Eureka客户端。
> Eureka服务器：执行服务注册，执行服务下线，接收服务获取，接收服务续约，服务自动剔除（定时），发起服务注册（Server之间互相注册实现数据共享），自我保护

> Eureka客户端：发起服务注册，发起服务下线，发起服务续约心跳，发起服务调用（获取服务）

## 二、实例

## 三、原理

## 四、源码

## 五、配置

## 六、总结

