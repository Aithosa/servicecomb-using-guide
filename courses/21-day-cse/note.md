# 21 天转型微服务实战营笔记

该笔记仅用于快速回顾课程教授的内容。

## 第 1 章 WEEK1 初步了解微服并教你搭建你的微服务

### 1.1 DAY1 微服务架构知识介绍

第一天主要介绍了微服务的基本概念和各个模块的特点，并且提到了容器，但是给出的链接 404 了。

而代码方面没有示例，只是介绍了如何在华为云开通 ServiceStage 与 CSE 服务。

### 1.2 DAY2 微服务入门之编写 HelloWorld

介绍了如何开发简单的微服务。但是所用的代码是很多年前的，虽然不影响运行，但是还是参考 ServiceComb 最新的代码比较好。不过开源的 ServiceComb 和华为云商业化的 CSE 不完全一样，华为云的示例代码版本也比较旧。

可以尝试在华为云真正开通服务给出模板代码，或者看下华为云产品文档。

这里比较重要的是给出的 maven setting 文件，可以使得从华为云的源下载依赖。
> 这个setting里的代码都很老旧，仅限于本课程使用，最新使用方式应改变了。
> 有些依赖包不包含文档，只有反编译的代码。

### 使用细节

创建一个简单的 `CSEJavaSDK` 微服务只需要引入 `cse-solution-service-engine` 包，但我们仍然推荐大家使用`<dependencyManagement>`管理依赖依赖，这在项目依赖关系复杂时可以有效降低依赖管理复杂度。

```xml
    <properties>
        <!-- 注意华为云上的最新版本 -->
        <cse.version>2.3.62</cse.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.huawei.paas.cse</groupId>
                <artifactId>cse-dependency</artifactId>
                <version>${cse.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>com.huawei.paas.cse</groupId>
            <artifactId>cse-solution-service-engine</artifactId>
        </dependency>
    </dependencies>
```
> 本课程的大部分代码引用仅限于此，应该这些就足够了。

修改配置以在本地连接 LocalCSE，如何具体开通华为云服务文档上的方法已不适用了，建议参考产品文档。

这里引入了契约的概念，但是契约内容目前都是自动生成和获取的，不用手动操作。

CSEJavaSDK 支持 JAX-RS、Spring MVC 和 RPC 三种开发风格，**一般推荐用户使用前两种**，配合 CSEJavaSDK 自动生成服务契约的能力开发更方便。具体参考 ServiceComb 文档。

本次示例代码的大致内容在后续课程中基本没变过。

### 1.3 DAY3 感知微服务和 CSE 的交互

本次课程讲了配置文件里各个配置项的作用，他们会连接华为云微服务引擎（Cloud Service Engine，CSE）的三个服务：

- 服务中心（service center，sc）
- 配置中心（config center，cc）
- monitor

### 1.4 DAY4 微服务实例的生命周期分析

本次课程讲了服务启动流程、服务发现、服务退出的流程。

代码部分涉及了监听 ServiceComb 框架启动事件，通过实现 `BootListener` 接口并实现`onBootEvent`方法。

注意有些 IDE 上会提供两种停止方式，以 IDEA 为例，上面的红色方形按钮是强制退出，不会触发优雅停机。下面的图标是正常退出进程，可以触发优雅停机。

运行环境是开发环境时（`service_description.environment=development`）可以自动重新注册契约而不报错。

这次课程文档中的小提示解答了一些疑问。

### 1.5 DAY5 教你如何配置你的微服务

本次课程讲了服务配置相关内容，包括：

- `microservice.yaml`配置文件
- 环境变量、`System property`
- 动态配置
- 通过 API 获取配置
- 日志配置

> **这里需要注意 CSE 是如何处理配置文件的，尤其是涉及 archaius 的配置。**

`microservice.yaml`配置文件

- 如果多个 jar 包下都有`microservice.yaml`文件，那么他们都会被加载。
- 磁盘目录下的`microservice.yaml`配置文件的优先级高于 jar 包内的配置文件，但是需要确认是哪里的目录。
- 可以通过在`microservice.yaml`文件内配置`servicecomb-config-order`来指定优先级。

> 配置文件和 jar 包在同一个目录就可以读取

环境变量、`System property`

微服务实例启动时也会从环境变量、系统属性中加载配置：

- Linux 系统的环境变量不允许有点号”.”，但 CSEJavaSDK 框架会自动将配置项 key 中的下划线映射为点号，因此我们可以将点转换为下划线来配置环境变量
- 环境变量的优先级高于配置文件，`System property`的优先级高于环境变量

动态配置

微服务实例连接配置中心后，可以从配置中心获取动态配置：

- 动态配置的优先级是最高的，并且可以在运行时刷新
- 服务治理所使用的诸多控制逻辑也是由配置项来控制的。实现服务动态治理的方式就是通过配置中心动态下发配置项

> **动态配置是如何到项目包的，是直接传输还是有配置文件在本地。**

通过 API 获取配置

CSEJavaSDK 使用统一的 API 来获取配置，用户使用配置的时候，不需要关心从环境变量或者配置中心来读取配置，框架已经自动为用户从各个配置来源读取配置，并根据优先级规则将所有配置进行了合并和覆盖。

优先级：动态配置 > system property > 环境变量 > 配置文件

> 这里的通过 API 获取配置就是指用 archaius 获取配置。
> 内容可能在 `foundation-config-2.1.0.jar` 和 `config-cc-2.1.0.jar`（ConfigCenter） 中

能读取`microservice.yaml`就意味着有 archaius 配置定义。

日志配置

- CSEJavaSDK 默认使用的日志框架是 Log4j，并且给出了一份默认的配置，在 `org.apache.servicecomb:foundation-common` 包的 `log4j.properties` 文件内。
- CSEJavaSDK 提供了 accesslog 功能，可以在传输方式为 REST over Vertx 的条件下使用，accesslog 默认也是基于 Log4j 打印的，配置文件在 `org.apache.servicecomb:transport-rest-vertx` 包的 `log4j.properties` 文件内。
- 如果要覆盖默认的日志配置，在项目的 `resources/config` 目录下配置一份 `log4j.properties` 文件即可。

在项目的`resources/config`目录下放置一份`log4j.properties`配置文件，覆盖默认的配置，令 accesslog 的内容合并到普通业务日志中输出。

> **日志打印的 traceId 是怎么实现的，servicecomb 就不支持默认展示。**

### 1.6 DAY6 CSE 实战之开发网关（含直播）

直播已不可用。

本次课程讲了网关的开发：

- 使用 `DefaultEdgeDispatcher` 开发网关服务
- 使用 `URLMappedEdgeDispatcher` 开发网关服务

网关服务不需要要更改代码，只需要增加网关服务的配置。

### 1.7 DAY7 CSE 实战之框架扩展机制

框架扩展机制：

- Handler 扩展机制
- Filter 扩展机制
- 异常转换扩展机制
- 请求处理流程简介

> 代码、xml、配置文件、META-INFO

## 第 2 章 WEEK2 微服务 CSE 多种技术实战操作

### 2.1 DAY8 CSE 实战之负载均衡

- 负载均衡策略
- 请求重试机制
- 实例隔离机制

这里代码用的是 Day-7，主要操作也不在代码上，而是 ServiceStage 页面上修改参数。这个在 LocalCSE 上是没有选项的。

### 2.2 DAY9 CSE 实战之服务治理（含直播）

- 限流策略
- 服务熔断
- 服务降级
- 灰度发布

限流、熔断需要在 ServiceStage 页面上修改参数，也可以设置服务参数。

降级通过设置服务参数实现。

灰度发布在 ServiceStage 页面上有对应选项，设置条件即可。

### 2.3 DAY10 CSE 实战之微服务线程模型和性能统计

- 线程模型简介
- 性能统计（Metrics）

线程模型的资料看 ServiceComb 文档。

原生的 CSEJavaSDK 框架开发的微服务默认工作于同步模式，传输方式为`Rest over Vertx`模式，基于`Vert.x`进行网络通信。

除了同步模式，CSEJavaSDK 也支持 Reactive 模式，该模式下所有的处理逻辑都运行在网络线程中。Reactive 模式的开发较为复杂，可以参考 ServiceComb 文档。

普通微服务默认都是工作于同步模式的，但是 EdgeService 网关服务默认是工作于 Reactive 模式的。因此，在 EdgeService 网关进行 handler 和 filter 扩展定制时需要注意不能阻塞线程。

判断一个微服务的工作模式是否是 Reactive 模式，最直接的办法是在本地以 Debug 模式启动该服务，在 Filter/Handler 扩展类或者业务代码里打断点，观察线程名。

### 2.7 DAY14 CSE 实战之其他服务何如接入 CSE

这里需要了解的主要是为为 SpringBoot 服务绑定 mesher，不过 PPT 上的 mesher 下载地址无法访问，在华为云上找不到其他可用文件。

Springboot 服务不包含任何 ServiceComb 和 CSE 依赖，主要依靠 mesher 来作为代理注册到 sc。当然 mesher 要进行配置，相当于一个服务实例一个 mesher。

## 第 3 章 WEEK3 微服务多种应用场景实战操作

这里主要就是华为云的使用方法了。

没有代码及 ServiceComb 级别需要注意的东西。
