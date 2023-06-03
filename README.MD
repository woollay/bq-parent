# 开源微服务代码框架

## 1. 开源微服务代码框架自述

1. 本人从事Java方面的工作已10+年了，经历了SSH/SpringMVC，终于等到了SpringBoot/SpringCloud，迎来了分布式微服务的崛起，盼来了云原生，这是最好的coding时代，也是最复杂的coding时代；
2. 本人最近几年一直在金融科技领域从事AI相关的算法商业化落地，能够深刻体会AI对各种行业的颠覆与创新，但是AI还远没有达到可以取代coding的程度；落后就要淘汰却是真实的；
3. 工作是最好的老师，在工作的过程中，逐渐总结出来了一套相对来说比较通用的架构设计思路、一套通用代码框架，该框架经过了团队的不断实践迭代，并逐步形成了一个不错的微服务解决方案。当然这主要是偏向特定领域的微服务解决方案，不可能通用于所有业务场景；本方案的特点是非常关注业务安全和业务的可扩展性；
4. 在给大家分享个人经验的同时，也是在自我梳理和总结，所谓温故而知新。与君共勉；

> 个人感悟
>> 1. 一路走来，Java语言世界已经发生了翻天覆地的变化，任我不眠不休，终究无法穷尽；
>> 2. 学以致用，我发现只有真正用过的，才会有真切的感悟，才能把框架里面的门道搞得清楚明白；
>> 3. 适合的，才是最好的；

## 2. 开源代码整体设计

```
                                                     +------------+
                                                     |   bq-log   |
                                                     |            |
                                                     +------------+
                                                    Based on SpringBoot
                                                            |
                                                            |
                                                            v
     +------------+           +------------+         +------------+         +-------------------+
     |bq-encryptor|  +----->  |   bq-base  | +-----> |bq-boot-root| +-----> | bq-service-gateway|
     |            |           |            |         |            |         |                   |
     +------------+           +------------+         +------------+         +-------------------+
  Based on BouncyCastle      Based on Spring       Based on SpringBoot    Based on SpringBoot-WebFlux
                                                            +
                                                            |
                                                            v
                                                     +------------+         +-------------------+
                                                     |bq-boot-base| +-----> | bq-service-auth   |
                                                     |            |     |   |                   |
                                                     +------------+     |   +-------------------+
                                                 ased on SpringBoot-Web | Based on SpringSecurity-Authorization-Server
                                                                        |
                                                                        |
                                                                        |
                                                                        |   +-------------------+
                                                                        +-> | bq-service-biz    |
                                                                            |                   |
                                                                            +-------------------+
```

> 说明：
> 1. `bq-encryptor`：基于`BouncyCastle`安全框架，已[开源](https://github.com/woollay/bq-encryptor) ，[加解密介绍](https://blog.csdn.net/dobuy/article/details/130799625)
     ，支持`RSA`/`AES`/`PGP`/`SM2`/`SM3`/`SM4`/`SHA-1`/`HMAC-SHA256`/`SHA-256`/`SHA-512`/`MD5`等常用加解密算法，并封装好了多种使用场景、做好了为SpringBoot所用的准备；
> 2. `bq-base`：基于Spring框架的基础代码框架，已[开源](https://github.com/woollay/bq-base) ，支持`json`/`redis`/`DataSource`/`guava`/`http`/`tcp`/`thread`/`jasypt`等常用工具API；
> 3. `bq-log`：基于SpringBoot框架的基础日志代码，已[开源](https://github.com/woollay/bq-log) ，支持接口Access日志、调用日志、业务操作日志等日志文件持久化，可根据实际情况扩展；
> 4. `bq-boot-root`：基于SpringBoot，已[开源](https://github.com/woollay/bq-boot-root) ，但是不包含`spring-boot-starter-web`，也不包含`spring-boot-starter-webflux`，可通用于`servlet`和`netty`web容器场景，封装了`redis`/`http
     `/`定时器`/`加密机`/`安全管理器`等的自动注入；
> 5. `bq-boot-base`：基于`spring-boot-starter-web`(servlet，BIO)，已[开源](https://github.com/woollay/bq-boot-base) ，提供常规的业务服务基础能力，支持`PostgreSQL`/`限流`/`bq-log`/`Web框架`/`业务数据加密机加密`等可配置自动注入；
> 6. `bq-service-gateway`：基于`spring-boot-starter-webflux`(Netty,NIO)，已[开源](https://github.com/woollay/bq-service-gateway) ，提供了Jwt Token安全校验能力，包括`接口完整性校验`/`接口数据加密`/Jwt Token合法性校验等；
> 7. `bq-service-auth`：基于`spring-security-oauth2-authorization-server`，已[开源](https://github.com/woollay/bq-service-auth) ，提供了JwtToken生成和刷新的能力；
> 8. `bq-service-biz`：业务微服务参考样例，已[开源](https://github.com/woollay/bq-service-biz) ；

## 3. 微服务逻辑架构设计

```
                           +-------------------+
                           |  Web/App Client   |
                           |                   |
                           +-------------------+
                                     |
                                     |
                                     v
  +--------------------------------------------------------------------+
  |                 |         Based On K8S                             |
  |                 |1                                                 |
  |                 v                                                  |
  |       +-------------------+    2      +-------------------+        |
  |       | bq-service-gateway| +-------> | bq-service-auth   |        |
  |       |                   |           |                   |        |
  |       +-------------------+           +-------------------+        |
  |                 |3                                                 |
  |                 +-------------------------------+                  |
  |                 v                               v                  |
  |       +-------------------+           +-------------------+        |
  |       | bq-service-biz1   |           | bq-service-biz2   |        |
  |       |                   |           |                   |        |
  |       +-------------------+           +-------------------+        |
  |                                                                    |
  +--------------------------------------------------------------------+
```

> 说明：
> 1. `bq-service-gateway`：基于`SpringCloud-Gateway`，用作JwtToken鉴权，并提供了接口、数据加解密的安全保障能力；
> 2. `bq-service-auth`：基于`spring-security-oauth2-authorization-server`，提供了JwtToken生成和刷新的能力；
> 3. `bq-service-biz`：基于`spring-boot-starter-web`，业务微服务参考样例；
> 4. `k8s`在上述微服务架构中，承担起了服务注册和服务发现的作用，鉴于`k8s`云原生环境构造较为复杂，实际开源的代码时，以`Nacos`(为主)/`Eureka`做服务注册和服务发现中间件；
> 5. 以上所有服务都以docker容器作为载体，确保服务有较好地集群迁移和弹性能力，并能够逐步平滑迁移至k8s的终极目标；
> 6. 逻辑架构不等同于物理架构(部署架构)，实际业务部署时，还有DMZ区和内网区，本逻辑架构做了简化处理；