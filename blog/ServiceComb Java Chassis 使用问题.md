# `ServiceComb Java Chassis` 使用问题

我最近在阅读文档和示例代码，尝试手动体验整体的功能和特性，但是中间出现了一些让我比较疑惑以及难受的地方，所以在这里记录一下，希望都能找到答案和解决方案。

其中最头疼的就是从零开始构建应用的时候根据文档和官方示例组装出来的代码没法正常运行，明明整个工程代码非常简洁，就是不知道哪里出问题了，问 ChatGPT 和 Claude 也没有靠谱的答案，这事让我难受了很久。

## 关于动态配置的问题

### 监听配置变化

`servicecomb-java-chassis`在`2.x`版本时用的是 archaius API 来管理配置，根据官方文档[在程序中读取配置信息](https://servicecomb.apache.org/references/java-chassis/2.x/zh_CN/config/read-config.html)，若要读取配置官方推荐使用 archaius 来读取动态配置：

```java
// 使用 DynamicPropertyFactory 监听配置变化
DynamicDoubleProperty myprop = DynamicPropertyFactory.getInstance()
  .getDoubleProperty("trace.handler.sampler.percent", 0.1);
 myprop.addCallback(new Runnable() {
      public void run() {
          // 当配置项的值变化时，该回调方法会被调用
          System.out.println("trace.handler.sampler.percent is changed!");
      }
  });
```

虽然使用的是`DynamicPropertyFactory`并且确实能读到项目中配置文件里的东西，不过经过测试只有通过配置中心设置的配置项才支持动态读取，只靠改文件内的配置并不能动态读取新值。

> 目前尚不清楚为何都使用了 archaius 依然不支持文件的动态配置读取

在`3.x`版本中，舍弃了 archaius API，而是全面采用 Spring 的`Environment`来管理配置，不过特性依然不变，根据官方文档[在程序中读取配置信息](https://servicecomb.apache.org/references/java-chassis/zh_CN/config/read-config.html)，推荐使用`DymamicProperties`监听配置变化：

```java
@RestSchema(schemaId = "ProviderController")
@RequestMapping(path = "/")
public class ProviderController {
  private DynamicProperties dynamicProperties;

  private String example;

  @Autowired
  public ProviderController(DynamicProperties dynamicProperties) {
    this.dynamicProperties = dynamicProperties;
    this.example = this.dynamicProperties.getStringProperty("basic.example",
            value -> this.example = value, "not set");
  }
}
```

经过测试只有通过配置中心设置的配置项才支持动态读取，只靠改文件内的配置并不能动态读取新值。

### 使用 Java Chassis 优先级配置

这里主要是使用`@InjectProperties`注解并声明为`Bean`来动态读取。

```java
@Component
  @InjectProperties(prefix = "jaxrstest.jaxrsclient")
  public class Configuration {
    /*
     * 方法的 prefix 属性值 "override" 会覆盖标注在类定义的 @InjectProperties
     * 注解的 prefix 属性值。
     *
     * keys属性可以为一个字符串数组，下标越小优先级越高。
     *
     * 这里会按照如下顺序的属性名称查找配置属性，直到找到已被配置的配置属性，则停止查找：
     * 1) jaxrstest.jaxrsclient.override.high
     * 2) jaxrstest.jaxrsclient.override.low
     *
     * 测试用例：
     * jaxrstest.jaxrsclient.override.high: hello high
     * jaxrstest.jaxrsclient.override.low: hello low
     * 预期：
     * hello high
     */
    @InjectProperty(prefix = "jaxrstest.jaxrsclient.override", keys = {"high", "low"})
    public String strValue;

    /**
     * keys支持通配符，并在可以在将配置属性注入的时候指定通配符的代入对象。
     *
     * 测试用例：
     * jaxrstest.jaxrsclient.k.value: 3
     * 预期：
     * 3
     */
    @InjectProperty(keys = "${key}.value")
    public int intValue;

    /**
     * 通配符的代入对象可以是一个字符串List，优先级遵循数组元素下标越小优先级越高策略。
     *
     * 测试用例：
     * jaxrstest.jaxrsclient.l1-1: 3.0
     * jaxrstest.jaxrsclient.l1-2: 2.0
     *
     * 预期：
     * 3.0
     */
    @InjectProperty(keys = "${full-list}")
    public float floatValue;

    /**
     * keys属性也支持多个通配符，优先级如下：首先通配符的优先级从左到右递减，
     * 然后如果通配符被代入List，遵循List中元素index越小优先级越高策略。
     *
     * 测试用例：
     * jaxrstest.jaxrsclient.low-1.a.high-1.b: 1
     * jaxrstest.jaxrsclient.low-1.a.high-2.b: 2
     * jaxrstest.jaxrsclient.low-2.a.high-1.b: 3
     * jaxrstest.jaxrsclient.low-2.a.high-2.b: 4
     * 预期：
     * 1
     */
    @InjectProperty(keys = "${low-list}.a.${high-list}.b")
    public long longValue;

    /**
     * 可以通过注解的defaultValue属性指定默认值。如果字段未关联任何配置属性，
     * 定义的默认值会生效，否则默认值会被覆盖。
     *
     * 测试用例：
     * 预期：
     * abc
     */
    @InjectProperty(defaultValue = "abc")
    public String strDef;
  }
```

此处的问题主要存在于占位符部分。

如果占位符是个普通字符串，比如`${key}`,那么在配置文件中（不管文件还是配置中心），加上配置`key=k`可以正常通过`jaxrstest.jaxrsclient.k.value`读取到配置值，并且只有在配置中心更改才能读到动态值。

> 然而`key=k`不支持动态读取，做出更改或者新增删除都不会影响到`jaxrstest.jaxrsclient.k.value`的值，除非重启。

这里另一个奇怪的地方是，都采用了占位符了，还必须要求`jaxrstest.jaxrsclient.k.value`这种必须带上`k`的形式，仿佛`${key}.value`不存在多此一举。

对于列表来说写在配置文件上就不会生效了，我看到这种特性测试代码上用法都是：

```java
ConfigWithAnnotation config = SCBEngine.getInstance().getPriorityPropertyManager()
  .createConfigObject(Configuration.class,
        "key", "k",
        "low-list", Arrays.asList("low-1", "low-2"),
        "high-list", Arrays.asList("high-1", "high-2"),
        "full-list", Arrays.asList("l1-1", "l1-2")
        );
```

必须写在代码里，同样的，在配置名上，必须指定的很具体，不能带占位符

```text
1.  root.low-1.a.high-1.b
2.  root.low-1.a.high-2.b
3.
3.  root.low-2.a.high-1.b
4.  root.low-2.a.high-2.b
```

## 关于项目骨架搭建

注意：当前并不推荐使用`2.8.x`版本开发新的微服务了，建议使用最新的`3.x`版本。

### 第一种：完全交由`ServiceComb`管理

#### `2.x`版本

参考官方文档：[开发第一个微服务](https://servicecomb.apache.org/references/java-chassis/2.x/zh_CN/start/first-sample.html)，里面的描述很简单，但是搭建好之后：

1. 可以连接服务中心，无法连接配置中心，

2. 服务中心能显示有实例，但是无法查看

3. 注册的信息有误，本地日志显示：

   ```log
   2025-02-11 22:01:25,692 [INFO][main][org.apache.servicecomb.core.SCBEngine:363] Service information is shown below:
   Service Center: [http://127.0.0.1:30100]
   Config Center: not exist # 1. 没连上配置中心
   Kie Center: not exist
   App ID: default # 2. App ID不对
   Service Name: defaultMicroservice # 3. 服务名不对
   Version: 1.0.0.0
   Environment:
   Endpoints: []
   Registration implementations:
     name:service center registration
       Service ID: a7b1a2ef36157edaa5eaee485b9320db793ae697
       Instance ID: 1048d0cd23f1483898275303c262aff6
   ```

#### 需要注意的问题

这种方法最大的特点是：

1. `pom.xml`文件表明完全交由`ServiceComb`管理，不直接引入 Spring Boot 组件

2. 启动类没有任何 Spring Boot 注解，main 函数中由`ServiceComb`初始化

   ```java
   public class CodeFirstProviderMain {
     public static void main(String[] args) {
       BeanUtils.init();
     }
   }
   ```

我看到 sample 示例中有类似方式的例子，不过按照文档写的可以启动成功并且连上服务中心，状态却不太对。

##### 启动成功但注册信息有误

参考`servicecomb-samples-2.x`的`codefirst-sample`示例，能够正常启动并连接注册中心，实例列表可以正常展示，契约可以正常读取。

但是显示的微服务信息都是默认的，没有遵从配置文件。而配置文件里的内容没什么问题。

用了默认信息就意味着启动了 provider 就没法再启动 consumer。

**需要知道是为什么没有读取到配置文件。**

###### 问题解决

目前找到了原因与解决方法。

关键启动日志：

```log
[2025-02-12 23:12:07,026][INFO][main][MicroserviceDefinition.java][]load microservice config, name=defaultMicroservice, paths=[jar:file:/D:/DevEnv/repository/gradle/caches/modules-2/files-2.1/org.apache.servicecomb/java-chassis-core/2.0.0/c15fff5e1afa40f794caff1c57389e99c8ee8202/java-chassis-core-2.0.0.jar!/microservice.yaml, file:/D:/OneDrive/Projects/Java/2-study/servicecomb/servicecomb-samples-2.x/java-chassis-samples/codefirst-sample/codefirst-provider/build/resources/main/microservice.yaml]
```

`ServiceComb` 在加载配置时，首先加载了 `java-chassis-core.jar` 包中的默认 `microservice.yaml`，然后才加载项目中的 `microservice.yaml`。这里出现了一个配置加载顺序的问题：

1. 首先加载了`java-chassis-core.jar` 中的默认配置，这个配置里面的服务名是 `defaultMicroservice`
2. 然后才加载项目中的 `microservice.yaml` 配置

看起来 `custom-handler-sample` 和 `codefirst-sample`的区别可能在于：

1. `custom-handler-sample` 成功覆盖了默认配置
2. `codefirst-sample` 没有成功覆盖默认配置

尝试在`microservice.yaml`中添加更高优先级的配置：

```yaml
# 方法一：指定加载顺序，最后加载（框架内部很多为了保证优先加载值是负的，这里使用正的）
servicecomb-config-order: 100
# 方法二：2.1.2之前版本配置
APPLICATION_ID: codefirsttest
service_description:
  name: codefirst
  version: 0.0.1
  properties:
    allowCrossApp: false
# 由于加载顺序原因，以下配置被覆盖
servicecomb:
  service:
    application: codefirsttest
    name: codefirst
    version: 0.0.1
```

> 官方文档：[微服务定义](https://servicecomb.apache.org/references/java-chassis/2.x/zh_CN/build-provider/definition/service-definition.html)

不知道为什么，`servicecomb-config-order`指定加载顺序并没有生效。

> 这个例子用的`servicecomb-java-chassis`版本是`2.0.0`，改为`2.6.0`就好了。
>
> 另外 gradle 配置没有包括`implementation group: 'org.apache.servicecomb', name: 'registry-service-center'`为什么之前可以连接上配置中心，改了版本号之后就开始报错了，是之前已经隐性包括了吗？

另一个值得深究的就是**为什么有的项目配置加载的晚，有的加载的早，理论上应该有固定顺序的**。

解决完 provider 的问题后，在启动 consumer，此时可以正常启动。

但是**注册的实例无法在 consumer 服务里查看，而且实例列表此时也会出问题，无法展示**。

猜测是因为这个实例出问题导致读取列表报错了。但是服务中心日志没报错。

再次回到根据官方文档建立的项目，还是运行错误，但是把配置文件名称从`application.yaml`改为`microservice.yaml`即可以正常工作，甚至配置中心都可以正常连接。

### 第二种：完全交由`servicecomb`管理-使用 Spring Boot 并开启`servicecomb`功能

#### `2.x`版本

参考示例见：`ServiceComb-SpringMVC`

`pom.xml`文件并没有什么变化，还是不直接引入任何 Spring Boot 组件。

启动类和平时写起来一样，只不过多了`@EnableServiceComb`注解，该注解由以下组件提供：

```xml
<dependency>
    <groupId>org.apache.servicecomb</groupId>
    <artifactId>java-chassis-spring-boot-starter-servlet</artifactId>
</dependency>
```

#### `3.x`版本

按照[官方文档](https://servicecomb.apache.org/references/java-chassis/zh_CN/start/first-sample.html)建议，`pom.xml`文件中添加如下依赖：

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.apache.servicecomb</groupId>
      <artifactId>java-chassis-dependencies</artifactId>
      <version>${java-chassis-dependencies.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>org.apache.servicecomb</groupId>
    <artifactId>solution-basic</artifactId>
  </dependency>
  <!-- using log4j2 -->
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j-impl</artifactId>
  </dependency>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
  </dependency>
  <dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
  </dependency>
  <!-- using service-center & kie -->
  <dependency>
    <groupId>org.apache.servicecomb</groupId>
    <artifactId>registry-service-center</artifactId>
  </dependency>
  <dependency>
    <groupId>org.apache.servicecomb</groupId>
    <artifactId>config-kie</artifactId>
  </dependency>
  <!-- using java chassis http transport -->
  <dependency>
    <groupId>org.apache.servicecomb</groupId>
    <artifactId>java-chassis-spring-boot-starter-standalone</artifactId>
  </dependency>
</dependencies>

<!-- 引入maven-compiler-plugin插件，使项目打包时保留方法参数名 -->
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>${maven-compiler-plugin.version}</version>
  <configuration>
    <compilerArgument>-parameters</compilerArgument>
    <source>17</source>
    <target>17</target>
  </configuration>
</plugin>
```

> `java-chassis-spring-boot-starter-standalone`只有 pom 文件并且内容都包含在了`solution-basic`中，可不必须引入。

## 一些用法解释

记录一下`servicecomb-java-chassis`示例和文档里一些涉及到的代码段和配置的详细解释，以尽可能消除使用上出错的可能性并且加深对框架的理解。

另外也会涉及一些 Spring 相关的知识。

### 不同组件的作用

`servicestage-environment`组件

### 关于启动类

完全由 ServiceComb 管理的时候启动类只有`BeanUtils.init()`，没有额外注解。

而遵照常规 Spring Boot 工程写的时候会遇到如下写法（目前主要出现在 gateway 类的服务上）：

```java
@SpringBootApplication
@EnableServiceComb // 3.x版本无需添加
public class GatewayApplication {
  public static void main(String[] args) throws Exception {
    try {
      new SpringApplicationBuilder()
              .web(WebApplicationType.NONE)
              .sources(GatewayApplication.class)
              .run(args);
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}
```

否则完全按照以下方式写即可：

```java
@SpringBootApplication
@EnableServiceComb
public class ConsumerApplication {
  public static void main(String[] args) throws Exception {
    try {
      SpringApplication.run(ConsumerApplication.class, args);
    } catch (Exception e) {
      e.printStackTrace();
    }
  }
}
```

这两者的启动方式具体有什么区别，尤其是`WebApplicationType`的枚举都代表着什么含义。

> 直接参考官方文档：[Java Chassis 与 Spring Boot 集成介绍](https://servicecomb.apache.org/references/java-chassis/zh_CN/spring-boot/introduction.html)

#### `WebApplicationType`的含义

```jav
/**
 * The application should not run as a web application and should not start an
 * embedded web server.
 * 应用程序不作为Web应用运行，也不启动嵌入式Web服务器
 */
NONE,

/**
 * The application should run as a servlet-based web application and should start an
 * embedded servlet web server.
 * 使用基于Servlet的Web应用程序，启动嵌入式Servlet Web服务器（如Tomcat）
 */
SERVLET,

/**
 * The application should run as a reactive web application and should start an
 * embedded reactive web server.
 * 使用响应式Web应用程序，启动嵌入式响应式Web服务器（如Netty）
 */
REACTIVE;
```

#### 两种启动方式的区别

第一种方式（Gateway 服务）：

- 使用`SpringApplicationBuilder`构建器模式
- 明确指定`WebApplicationType.NONE`，表示不启动内嵌的 Web 服务器
- 这种方式通常用于网关服务，因为 servicecomb 自己会管理服务器，不需要 Spring Boot 的内嵌 Web 服务器

第二种方式（普通消费者服务）：

- 使用简化的启动方式
- 不指定`WebApplicationType`，让 Spring Boot 自动检测和选择合适的 Web 环境
- 这种方式适用于普通的 Spring Boot 应用，会根据类路径下的依赖自动选择合适的 Web 环境

**为什么网关服务要使用`WebApplicationType.NONE`**

这行代码明确告诉 Spring Boot 不要初始化任何 Web 环境，因为该应用程序不需要处理 HTTP 请求或提供 Web 服务。

在 ServiceComb 的网关服务中使用`WebApplicationType.NONE`的原因是：

- ServiceComb 网关服务使用自己的网络层实现（可能基于`Vert.x`或其他技术）
- 避免启动不必要的 Spring Boot 内嵌 Web 服务器，防止端口冲突
- 减少资源消耗，因为不需要运行额外的 Web 服务器

如果你的服务有特殊的 Web 服务器需求，也可以使用第一种方式，但使用不同的`WebApplicationType`值。

> 如第一种使用`WebApplicationType.SERVLET`是否等价于常规的第二种写法？

#### 一些澄清

使用 `WebApplicationType.NONE` 时：

1. **依然可以用 `java -jar` 运行**
   - 完全可以正常运行，打包后的 jar 文件可以通过 `java -jar xxx.jar` 命令启动
2. **主要区别在于：**
   - ❌ 不会启动内置的 Web 服务器（如 Tomcat）
   - ❌ 不会开放 HTTP 端口（比如默认的 8080 端口）
   - ❌ 无法通过浏览器访问

举个例子：

1. **普通的 Spring Boot Web 应用**（不使用 `NONE`）：

```bash
# 启动后
> java -jar myapp.jar

# 可以通过浏览器访问
http://localhost:8080/hello
```

2. **使用 `WebApplicationType.NONE` 的应用**：

```bash
# 启动后
> java -jar myapp.jar

# 没有 Web 服务器
# 不能通过 http://localhost:8080 访问（Postman当然也不行）
```

常见使用场景：

1. **定时任务程序**

```java
@Scheduled(fixedRate = 5000)
public void doTask() {
    // 每5秒执行一次的后台任务
}
```

2. **消息队列处理程序**

```java
@KafkaListener(topics = "my-topic")
public void processMessage(String message) {
    // 处理接收到的消息
}
```

3. **批处理程序**

```java
@Scheduled(cron = "0 0 2 * * ?")
public void nightlyBatchJob() {
    // 每天凌晨2点执行的批处理任务
}
```

这些场景都不需要 Web 服务器，所以使用 `WebApplicationType.NONE` 可以减少资源占用，使应用程序更轻量级。

而使用 ServiceComb 框架时：

1. **独立的 HTTP 服务器**
   - ServiceComb 有自己的 HTTP 服务器实现
   - 不依赖于 Spring Boot 的内嵌 Web 服务器（如 Tomcat）
2. **配置说明**

```yaml
servicecomb:
  rest:
    address: 0.0.0.0:7777 # ServiceComb 自己的 REST 端点配置
```

所以可以通过`http://localhost:8080/hello`访问服务。

### Java Chassis 与 Spring Boot 集成

`3.x`版本，见官方文档[Java Chassis 与 Spring Boot 集成介绍](https://servicecomb.apache.org/references/java-chassis/zh_CN/spring-boot/introduction.html)。

分为**高性能模式**和**Web 模式**，其依赖分别为：

```xml
<!-- 高性能模式 -->
<dependency>
  <groupId>org.apache.servicecomb</groupId>
  <artifactId>java-chassis-spring-boot-starter-standalone</artifactId>
</dependency>
<!-- Web模式 -->
<dependency>
  <groupId>org.apache.servicecomb</groupId>
  <artifactId>java-chassis-spring-boot-starter-servlet</artifactId>
</dependency>
```

其具体 pom 差异为：

| 高性能模式（standalone）                             | Web 模式（servlet）                                  |                                                                      |
| ---------------------------------------------------- | ---------------------------------------------------- | -------------------------------------------------------------------- |
|                                                      | `org.springframework.boot:spring-boot-starter-web`   |                                                                      |
| `org.apache.servicecomb:transport-rest-vertx`        | `org.apache.servicecomb:transport-rest-servlet`      |                                                                      |
| `org.apache.servicecomb:provider-springmvc`          | `org.apache.servicecomb:provider-springmvc`          |                                                                      |
| `org.apache.servicecomb:provider-jaxrs`              | `org.apache.servicecomb:provider-jaxrs`              |                                                                      |
| `org.apache.servicecomb:provider-pojo`               | `org.apache.servicecomb:provider-pojo`               |                                                                      |
| `org.apache.servicecomb:handler-loadbalance`         | `org.apache.servicecomb:handler-loadbalance`         | ConditionEvaluationReportLoggingListener <br />need in autoconfigure |
| `org.hibernate.validator:hibernate-validator`        |                                                      |                                                                      |
|                                                      | `jakarta.servlet:jakarta.servlet-api`                |                                                                      |
| `org.apache.servicecomb:foundation-test-scaffolding` | `org.apache.servicecomb:foundation-test-scaffolding` | scope:test                                                           |
|                                                      |                                                      |                                                                      |

官方有的示例没有引入也能正常运行应该是因为已经引入了部分依赖。

### Maven 插件
