# SpringBoot基础系列-消息体系
## 概述
SpringBoot支持多种消息系统，包括JMS、AMQP、KAFKA、Spring WebSocket等。
其中，JMS是Java提供的消息服务，它本身并不是具体的消息系统，而是一类消息系统的抽象，其具体实现包括ActiveMQ、Artemis等。
AMQP是高级消息队列协议，同样是一类消息系统的抽象，其实现有RabbitMQ，
KAFKA是一个具体的消息系统，现属于Apache项目。
Spring WebSocket是Spring体系中的一员。
## JMS
   