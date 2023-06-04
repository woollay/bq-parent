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

## 4. 微服务运行说明

### 4.1 微服务运行前，需要运行如下公共组件：

1. 进入zipkin目录并启动zipkin:

```shell
java -jar ./zipkin-server/target/zipkin-server-*exec.jar
```

2. 启动nacos:

```shell
sh ~/opensource/nacos-2.2.1/distribution/target/nacos-server-2.2.1/nacos/bin/startup.sh -m standalone
```

3. 启动本地redis:

```shell
redis-server
```

### 4.2 各微服务的运行参见其详细说明：

1. `bq-service-auth`初始化指导：

- 把${bq-service-auth}/${bq-service-auth}-startup/src/main/resources/sql下的sql全部执行一遍；
- 启动bq-service-auth服务，成功后执行如下curl命令：

```shell
curl --location 'http://localhost:9991/auth/user/add' \
--header 'Content-Type: application/json' \
--data '{
    "app_id":"app001",
    "app_key":"hao123",
    "app_name":"bq-app",
    "expire_time":1676800613607
}'
```

- 查询数据库表`select * from sys_access;`，会发现新增了用户数据：

```json
[
  {
    "id": "a4b42fa45e6f4dd8a942c34c62a6bf57",
    "app_name": "2a6fd05579481bfa4b08ebe9251a7f88eb376f1ea6fe",
    "app_id": "app001",
    "app_key": "4b649584514495a77a50f72495c7f31351b1b493cb40",
    "status": 1,
    "expire_time": 1715348560590,
    "create_time": 1683812560590,
    "sec_key": "3045022100b5b41861fb3b87fd6e0f1cfce423eae5db77672fe5cde4e63c11a7fe58d2bc52022040d8e0de733b10baa64cb843071e577c1a998cb60e84b844f0e53688ddf4adb1"
  }
]
```

- 执行curl命令：

```shell
curl --location --request POST 'http://127.0.0.1:9991/oauth/token?scope=read&grant_type=client_credentials' \
--header 'Authorization: Basic YXBwMDAxOmhhbzEyMw==' \
```

- 生成JwtToken的响应结果：

```json
{
  "code": "100001",
  "msg": "通过",
  "data": {
    "access_token": "eyJraWQiOiI2ZTNjNmYzMWI2ODk0MjU0YWUwY2Q4ODdkZWFmMzMxOCIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiJhcHAwMDEiLCJpc3MiOiJodHRwOlwvXC8xMjcuMC4wLjE6OTk5MSIsInJlc291cmNlcyI6WyJcL2F1dGhcL3d4Il0sInNvdXJjZV90eXBlIjoiU0RLIiwidG9rZW5fdHlwZSI6IkJlYXJlciIsImF1ZCI6ImFwcDAwMSIsIm5iZiI6MTY4NTg0MTYzNSwic2NvcGUiOlsicmVhZCJdLCJqd3RfdHlwZSI6InRva2VuIiwiZXhwIjoxNjg1ODQzNDM1LCJpYXQiOjE2ODU4NDE2MzUsImp0aSI6ImEwMDQyYzg5NjcxZTQyMDU5OWIyNDgwOGVmM2VjZGNlIn0.gSQvnW0nBIcDsA8Rj1H1lZHw30Heh-o3Vah4iDRbc1xlvH7NIzeoWNKq4sO__ilfTb21_9O87uKcmc5W2PHQr1gNoQ3PL2EHPRkacg9NdmFb1v5AYkzoOkU9fOr_MwE9J7W0leS0epalN13xt7QddNGhU-cTEToWfNdN-g7FUrkp0tdgvJw1F7h3JGzECGmD_XYWw02gPxuWqeywyf2M83yAJ8sYd7BK_ycdBkhSPPuuFcU4dzPvJrNldjzuiG2PMFnH7nREPiymG6dO3xCV5zTFiQLluAYYfON04BNZXPaoxUOJ3EfUAe2_xpJn5NEZu9lJZW6bCTv6Vw9vQEgPYw",
    "token_type": "Bearer",
    "scope": "read",
    "refresh_token": "eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhcHAwMDEiLCJhdWQiOiJhcHAwMDEiLCJuYmYiOjE2ODU4NDE2MzUsInNjb3BlIjpbInJlYWQiXSwiand0X3R5cGUiOiJyZWZyZXNoIiwiaXNzIjoiaHR0cDpcL1wvMTI3LjAuMC4xOjk5OTEiLCJzb3VyY2VfdHlwZSI6IlNESyIsInRva2VuX3R5cGUiOiJCZWFyZXIiLCJleHAiOjE2ODU4NDUyMzUsImlhdCI6MTY4NTg0MTYzNSwianRpIjoiZGZjYTI1ZjFmOTBhNGYzNmFlZDU1YTE4NmI4NjEwOTgifQ.CgQqeYN6EBIdostIEFTAkFziuVhkyVd3m2cQ4jFHAGQLcAtwXLN6iwJK0aCXNUMAYuFutUaxY23VlRfarYMWtHzj1U6ciDAKA_MI7dJw9XX0L1VKQzRyWXb1DYPuVAKXyLIt8tpOFTeXPK6RNtR92QzY0dKkYAsMKpV9zSMrewXRlTSVIwbKOCbsa8in2KQ1VimRUK64NixZqX4QYEnlgzfixYyHTw6phUb9XiDLKIBZ8LKM8Tb37c3XYZS8vKpWfdACF6jYynUiodRrqJmo2_U6QAP_8q2wOYRxq7NEffr8zo5_8wUyjJPw1D7M8mjiUFUn9S_4-TML6L7NFLQmOw",
    "client_id": "app001",
    "jti": "a0042c89671e420599b24808ef3ecdce",
    "resources": [
      "/auth/wx"
    ],
    "expires_in": 1799
  },
  "cost": 0
}
```

> 如果代码编译不过，则需要更换alibaba maven代理为官方代理：`https://repo1.maven.org/maven2/` ，因为alibaba在2022年底之后就更改了同步策略，新的jar包不保证同步；

2. 正常运行`bq-service-biz`即可；

> 如果代码编译不过，则需要更换alibaba maven代理为官方代理：`https://repo1.maven.org/maven2/` ，因为alibaba在2022年底之后就更改了同步策略，新的jar包不保证同步；

3. `bq-service-gateway`初始化指导：

- 在启动服务前，需要在`nacos`中`配置列表`创建配置`bq-gateway-route-id`，并把`bq-service-gateway-startup`下的application-routes中注释掉的路由配置填入其中：
	- GROUP: `DEFAULT_GROUP`
	- 配置内容JSON:
     ```json
     [
       {
         "id": "auth-route",
         "uri": "lb://bq-auth/",
         "order": 1,
         "predicates": [
           {
             "name": "Path",
             "args": {
               "pattern1": "/oauth/**",
               "pattern2": "/auth/**",
               "pattern3": "/bq-auth/monitor/**"
             }
           }
         ],
         "filters": [
           {
             "name": "CircuitBreaker",
             "args": {
               "name": "circuitBreaker",
               "fallbackUri": "forward:/fallback"
             }
           }
         ]
       },
       {
         "id": "biz_route",
         "uri": "lb://bq-biz/",
         "order": 1,
         "predicates": [
           {
             "name": "Path",
             "args": {
               "pattern1": "/bq-biz/demo/**",
               "pattern2": "/bq-biz/monitor/**",
               "pattern3": "/demo/**"
             }
           }
         ]
       },
       {
         "id": "fallback_route",
         "uri": "http://localhost:9992",
         "order": 1,
         "predicates": [
           {
             "name": "Path",
             "args": {
               "pattern": "/fallback"
             }
           }
         ]
       },
       {
         "id": "test_route",
         "uri": "lb://bq-test/",
         "order": 1,
         "predicates": [
           {
             "name": "Path",
             "args": {
               "pattern1": "/test/**",
               "pattern2": "/bq-test/monitor/**"
             }
           }
         ]
       }
     ]
     ```
- 启动网关服务；
- 从网关`bq-service-gateway`处发起对`bq-service-auth`的JwtToken生成调用：

```shell
curl --location --request POST 'http://127.0.0.1:9992/oauth/token?scope=read&grant_type=client_credentials' \
--header 'bq-integrity: 966b52885179be9c5842074f66cdd6c6a79f67445c3a27329a401072cdc536ff' \
--header 'Authorization: Basic YXBwMDAxOmhhbzEyMw==' \
```

- 响应结果：

```json
{
  "code": "100001",
  "msg": "通过",
  "data": {
    "access_token": "eyJraWQiOiI2ZTNjNmYzMWI2ODk0MjU0YWUwY2Q4ODdkZWFmMzMxOCIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiJhcHAwMDEiLCJpc3MiOiJodHRwOlwvXC9sb2NhbGhvc3Q6OTk5MSIsInJlc291cmNlcyI6WyJcL2F1dGhcL3d4Il0sInNvdXJjZV90eXBlIjoiU0RLIiwidG9rZW5fdHlwZSI6IkJlYXJlciIsImF1ZCI6ImFwcDAwMSIsIm5iZiI6MTY4NTg5NjIwNSwic2NvcGUiOlsicmVhZCJdLCJqd3RfdHlwZSI6InRva2VuIiwiZXhwIjoxNjg1ODk4MDA1LCJpYXQiOjE2ODU4OTYyMDUsImp0aSI6Ijg2ZjY0OTM5YjE4MDRjMWNhNWJkNDc4YTIyMjBjZGQ5In0.B22Zbcnb50vy2PjSKnCb6UcHduDFEXRD5_ApxP6nYygaBZwyb3IOtKER-ZjKsbdBzO3ecDMROnrsbLGx5PcBki0Wq-yMOhD6dPyLgO5FuvYjbuFr8omst-UULk7Wb2tHMwYX82Ud1_KK3VTCKFZDlnV2zplpoQYo-di1CbAcqXfS65ecHObfuqSbmpIjNfxOfS4AcbvsmRNg-pGpj45eHotTJhkTVw17FXATT8CSJnJ9kP3uO0ZZTNfI69ZYqA79OOhfnCJLIBaabOBkur1kKuTc3IKBgY8fFTKNMNztpuN4spsMYo8snchw6ZPTIp5MjZFR18CaQA1JomKDZ0OzjA",
    "token_type": "Bearer",
    "scope": "read",
    "refresh_token": "eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJhcHAwMDEiLCJhdWQiOiJhcHAwMDEiLCJuYmYiOjE2ODU4OTYyMDUsInNjb3BlIjpbInJlYWQiXSwiand0X3R5cGUiOiJyZWZyZXNoIiwiaXNzIjoiaHR0cDpcL1wvbG9jYWxob3N0Ojk5OTEiLCJzb3VyY2VfdHlwZSI6IlNESyIsInRva2VuX3R5cGUiOiJCZWFyZXIiLCJleHAiOjE2ODU4OTk4MDUsImlhdCI6MTY4NTg5NjIwNSwianRpIjoiYTQ4NzVhMzAxMmRhNDFhODllOTkxZjU0OWEzNzc4OGQifQ.DjdPmv7Pm8FUAQ5AghBofJjhjWG86h7QG6pi8rFmcQpAubCWlPTUebVhlh4Yhi9Wglk5MnsaBL_rzhN9OBwe0Z3sPnLtJJ8XWJtZeZOZW3lCzuU-tiQLw0u6G40Wlo_Gu_U3pz3CnlG1w1-lXjSsnXl4aZxjHKsbEjmSouz3DAkH84KWGCEkOcK81VaT0IYcTWbvfMMuIStIIbg_R8C1JSty2aMIG8PTpk6VUZP7xg9Krd1D4iIljAwLpnq9prpj8swApm3jDIulS1PWMyafpzkbXp-rDRMmRvS4nfiF8ERm_FJaxMJgH2qbGevyHWVCIT2DgCDnwIqGZRCrfRSfrA",
    "client_id": "app001",
    "jti": "86f64939b1804c1ca5bd478a2220cdd9",
    "resources": [
      "/auth/wx"
    ],
    "expires_in": 1800
  },
  "cost": 0
}
```

> 由此，完成了网关`bq-service-gateway`到`bq-service-auth`的服务发现；

- 网关`bq-service-gateway`再根据jwt token发起对`bq-service-biz`服务的调用：

```shell
curl --location 'http://localhost:9992/demo/qr' \
--header 'Authorization: Bearer eyJraWQiOiI2ZTNjNmYzMWI2ODk0MjU0YWUwY2Q4ODdkZWFmMzMxOCIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiJhcHAwMDEiLCJpc3MiOiJodHRwOlwvXC9sb2NhbGhvc3Q6OTk5MSIsInJlc291cmNlcyI6WyJcL2F1dGhcL3d4Il0sInNvdXJjZV90eXBlIjoiU0RLIiwidG9rZW5fdHlwZSI6IkJlYXJlciIsImF1ZCI6ImFwcDAwMSIsIm5iZiI6MTY4NTg5NjIwNSwic2NvcGUiOlsicmVhZCJdLCJqd3RfdHlwZSI6InRva2VuIiwiZXhwIjoxNjg1ODk4MDA1LCJpYXQiOjE2ODU4OTYyMDUsImp0aSI6Ijg2ZjY0OTM5YjE4MDRjMWNhNWJkNDc4YTIyMjBjZGQ5In0.B22Zbcnb50vy2PjSKnCb6UcHduDFEXRD5_ApxP6nYygaBZwyb3IOtKER-ZjKsbdBzO3ecDMROnrsbLGx5PcBki0Wq-yMOhD6dPyLgO5FuvYjbuFr8omst-UULk7Wb2tHMwYX82Ud1_KK3VTCKFZDlnV2zplpoQYo-di1CbAcqXfS65ecHObfuqSbmpIjNfxOfS4AcbvsmRNg-pGpj45eHotTJhkTVw17FXATT8CSJnJ9kP3uO0ZZTNfI69ZYqA79OOhfnCJLIBaabOBkur1kKuTc3IKBgY8fFTKNMNztpuN4spsMYo8snchw6ZPTIp5MjZFR18CaQA1JomKDZ0OzjA' \
--header 'userAgent: ASas' \
--header 'bq-integrity: c999d31229e1b6a7b7b6329e0eff9b39d92389248c7cffa5d81ef910c34cfa82' \
--header 'Content-Type: application/json' \
--data '{
    "code":"test123"
}'
```

- 响应结果：

```json
{
  "code": "100001",
  "msg": "通过",
  "data": {
    "open_id": "6c241d49a18742f1a9c5715f828e17a2"
  },
  "cost": 0
}
```

> 由此，完成了网关`bq-service-gateway`到`bq-service-biz`的服务发现；