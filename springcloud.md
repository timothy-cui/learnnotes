# SpringCloud微服务实操笔记
***
## 相关知识点综述
   * 一，Spring Cloud Config : 用于微服务中管理应用程序的配置文件；
   * 二，Spring Eureka : 用于定位应用程序资源的物理位置；
   * 三，Spring Cloud 和 Netflix Hystrix : 实现客户端弹性模式，在远程服务发生错误或者表现不佳时保护远程资源的客户端免于崩溃；
   * 四，Spring Cloud 和 Zuul : 实现服务网关，充当服务客户端和被调用的服务之间的中介；
   * 五，Spring Cloud 和 Oauth2 : 用于保护微服务，实现令牌机制；
   * 六，Spring Cloud Stream : 用于实现事件驱动；
   * 七，Spring Cloud Sleuth 和 Zipkin : 用于关联微服务之间的调用，实现日志聚合，进行分布式跟踪。
***
## 微服务基本概念
* 什么是微服务？
   * 微服务通过将大型代码分解为小型的精确定义的部分，帮助解决大型代码库中传统的复杂问题。分解和分离应用程序的功能，使它们完全彼此独立；
* 微服务架构的特征：
   * 应用程序逻辑分解为具有明确定义了职责范围的细粒度组件，这些组件互相协调提供解决方案；
   * 每个组件都有一个小的职责领域，并且完全独立部署。
   * 微服务通信基于一些基本的原则，并采用http和json这样的轻量级通信协议，在服务消费者和服务提供者之间进行数据交换；
   * 构建在微服务之上的应用程序能够使用多种编程语言和技术进行构建；
   * 微服务利用其小、独立和分布式的性质，使组织拥有明确责任领域的小型开发团队。
* Spring和微服务的关系：
   * Spring的核心是建立在依赖注入的概念上的，Spring团队启动了Spring Boot和Spring Cloud两个项目来响应从单体应用程序模型到高度分布式模型的转变；
   * Spring Cloud框架使实施和部署微服务到私有云或公有云变得更加简单；
* 为什么要改变构建应用的方式？
   * 全球性的竞争意味着复杂性上升、客户期待更快速的交付、性能和可伸缩性、客户期望他们的应用程序可用的这些需求特性促使应用程序的构建需要具有灵活性、有弹性、可伸缩性这些特性。
* 云的概念：
   * 云计算的三种核心模式：基础设施即服务[IaaS];平台即服务[PaaS];软件即服务【SaaS】。
   * 新的云平台模式：函数即服务[FaaS];容器即服务[CaaS].
* 云和微服务：
   * 基于云的微服务的优势是以弹性的概念为中心，通过Docker将微服务和所需基础设施部署到基于IaaS的云供应商，以下是用于微服务的常见部署拓扑结构：
      * 简化的基础设施管理；
      * 大规模的水平可伸缩性；
      * 通过地理分布实现高冗余。
   * IaaS相较于PaaS而言，更灵活，可以跨多个云供应商进行移植。
* 构建微服务：
   * 架构师-分析业务问题、设计微服务架构【专注于业务问题的自然轮廓】；
   * 什么时候不应该使用微服务？
      * 考虑构建分布式系统的复杂性；
      * 考虑服务器散乱问题；
      * 考虑应用程序的类型；
      * 考虑数据事务和一致性。
   * 开发人员-使用springboot和java构建微服务【运用良好的设计原则】；
   * DevOps工程师-构建运行时的严谨性，关注如何打包、部署和监控【尽早建立服务的生命周期】。
***
## 项目基础依赖
* Spring Boot与Spring Cloud的依赖【具有版本关联，可从官网查询，否则会产生版本冲突】。
```
<properties>
    <spring-boot-dependencies.version>2.2.4.RELEASE</spring-boot-dependencies.version>
    <spring-cloud-dependencies.version>Hoxton.SR12</spring-cloud-dependencies.version>
</properties>

<dependencyManagement>
    <dependencies>
        <!-- spring boot 版本 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot-dependencies.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <!-- spring cloud 版本 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring-cloud-dependencies.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
* 本文实操github代码库：
***
## 一，Spring Cloud Config
* 基于云的微服务开发强调以下几点：
   * 应用程序的配置与正在部署的实际代码完全分离；
   * 构建服务器、应用程序以及一个不可变的镜像，它们在各个环境中进行提升时永远不会发生变化；
   * 在服务器启动时通过环境变量注入应用程序配置信息，或者在微服务启动时通过集中式存储库读取应用程序配置信息。
* 建立应用程序配置管理的四条原则：分离、抽象、集中、稳定。
* 概念架构
   * 当一个服务实例出现时，它将调用一个服务端点来读取其所在环境的特定配置信息。配置管理的连接信息将在微服务启动时被传递给微服务；
   * 实际的配置信息驻留在存储库中；
   * 应用程序配置数据的实际管理与应用程序的部署方式无关；
   * 进行配置管理更改时，必须通知使用该应用程序配置数据的服务，并刷新应用程序数据的副本。
* 方案选择：
   * Eureka，久经测试，用于服务发现和键值管理，分布式键值存储；
   * Spring Cloud Config，提供不同后端支持的通用配置管理解决方案，可以将git、Eureka等作为后端进行整合，非分布式键值存储【本文所选择实现的方案】；
   * 其他···。
* maven依赖-Spring Cloud Config[服务器]
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
* 引导类[服务器]
```
package com.thoughtmechanix.confsvr;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```
* Spring Cloud配置服务器的application.yml配置[服务器]
```
server:
   port: 8888
spring:
  profiles:
    active: native
  cloud:
     config:
       server:
           native:
              searchLocations: file:///Users/username/IdeaProjects/find_source_code/spring_cloud_config/src/main/resources/config/licensingservice,
                               file:///Users/username/IdeaProjects/find_source_code/spring_cloud_config/src/main/resources/config/organizationservice,
                               file:///Users/username/IdeaProjects/find_source_code/spring_cloud_config/src/main/resources/config/specialroutesservice,
                               file:///Users/username/IdeaProjects/find_source_code/spring_cloud_config/src/main/resources/config/authenticationservice
```
* maven依赖-Spring Cloud Config[客户端]
```
<!-- Spring Cloud客户端 依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>
```
* bootstrap.yml配置服务使用Spring Cloud Config[客户端]
```
spring:
  application:
    name: licensingservice
  profiles:
    active:
      default
  cloud:
    config:
      uri: http://localhost:8888
```
* 使用Spring Cloud配置服务器刷新属性@RefreshScope，访问http://localhost:8080[服务端口号]/refresh即可执行刷新。
```
@SpringBootApplication
@RefreshScope
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
* 保护敏感配置信息
   * 下载并安装加密所需的Oracle JCE jar；
   * 创建加密密匙；
   * 加密和解密属性；
   * 配置微服务以在客户端使用加密。
***
## 二，Spring Eureka
