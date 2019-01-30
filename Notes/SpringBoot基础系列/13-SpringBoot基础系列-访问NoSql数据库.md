# SpringBoot基础系列-访问NoSql数据库
## Redis
Redis是一个缓存、消息代理和功能丰富的键值存储，SpringBoot提供了针对Redis的两种lib包的自动配置支持：Lettuce和Jedis，Spring Data Redis提供了二者的抽象。

可以通过添加`spring-boot-starter-data-redis`依赖来提供对Redis的支持，默认使用的是Lettuce。

我们可以在使用的时候，将RedisConnectionFactory、StringRedisTemplate或者RedisTemplate注入当前类中来使用。

具体参见[SpringBoot整合Redis]()
## MongoDB
MongoDB是一个基于文件的存储系统，使用类似json的模式来存储。SpringBoot可以通过引入`spring-boot-starter-data-mongodb`依赖来使用mongodb。

我们使用的时候，需要在当前类中注入MongoDbFactory来使用MongoDb。

## Neo4j
Neo4j是一个Java开发的，开源的图形数据存储系统，使用`spring-boot-starter-data-neo4j`依赖引入。
## Gemfire

https://www.jianshu.com/p/ae2b23ca7535
## Solr

## Elasticsearch

## Cassandra

## Couchbase

## LDAP

## InfluxDB