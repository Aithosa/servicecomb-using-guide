# Servicecomb Using Guide

> Things about learning and using ServiceComb

本项目存储一些 ServiceComb 的使用教程、示例以及文档等。

## 官方资料

项目地址：https://servicecomb.apache.org/

> 具体项目信息可以阅读官方项目文档

## ServiceComb 简介

ServiceComb 是华为基于内部多个大型 IT 系统实践提炼出来的一套微服务开发框架，在开发态基于最佳实践封装了一套微服务运行模型，这些能力对用户完全透明，可以通过配置引入功能和对其进行调整。在运维阶段充分考虑了微服务运维，提供了丰富的监控指标和动态治理能力。

ServiceComb 的这套能力可以作为一个单独的开发框架，在需要轻量级微服务解决方案的的场景中单独使用，也可以建立在 SpringCloud 上，与 SpringCloud 提供的其他组件一起工作，在重量级场景中和 SpringCloud 一起产生 “1+1 大于 2”的效果。

ServiceComb 项目托管在[Github](https://github.com/apache?q=servicecomb)上，其各子项目如下表所示：

| 项目名                                                                                | 项目简介                             | 编程语言   | 说明        |
|------------------------------------------------------------------------------------|----------------------------------|--------|-----------|
| [servicecomb-java-chassis](https://github.com/apache/servicecomb-java-chassis)     | Java微服务框架（SDK）                   | Java   ||
| [servicecomb-service-center](https://github.com/apache/servicecomb-service-center) | 服务中心（服务注册及发现）                    | Golang ||
| [servicecomb-pack](https://github.com/apache/servicecomb-pack)                     | 支持Saga/TCC等多协议的分布式事务方案           | Java   | 项目归档，停止维护 |
| [servicecomb-mesher](https://github.com/apache/servicecomb-Mesher)                 | 微服务网格                            | Golang ||
| [servicecomb-kie](https://github.com/apache/servicecomb-kie)                       | 微服务配置管理中心                        | Golang ||
| [servicecomb-toolkit](https://github.com/apache/servicecomb-toolkit)               | 基于契约的微服务开发工具                     | Java   ||
| [servicecomb-samples](https://github.com/apache/servicecomb-samples)               | 提供了微服务示例                         | Java   ||
| [servicecomb-fence](https://github.com/apache/servicecomb-fence)                   | ServiceComb Java-chassis安全认证解决方案 | Java   ||
| [servicecomb-docs](https://github.com/apache/servicecomb-docs)                     | ServiceComb用户手册                  | CSS    ||
| [servicecomb-website](https://github.com/apache/servicecomb-website)               | ServiceComb网站                    | HTML   ||
| [servicecomb-saga-actuator](https://github.com/apache/servicecomb-saga-actuator)   | 集中式Saga事务协调器 （归档）                | Java   | 项目归档，停止维护 |

## 开发工具

本地轻量化 ServiceComb 引擎: https://support.huaweicloud.com/devg-cse/cse_04_0046.html
