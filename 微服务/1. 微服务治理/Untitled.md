### 1. 微服务技术对比

|                | Dubbo               | SpringCloud              | SpringCloudAlibaba       |
| -------------- | ------------------- | ------------------------ | ------------------------ |
| 注册中心       | zookeeper, redis    | Eureka, Consul           | Nacos, Eureka            |
| 服务远程调用   | Dubbo协议           | Feign(http协议)          | Dubbo, Feign             |
| 配置中心       | 无                  | SpringCloudConfig        | SpringCloudConfig, Nacos |
| 服务网关       | 无                  | SpringCloudGateway, Zuul | SpringCloudGateway, Zuul |
| 服务监控和保护 | dubbo-admin, 功能弱 | Hystrix                  | Sentinel                 |

