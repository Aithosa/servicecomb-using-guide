# Servicecomb Using Guide

> Things about learning and using ServiceComb

本项目存储一些 ServiceComb 的使用教程、示例、文档以及官方课程资料等。

## 官方资料

项目地址：https://servicecomb.apache.org/

> 具体项目信息可以阅读官方项目文档

## ServiceComb 简介

ServiceComb 是华为基于内部多个大型 IT 系统实践提炼出来的一套微服务开发框架，在开发态基于最佳实践封装了一套微服务运行模型，这些能力对用户完全透明，可以通过配置引入功能和对其进行调整。在运维阶段充分考虑了微服务运维，提供了丰富的监控指标和动态治理能力。

ServiceComb 的这套能力可以作为一个单独的开发框架，在需要轻量级微服务解决方案的的场景中单独使用，也可以建立在 SpringCloud 上，与 SpringCloud 提供的其他组件一起工作，在重量级场景中和 SpringCloud 一起产生 “1+1 大于 2”的效果。

ServiceComb 项目托管在[Github](https://github.com/apache?q=servicecomb)上，其各子项目如下表所示：

| 项目名                                                                             | 项目简介                                  | 编程语言 | 说明               |
| ---------------------------------------------------------------------------------- | ----------------------------------------- | -------- | ------------------ |
| [servicecomb-java-chassis](https://github.com/apache/servicecomb-java-chassis)     | Java 微服务框架（SDK）                    | Java     |                    |
| [servicecomb-service-center](https://github.com/apache/servicecomb-service-center) | 服务中心（服务注册及发现）                | Golang   |                    |
| [servicecomb-pack](https://github.com/apache/servicecomb-pack)                     | 支持 Saga/TCC 等多协议的分布式事务方案    | Java     | 项目归档，停止维护 |
| [servicecomb-mesher](https://github.com/apache/servicecomb-Mesher)                 | 微服务网格                                | Golang   |                    |
| [servicecomb-kie](https://github.com/apache/servicecomb-kie)                       | 微服务配置管理中心                        | Golang   |                    |
| [servicecomb-toolkit](https://github.com/apache/servicecomb-toolkit)               | 基于契约的微服务开发工具                  | Java     |                    |
| [servicecomb-samples](https://github.com/apache/servicecomb-samples)               | 提供了微服务示例                          | Java     |                    |
| [servicecomb-fence](https://github.com/apache/servicecomb-fence)                   | ServiceComb Java-chassis 安全认证解决方案 | Java     |                    |
| [servicecomb-docs](https://github.com/apache/servicecomb-docs)                     | ServiceComb 用户手册                      | CSS      |                    |
| [servicecomb-website](https://github.com/apache/servicecomb-website)               | ServiceComb 网站                          | HTML     |                    |
| [servicecomb-saga-actuator](https://github.com/apache/servicecomb-saga-actuator)   | 集中式 Saga 事务协调器 （归档）           | Java     | 项目归档，停止维护 |

## 华为云对应产品

- [微服务引擎 CSE](https://www.huaweicloud.com/product/cse.html)
- [应用管理与运维平台 ServiceStage](https://www.huaweicloud.com/product/servicestage.html)

## 开发工具

本地轻量化 ServiceComb 引擎: https://support.huaweicloud.com/devg-cse/cse_04_0046.html

## 课程

[21 天转型微服务实战营](./courses/21-day-cse/)

课程笔记:

- [WEEK1 初步了解微服并教你搭建你的微服务](./courses/21-day-cse/notes/21-day-cse-notes-week1.md)
- [WEEK2 微服务 CSE 多种技术实战操作](./courses/21-day-cse/notes/21-day-cse-notes-week2.md)

## 学习建议

当前建议使用`3.x`版本，由于官方课程使用的版本比较老，一些组件的实现已经更改了，所以不再强烈推荐学习了，不过课程并不复杂，有兴趣的话可以尝试学习一下。

建议从[官方文档](https://servicecomb.apache.org/references/java-chassis/zh_CN/index.html)入手，先了解 Servicecomb 的设计和功能特性，
同时上手官方示例[`servicecomb-samples`](https://github.com/apache/servicecomb-samples)。通过亲自修改与调试熟悉这套框架。

> 注：官方文档有些欠缺维护，对于一些组件的描述不是特别详细，所以还需要对代码进行深入学习。建议多动手调试观察一下官方示例，了解其运行逻辑的同时阅读下相关逻辑的框架源码。
