# SpringCloud微服务实操笔记
***
## 相关知识点综述
   * 一，Spring Cloud Config : 用于微服务中管理应用程序的配置文件；
   * 二，Spring Eureka : 用于定位应用程序资源的物理位置；
   * 三，Spring Cloud 和 Netflix Hystrix : 实现客户端弹性模式，在远程服务发生错误或者表现不佳时保护远程资源的客户端免于崩溃；
   * 四，Spring Cloud 和 Zuul : 实现服务网关，充当服务客户端和被调用的服务之间的中介；
   * 五，Spring Cloud 和 OAuth2  : 用于保护微服务，实现令牌机制；
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
* [本文实操github代码库](https://github.com/timothy-cui/find_source_code)
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
### 构建Spring Cloud Config服务
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

/*
 * @EnableConfigServer 使得服务成为Spring Cloud Config服务。
 */
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
### 微服务引入Spring Cloud Config客户端
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
// 相关依赖
<!-- spring boot actuator 依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
```
@SpringBootApplication
@RefreshScope
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
* 保护敏感配置信息[*未实现*]
   * 下载并安装加密所需的Oracle JCE jar；
   * 创建加密密匙；
   * 加密和解密属性；
   * 配置微服务以在客户端使用加密。
***
## 二，Spring Eureka
* 服务发现机制的特点：高可用、点对点、负载均衡、有弹性、容错；
* 服务发现架构相关的四个概念：
   * 服务注册；
   * 服务地址的客户端查找；
   * 信息共享；
   * 健康监测。
* 服务客户端完全依赖于服务发现引擎来查找和调用服务：服务消费者每次调用注册的微服务时，都会调用服务发现引擎解析地址；
* 客户端负载均衡，当服务消费者需要调用一个服务时：
   1. 联系服务发现服务，获取它请求的所有服务实例，然后在消费者的机器上本地缓存数据；
   2. 每当客户端需要调用该服务时，服务消费者将从缓存中查找该服务的位置信息；
   3. 客户端将定期与服务发现服务进行联系，并刷新服务实例的缓存。
   4. 如果调用服务的过程中，服务调用失败，那么本地的服务发现缓存失效，服务发现客户端将尝试从服务发现代理刷新数据。
* Spring Cloud实现方案：
   * 服务启动，通过Eureka服务进行注册。该过程会告诉Eureka每个服务实例的物理位置和端口号，以及正在启动的服务的服务ID；
   * 进行服务调用时，通过Netflix Ribbon库来提供客户端负载均衡。Ribbon将联系Eureka服务去检索服务位置信息，然后在本地缓存；
   * Netflix Ribbon将定期对Eureka服务进行ping操作，并刷新服务位置的缓存。
### 构建Spring Eureka服务
* 引入依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```
* 创建application.yml文件
```
server:
  port: 8761

eureka:
  client:
    registerWithEureka: false    // 不要使用Eureka服务进行注册
    fetchRegistry: false         // 不要在本地缓存注册表信息
  server:
    waitTimeInMsWhenSyncEmpty: 5 // 在服务器接受请求之前等待的初试时间
```
* 创建服务引导类
```
package com.thoughtmechanix.eurekasvr;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

/*
 * @EnableEurekaServer 在服务中启用Eureka服务器。
 */
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```
### 通过Spring Eureka注册服务
* 微服务引入依赖
```
<!-- 引入Eureka依赖, 以便可以使用Eureka注册服务 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
* 微服务的application.yml文件设置
```
eureka:
  instance:
    preferIpAddress: true     // 注册服务的ip，而不是服务器名称
  client:
    registerWithEureka: true  // 向Eureka注册服务
    fetchRegistry: true       // 拉取注册表本地副本
    serviceUrl:               // Eureka服务的位置
      defaultZone: http://localhost:8761/eureka/
```
* 注册服务后，http://localhost:8761/eureka/apps/<APPID>,查看该服务对应的所有实例。
### 使用服务发现来查找服务
1. 使用Spring DiscoveryClient查找服务实例
   * 其可以通过Ribbon注册的所有服务以及这些服务对应的URL。
   * 创建引导类
```
/*
 * @EnableDiscoveryClient 激活Spring DiscoveryClient。
 */
@SpringBootApplication
@EnableDiscoveryClient
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
* * 通过Spring DiscoveryClient进行调用
```
package com.thoughtmechanix.licenses.clients;


import com.thoughtmechanix.licenses.model.Organization;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

import java.util.List;

@Component
public class OrganizationDiscoveryClient {

    @Autowired
    private DiscoveryClient discoveryClient;

    public Organization getOrganization(String organizationId) {
        RestTemplate restTemplate = new RestTemplate();
        List<ServiceInstance> instances = discoveryClient.getInstances("organizationservice");

        if (instances.size() == 0) return null;
        String serviceUri = String.format("%s/v1/organizations/%s", instances.get(0).getUri().toString(), organizationId);
        System.out.println("!!!! SERVICE URI:  " + serviceUri);


        ResponseEntity<Organization> restExchange =
                restTemplate.exchange(
                        serviceUri,
                        HttpMethod.GET,
                        null, Organization.class, organizationId);

        return restExchange.getBody();
    }
}
```
2. 使用带有Ribbon功能的Spring RestTemplate调用服务
   * 创建引导类
```
/*
 * @EnableDiscoveryClient 激活Spring DiscoveryClient。
 */
@SpringBootApplication
public class Application {

    // @LoadBalanced告诉spring cloud注入一个支持Ribbon的RestTemplate类。
    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
* * 使用带有Ribbon功能的Spring RestTemplate调用服务
```
package com.thoughtmechanix.licenses.clients;

import com.thoughtmechanix.licenses.model.Organization;
import com.thoughtmechanix.licenses.repository.OrganizationRedisRepository;
import com.thoughtmechanix.licenses.utils.UserContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.security.oauth2.client.OAuth2RestTemplate;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

@Component
public class OrganizationRestTemplateClient {

    @Autowired
    RestTemplate restTemplate;

    public Organization getOrganization(String organizationId){
        /*
        ResponseEntity<Organization> restExchange =
                restTemplate.exchange(
                        "http://organizationservice/v1/organizations/{organizationId}",
                        HttpMethod.GET,
                        null, Organization.class, organizationId);
         */
        /*Save the record from cache*/
        org = restExchange.getBody();
        return org;
    }
}
```
3. 使用Netflix Feign客户端调用服务
   * Netflix的Feign库是Spring启用Ribbon的RestTemplate类的替代方案。
   * 创建引导类
```
/*
 * @EnableFeignClients启用Feign客户端。
 */
@SpringBootApplication
@EnableFeignClients
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
* * 定义用于调用其他服务的Fegin接口
```
package com.thoughtmechanix.licenses.clients;

import com.thoughtmechanix.licenses.model.Organization;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

/*
 * 使用@FeignClient标识服务
 */
@FeignClient("organizationservice")
public interface OrganizationFeignClient {

    // 使用@RequestMapping注解定义端点的路径和动作。使用@PathVariable来定义传入端点的参数。
    @RequestMapping(
            method = RequestMethod.GET,
            value = "/v1/organizations/{organizationId}",
            consumes = "application/json")
    Organization getOrganization(@PathVariable("organizationId") String organizationId);
}
```
***
## 三，Spring Cloud 和 Netflix Hystrix
* 所有的系统，特别是分布式系统，都会遇到故障。如何构建应用程序来应对这种故障，是每个软件开发工作人员工作的关键部分。
* 什么是客户端弹性模式？
   * 在远程服务发生错误或表现不佳时保护远程资源的客户端免于崩溃。
   * 有四种客户端弹性模式：
      1. 客户端负载均衡模式：客户端负载均衡器位于服务客户端和消费者之间，其可以检测服务实例是否抛出错误或者表现不佳。如果检测到问题，可以从可用服务位置池中移除该服务实例，并防止将来的服务调用访问该服务实例；
      2. 断路器模式：当远程服务被调用时，会监视这个调用，如果调用时间长，软件断路器会介入并中断调用；
      3. 后备模式：当远程服务调用失败时，服务消费者将执行替代代码路径，并尝试通过其他方式执行操作，而不是生成一个异常；
      4. 舱壁模式：把远程资源的调用分到线程池中，并降低一个缓慢的远程资源调用拖垮整个应用程序的风险。
### 微服务中引入Hystrix
* 引入依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```
* 引导类
```
/*
 * @EnableHystrix告诉Spring Cloud将要为服务使用Hystrix。
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableHystrix
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
* 用短路器包装远程资源调用
```
// 调用时间超过1000ms时，中断器将中断调用
@HystrixCommand
private Organization getOrganization(String organizationId) {
    randomlyRunLong();
    return organizationRestClient.getOrganization(organizationId);
}
private void randomlyRunLong(){
    Random rand = new Random();

    int randomNum = rand.nextInt((3 - 1) + 1) + 1;

    if (randomNum==3) {
        sleep();
    }
}

private void sleep(){
    try {
        Thread.sleep(11000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```
* 定制断路器的超时时间
```
// 设置超时时间为12s
@HystrixCommand(
    commandProperties = {
        @HystrixProperty(
            name = "execution.isolation.thread.timeoutInMilliseconds",
            value = "120000")})
private Organization getOrganization(String organizationId) {
    randomlyRunLong();
    return organizationRestClient.getOrganization(organizationId);
}
```
* 后备处理
```
@HystrixCommand(fallbackMethod = "buildFallbackLicenseList")
public List<License> getLicensesByOrg(String organizationId){
    logger.info("LicenseService.getLicensesByOrg  Correlation id: {}",
            UserContextHolder.getContext().getCorrelationId());
    randomlyRunLong();

    return licenseRepository.findByOrganizationId(organizationId);
}

private List<License> buildFallbackLicenseList(String organizationId){
    List<License> fallbackList = new ArrayList<>();
    License license = new License()
            .withId("0000000-00-00000")
            .withOrganizationId( organizationId )
            .withProductName("Sorry no licensing information currently available");

    fallbackList.add(license);
    return fallbackList;
}
```
* 实现舱壁模式
```
@HystrixCommand(
        threadPoolKey = "licenseByOrgThreadPool",
        threadPoolProperties = {
                @HystrixProperty(name = "coreSize",value="30"),
                @HystrixProperty(name="maxQueueSize", value="10")}
)
public List<License> getLicensesByOrg(String organizationId){
    logger.info("LicenseService.getLicensesByOrg  Correlation id: {}",
            UserContextHolder.getContext().getCorrelationId());
    randomlyRunLong();

    return licenseRepository.findByOrganizationId(organizationId);
}
```
* 微调Hystrix
```
@HystrixCommand(
        fallbackMethod = "buildFallbackLicenseList",
        threadPoolKey = "licenseByOrgThreadPool",
        threadPoolProperties = {
                @HystrixProperty(name = "coreSize",value="30"),
                @HystrixProperty(name="maxQueueSize", value="10")},
        commandProperties={
                @HystrixProperty(name="circuitBreaker.requestVolumeThreshold", value="10"),
                @HystrixProperty(name="circuitBreaker.errorThresholdPercentage", value="75"),
                @HystrixProperty(name="circuitBreaker.sleepWindowInMilliseconds", value="7000"),
                @HystrixProperty(name="metrics.rollingStats.timeInMilliseconds", value="15000"),
                @HystrixProperty(name="metrics.rollingStats.numBuckets", value="5")}
)
public List<License> getLicensesByOrg(String organizationId){
    logger.info("LicenseService.getLicensesByOrg  Correlation id: {}",
            UserContextHolder.getContext().getCorrelationId());
    randomlyRunLong();

    return licenseRepository.findByOrganizationId(organizationId);
}
```
* 在配置Hystrix环境时，有三个配置级别：
   * 整个应用程序级别的默认值；
   * 类级别的默认值；
   * 在类中定义的线程池级别。
### 线程上下文和Hystrix
* 默认情况下，使用线程隔离策略，可以进行设置更改。
* ThreadLocal和Hystrix
   * 对被父线程调用并被@HystrixCommand保护的方法而言，在父线程中设置的ThreadLocal值的值是不可用的。
   * 通过UserContextHolder存储UserContext信息，并在UserContextFilter中进行设置，然后通过自定义Hystrix并发策略类进行传递[**实际可直接通过Spring Cloud Sleuth实现**]：
```
package com.thoughtmechanix.licenses.utils;

import org.springframework.stereotype.Component;

@Component
public class UserContext {
    public static final String CORRELATION_ID = "tmx-correlation-id";
    public static final String AUTH_TOKEN     = "tmx-auth-token";
    public static final String USER_ID        = "tmx-user-id";
    public static final String ORG_ID         = "tmx-org-id";

    private String correlationId= new String();
    private String authToken= new String();
    private String userId = new String();
    private String orgId = new String();

    public String getCorrelationId() { return correlationId;}
    public void setCorrelationId(String correlationId) {
        this.correlationId = correlationId;
    }

    public String getAuthToken() {
        return authToken;
    }

    public void setAuthToken(String authToken) {
        this.authToken = authToken;
    }

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getOrgId() {
        return orgId;
    }

    public void setOrgId(String orgId) {
        this.orgId = orgId;
    }

}
```
```
package com.thoughtmechanix.licenses.utils;


import org.springframework.util.Assert;

public class UserContextHolder {
    private static final ThreadLocal<UserContext> userContext = new ThreadLocal<UserContext>();

    public static final UserContext getContext(){
        UserContext context = userContext.get();

        if (context == null) {
            context = createEmptyContext();
            userContext.set(context);

        }
        return userContext.get();
    }

    public static final void setContext(UserContext context) {
        Assert.notNull(context, "Only non-null UserContext instances are permitted");
        userContext.set(context);
    }

    public static final UserContext createEmptyContext(){
        return new UserContext();
    }
}
```
```
package com.thoughtmechanix.licenses.utils;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

@Component
public class UserContextFilter implements Filter {
    private static final Logger logger = LoggerFactory.getLogger(UserContextFilter.class);

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {


        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;

        UserContextHolder.getContext().setCorrelationId(httpServletRequest.getHeader(UserContext.CORRELATION_ID));
        UserContextHolder.getContext().setUserId(httpServletRequest.getHeader(UserContext.USER_ID));
        UserContextHolder.getContext().setAuthToken(httpServletRequest.getHeader(UserContext.AUTH_TOKEN));
        UserContextHolder.getContext().setOrgId(httpServletRequest.getHeader(UserContext.ORG_ID));
        UserContextHolder.getContext().setCorrelationId("tgcuitest");
        logger.info("UserContextFilter Correlation id: {}", UserContextHolder.getContext().getCorrelationId());

        filterChain.doFilter(httpServletRequest, servletResponse);
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void destroy() {}
}
```
```
package com.thoughtmechanix.licenses.hystrix;

import com.netflix.hystrix.HystrixThreadPoolKey;
import com.netflix.hystrix.strategy.concurrency.HystrixConcurrencyStrategy;
import com.netflix.hystrix.strategy.concurrency.HystrixRequestVariable;
import com.netflix.hystrix.strategy.concurrency.HystrixRequestVariableLifecycle;
import com.netflix.hystrix.strategy.properties.HystrixProperty;
import com.thoughtmechanix.licenses.utils.UserContextHolder;

import java.util.concurrent.BlockingQueue;
import java.util.concurrent.Callable;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;


public class ThreadLocalAwareStrategy extends HystrixConcurrencyStrategy{
    private HystrixConcurrencyStrategy existingConcurrencyStrategy;

    /*
     * spring Cloud已经定义了一个并发类，将已存在的并发策略通过构造器传入
     * 所有可能被调用的方法都要检查现有的并发策略是否存在，如存在则调用现有的并发策略方法，否则调用基类的方法
     */
    public ThreadLocalAwareStrategy(
            HystrixConcurrencyStrategy existingConcurrencyStrategy) {
        this.existingConcurrencyStrategy = existingConcurrencyStrategy;
    }


    /*
     * 方法重写，要么调用existingConcurrencyStrategy，要么调用基类
     */
    @Override
    public BlockingQueue<Runnable> getBlockingQueue(int maxQueueSize) {
        return existingConcurrencyStrategy != null
                ? existingConcurrencyStrategy.getBlockingQueue(maxQueueSize)
                : super.getBlockingQueue(maxQueueSize);
    }

    /*
     * 方法重写，要么调用existingConcurrencyStrategy，要么调用基类
     */
    @Override
    public <T> HystrixRequestVariable<T> getRequestVariable(
            HystrixRequestVariableLifecycle<T> rv) {
        return existingConcurrencyStrategy != null
                ? existingConcurrencyStrategy.getRequestVariable(rv)
                : super.getRequestVariable(rv);
    }

    /*
     * 方法重写，要么调用existingConcurrencyStrategy，要么调用基类
     */
    @Override
    public ThreadPoolExecutor getThreadPool(HystrixThreadPoolKey threadPoolKey,
                                            HystrixProperty<Integer> corePoolSize,
                                            HystrixProperty<Integer> maximumPoolSize,
                                            HystrixProperty<Integer> keepAliveTime, TimeUnit unit,
                                            BlockingQueue<Runnable> workQueue) {
        return existingConcurrencyStrategy != null
                ? existingConcurrencyStrategy.getThreadPool(threadPoolKey, corePoolSize,
                maximumPoolSize, keepAliveTime, unit, workQueue)
                : super.getThreadPool(threadPoolKey, corePoolSize, maximumPoolSize,
                keepAliveTime, unit, workQueue);
    }

    /*
     * 注入Callable实现，其将设置UserContext
     */
    @Override
    public <T> Callable<T> wrapCallable(Callable<T> callable) {
        return existingConcurrencyStrategy != null
                ? existingConcurrencyStrategy
                .wrapCallable(new DelegatingUserContextCallable<T>(callable, UserContextHolder.getContext()))
                : super.wrapCallable(new DelegatingUserContextCallable<T>(callable, UserContextHolder.getContext()));
    }
}
```
```
package com.thoughtmechanix.licenses.hystrix;

import com.thoughtmechanix.licenses.utils.UserContext;
import com.thoughtmechanix.licenses.utils.UserContextHolder;

import java.util.concurrent.Callable;


public final class DelegatingUserContextCallable<V> implements Callable<V> {
    private final Callable<V> delegate;
    private UserContext originalUserContext;

    /*
     * 原始callable被传递到自定义的callable类中，自定义的callable将使用hystrix保护的代码和来自父线程的UserContext。
     */
    public DelegatingUserContextCallable(Callable<V> delegate,
                                             UserContext userContext) {
        this.delegate = delegate;
        this.originalUserContext = userContext;
    }

    // call()方法在被@HystrixCommand注解保护的方法之前调用。
    public V call() throws Exception {
        // 已设置UserContext，存储UserContext的UserContextHolder将与运行hystrix保护的方法的线程相关联。
        UserContextHolder.setContext( originalUserContext );

        // UserContext设置后，在hystrix保护的方法上调用call()方法。
        try {
            return delegate.call();
        }
        finally {
            this.originalUserContext = null;
        }
    }

    public static <V> Callable<V> create(Callable<V> delegate,
                                         UserContext userContext) {
        return new DelegatingUserContextCallable<V>(delegate, userContext);
    }
}
```
```
package com.thoughtmechanix.licenses.hystrix;

import com.netflix.hystrix.strategy.HystrixPlugins;
import com.netflix.hystrix.strategy.concurrency.HystrixConcurrencyStrategy;
import com.netflix.hystrix.strategy.eventnotifier.HystrixEventNotifier;
import com.netflix.hystrix.strategy.executionhook.HystrixCommandExecutionHook;
import com.netflix.hystrix.strategy.metrics.HystrixMetricsPublisher;
import com.netflix.hystrix.strategy.properties.HystrixPropertiesStrategy;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;

@Configuration
public class ThreadLocalConfiguration {
        @Autowired(required = false)
        private HystrixConcurrencyStrategy existingConcurrencyStrategy;

        @PostConstruct
        public void init() {
            // Keeps references of existing Hystrix plugins.
            HystrixEventNotifier eventNotifier = HystrixPlugins.getInstance()
                    .getEventNotifier();
            HystrixMetricsPublisher metricsPublisher = HystrixPlugins.getInstance()
                    .getMetricsPublisher();
            HystrixPropertiesStrategy propertiesStrategy = HystrixPlugins.getInstance()
                    .getPropertiesStrategy();
            HystrixCommandExecutionHook commandExecutionHook = HystrixPlugins.getInstance()
                    .getCommandExecutionHook();

            HystrixPlugins.reset();

            // register all Hystrix plugins
            HystrixPlugins.getInstance().registerConcurrencyStrategy(new ThreadLocalAwareStrategy(existingConcurrencyStrategy));
            HystrixPlugins.getInstance().registerEventNotifier(eventNotifier);
            HystrixPlugins.getInstance().registerMetricsPublisher(metricsPublisher);
            HystrixPlugins.getInstance().registerPropertiesStrategy(propertiesStrategy);
            HystrixPlugins.getInstance().registerCommandExecutionHook(commandExecutionHook);
        }
}
```
***
## 四，Spring Cloud 和 Zuul
* 什么是服务网关？
   * 其充当服务客户端和被调用的服务之间的中介。有了服务网关，服务客户端永远不会直接调用单个服务的url，而是将所有的调用都放到服务网关上。
   * 由于服务网关位于客户端和各个服务的所有调用之间，故其还充当服务调用的中央策略执行点，可实现横切关注点，主要有以下几个方面：
      * 静态路由：服务网关将所有的服务调用放置在单个url和api路由的后面：
      * 动态路由：服务网关可以检查传入的服务请求，根据来自传入请求的数据和服务调用者的身份执行智能路由；
      * 验证和授权：其天然是检查服务调用者是否已经进行了验证并被授权进行服务调用的场所；
      * 度量数据收集和日志记录：可以通过服务网关来收集数据和日志信息，还可使用服务网关确保在用户请求上提供关键信息以确保日志统一。
* zuul的功能：
   * 将应用程序的所有服务的路由映射到一个URL；
   * 构建可以对通过网关的请求进行检查和操作的过滤器。
### 构建zuul服务
* 引入依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```
* 引导类
```
@SpringBootApplication
@EnableZuulProxy
public class ZuulServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZuulServerApplication.class, args);
    }
}
```
* 配置Zuul与Eureka通信【application.yml文件设置】
   * Zuul通过Eureka进行服务查找，通过Ribbon对来自Zuul的请求进行客户端负载均衡。
```
server:
  port: 5555

# http://localhost:5555/actuator/routes 访问服务映射表[其默认不暴露，故进行以下配置]
management:
  endpoints:
    web:
      exposure:
        include: routes

eureka:
  instance:
    preferIpAddress: true
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
        defaultZone: http://localhost:8761/eureka/
```
* 在Zuul中配置路由
   * 通过Eureka服务发现自动映射；
   * 使用服务发现手动配置【application.yml文件设置】；
```
# 手动映射配置【自动映射的路由需要手动配置去除，ignored-service: 'organizationservice', 加前缀，prefix:  /api】
#zuul:
#  prefix:  /api
#  ignored-service: 'organizationservice'
#  routes:
#    organizationservice: /organization/**
#    licensingservice: /licensing/**
```
 * * 可将配置放入spring cloud config，然后进行动态配置刷新。
 * 可通过配置对通过zuul进行服务调用设置超时属性。
 * Zuul的过滤器：前置过滤器、后置过滤器、路由过滤器
   * 构建一个生成关联ID的前置过滤器,配合UserContextFilter与UserContextInterceptor完成关联ID的传递；
```
 package com.thoughtmechanix.zuulsvr.filters;

import com.netflix.zuul.context.RequestContext;
import org.springframework.stereotype.Component;

@Component
public class FilterUtils {

    public static final String CORRELATION_ID = "tmx-correlation-id";
    public static final String AUTH_TOKEN = "tmx-auth-token";
    public static final String USER_ID = "tmx-user-id";
    public static final String ORG_ID = "tmx-org-id";
    public static final String PRE_FILTER_TYPE = "pre";
    public static final String POST_FILTER_TYPE = "post";
    public static final String ROUTE_FILTER_TYPE = "route";

    public String getCorrelationId() {
        RequestContext ctx = RequestContext.getCurrentContext();

        if (ctx.getRequest().getHeader(CORRELATION_ID) != null) {
            return ctx.getRequest().getHeader(CORRELATION_ID);
        } else {
            return ctx.getZuulRequestHeaders().get(CORRELATION_ID);
        }
    }

    public void setCorrelationId(String correlationId) {
        RequestContext ctx = RequestContext.getCurrentContext();
        // 单独的http首部映射，调用目标服务时会被合并
        ctx.addZuulRequestHeader(CORRELATION_ID, correlationId);
    }

    public final String getOrgId() {
        RequestContext ctx = RequestContext.getCurrentContext();
        if (ctx.getRequest().getHeader(ORG_ID) != null) {
            return ctx.getRequest().getHeader(ORG_ID);
        } else {
            return ctx.getZuulRequestHeaders().get(ORG_ID);
        }
    }

    public void setOrgId(String orgId) {
        RequestContext ctx = RequestContext.getCurrentContext();
        ctx.addZuulRequestHeader(ORG_ID, orgId);
    }

    public final String getUserId() {
        RequestContext ctx = RequestContext.getCurrentContext();
        if (ctx.getRequest().getHeader(USER_ID) != null) {
            return ctx.getRequest().getHeader(USER_ID);
        } else {
            return ctx.getZuulRequestHeaders().get(USER_ID);
        }
    }

    public void setUserId(String userId) {
        RequestContext ctx = RequestContext.getCurrentContext();
        ctx.addZuulRequestHeader(USER_ID, userId);
    }

    public final String getAuthToken() {
        RequestContext ctx = RequestContext.getCurrentContext();
        return ctx.getRequest().getHeader(AUTH_TOKEN);
    }

    public String getServiceId() {
        RequestContext ctx = RequestContext.getCurrentContext();

        //We might not have a service id if we are using a static, non-eureka route.
        if (ctx.get("serviceId") == null) return "";
        return ctx.get("serviceId").toString();
    }


}

```
```
 package com.thoughtmechanix.zuulsvr.filters;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
/*
 * 所有的zuul过滤器必须扩展ZuulFilter类，并覆盖4个方法
 */
@Component
public class TrackingFilter extends ZuulFilter{
    private static final int      FILTER_ORDER =  1;
    private static final boolean  SHOULD_FILTER=true;
    private static final Logger logger = LoggerFactory.getLogger(TrackingFilter.class);

    @Autowired
    FilterUtils filterUtils;

    // 设置过滤器类型
    @Override
    public String filterType() {
        return FilterUtils.PRE_FILTER_TYPE;
    }

    // 设置过滤器执行顺序
    @Override
    public int filterOrder() {
        return FILTER_ORDER;
    }

    // 设置过滤器是否执行
    @Override
    public boolean shouldFilter() {
        return SHOULD_FILTER;
    }

    // 每次服务通过过滤器时所执行的代码
    @Override
    public Object run() {

        if (isCorrelationIdPresent()) {
           logger.debug("tmx-correlation-id found in tracking filter: {}. ", filterUtils.getCorrelationId());
        }
        else{
            filterUtils.setCorrelationId(generateCorrelationId());
            logger.debug("tmx-correlation-id generated in tracking filter: {}.", filterUtils.getCorrelationId());
        }

        RequestContext ctx = RequestContext.getCurrentContext();
        logger.debug("Processing incoming request for {}.",  ctx.getRequest().getRequestURI());
        return null;
    }

    private boolean isCorrelationIdPresent(){
        if (filterUtils.getCorrelationId() !=null){
            return true;
        }

        return false;
    }

    // 生成CorrelationId
    private String generateCorrelationId(){
        return java.util.UUID.randomUUID().toString();
    }
}
```
```
package com.thoughtmechanix.zuulsvr.config;

import com.thoughtmechanix.zuulsvr.utils.UserContextInterceptor;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

import java.util.Collections;
import java.util.List;

@Configuration
public class ConfigBeans {
    @Bean
    @LoadBalanced
    RestTemplate restTemplate(){
        RestTemplate template = new RestTemplate();
        List interceptors = template.getInterceptors();
        if (interceptors == null) {
            template.setInterceptors(Collections.singletonList(new UserContextInterceptor()));
        } else {
            interceptors.add(new UserContextInterceptor());
            template.setInterceptors(interceptors);
        }

        return template;
    }

```
```
package com.thoughtmechanix.zuulsvr.utils;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

@Component
public class UserContextFilter implements Filter {
    private static final Logger logger = LoggerFactory.getLogger(UserContextFilter.class);

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;

        UserContextHolder.getContext().setCorrelationId(httpServletRequest.getHeader(UserContext.CORRELATION_ID));
        UserContextHolder.getContext().setUserId(httpServletRequest.getHeader(UserContext.USER_ID));
        UserContextHolder.getContext().setAuthToken(httpServletRequest.getHeader(UserContext.AUTH_TOKEN));
        UserContextHolder.getContext().setOrgId(httpServletRequest.getHeader(UserContext.ORG_ID));

        logger.info("Special Routes Service Incoming Correlation id: {}", UserContextHolder.getContext().getCorrelationId());

        filterChain.doFilter(httpServletRequest, servletResponse);
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void destroy() {
    }
}
```
```
package com.thoughtmechanix.zuulsvr.utils;

import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpRequest;
import org.springframework.http.client.ClientHttpRequestExecution;
import org.springframework.http.client.ClientHttpRequestInterceptor;
import org.springframework.http.client.ClientHttpResponse;

import java.io.IOException;

public class UserContextInterceptor implements ClientHttpRequestInterceptor {
    @Override
    public ClientHttpResponse intercept(
            HttpRequest request, byte[] body, ClientHttpRequestExecution execution)
            throws IOException {

        HttpHeaders headers = request.getHeaders();
        headers.add(UserContext.CORRELATION_ID, UserContextHolder.getContext().getCorrelationId());
        headers.add(UserContext.AUTH_TOKEN, UserContextHolder.getContext().getAuthToken());

        return execution.execute(request, body);
    }
}
```
```
package com.thoughtmechanix.zuulsvr.utils;

import org.springframework.stereotype.Component;

@Component
public class UserContext {
    public static final String CORRELATION_ID = "tmx-correlation-id";
    public static final String AUTH_TOKEN     = "tmx-auth-token";
    public static final String USER_ID        = "tmx-user-id";
    public static final String ORG_ID         = "tmx-org-id";

    private String correlationId= new String();
    private String authToken= new String();
    private String userId = new String();
    private String orgId = new String();

    public String getCorrelationId() { return correlationId;}
    public void setCorrelationId(String correlationId) {
        this.correlationId = correlationId;
    }

    public String getAuthToken() {
        return authToken;
    }

    public void setAuthToken(String authToken) {
        this.authToken = authToken;
    }

    public String getUserId() {
        return userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    public String getOrgId() {
        return orgId;
    }

    public void setOrgId(String orgId) {
        this.orgId = orgId;
    }

}
```
```
package com.thoughtmechanix.zuulsvr.utils;


import org.springframework.util.Assert;

public class UserContextHolder {
    private static final ThreadLocal<UserContext> userContext = new ThreadLocal<UserContext>();

    public static final UserContext getContext(){
        UserContext context = userContext.get();

        if (context == null) {
            context = createEmptyContext();
            userContext.set(context);

        }
        return userContext.get();
    }

    public static final void setContext(UserContext context) {
        Assert.notNull(context, "Only non-null UserContext instances are permitted");
        userContext.set(context);
    }

    public static final UserContext createEmptyContext(){
        return new UserContext();
    }
}
```
* 构建接受关联ID的后置过滤器
```
package com.thoughtmechanix.zuulsvr.filters;

import brave.Tracer;
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class ResponseFilter extends ZuulFilter{
    private static final int  FILTER_ORDER=1;
    private static final boolean  SHOULD_FILTER=true;
    private static final Logger logger = LoggerFactory.getLogger(ResponseFilter.class);

    @Autowired
    Tracer tracer;
    
    @Autowired
    FilterUtils filterUtils;

    @Override
    public String filterType() {
        return FilterUtils.POST_FILTER_TYPE;
    }

    @Override
    public int filterOrder() {
        return FILTER_ORDER;
    }

    @Override
    public boolean shouldFilter() {
        return SHOULD_FILTER;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();

        logger.info("Adding the correlation id to the outbound headers. {}", filterUtils.getCorrelationId());
        ctx.getResponse().addHeader(FilterUtils.CORRELATION_ID, filterUtils.getCorrelationId());

        // 通过sleuth获取tracerId返回到response中。
        ctx.getResponse().addHeader("tmx-correlation-id", tracer.currentSpan().context().traceIdString());

        logger.info("Completing outgoing request for {}.", ctx.getRequest().getRequestURI());

        return null;
    }
}
```
* 构建路由过滤器
```
package com.thoughtmechanix.zuulsvr.filters;
import ...

@Component
public class SpecialRoutesFilter extends ZuulFilter {
    private static final int FILTER_ORDER =  1;
    private static final boolean SHOULD_FILTER =true;

    @Autowired
    FilterUtils filterUtils;

    @Autowired
    RestTemplate restTemplate;

    @Override
    public String filterType() {
        return filterUtils.ROUTE_FILTER_TYPE;
    }

    @Override
    public int filterOrder() {
        return FILTER_ORDER;
    }

    @Override
    public boolean shouldFilter() {
        return SHOULD_FILTER;
    }


    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();

        // 执行对specialRoutes服务的调用，以确定该服务id是否有路由记录。
        AbTestingRoute abTestRoute = getAbRoutingInfo( filterUtils.getServiceId() );

        if (abTestRoute!=null && useSpecialRoute(abTestRoute)) {
            // 如果有路由记录，构建route到specialRoutes指定的服务
            String route = buildRouteString(ctx.getRequest().getRequestURI(),
                    abTestRoute.getEndpoint(),
                    ctx.get("serviceId").toString());
            // 完成转发到其他服务的工作
            forwardToSpecialRoute(route);
        }

        return null;
    }

    private ProxyRequestHelper helper = new ProxyRequestHelper();

    private AbTestingRoute getAbRoutingInfo(String serviceName){
        ResponseEntity<AbTestingRoute> restExchange = null;
        try {
            restExchange = restTemplate.exchange(
                             "http://specialroutesservice/v1/route/abtesting/{serviceName}",
                             HttpMethod.GET,
                             null, AbTestingRoute.class, serviceName);
        }
        catch(HttpClientErrorException ex){
            if (ex.getStatusCode()== HttpStatus.NOT_FOUND) return null;
            throw ex;
        }
        return restExchange.getBody();
    }

    private String buildRouteString(String oldEndpoint, String newEndpoint, String serviceName){
        int index = oldEndpoint.indexOf(serviceName);

        String strippedRoute = oldEndpoint.substring(index + serviceName.length());
        System.out.println("Target route: " + String.format("%s/%s", newEndpoint, strippedRoute));
        return String.format("%s/%s", newEndpoint, strippedRoute);
    }

    private String getVerb(HttpServletRequest request) {
        String sMethod = request.getMethod();
        return sMethod.toUpperCase();
    }

    private HttpHost getHttpHost(URL host) {
        HttpHost httpHost = new HttpHost(host.getHost(), host.getPort(),
                host.getProtocol());
        return httpHost;
    }

    private Header[] convertHeaders(MultiValueMap<String, String> headers) {
        List<Header> list = new ArrayList<>();
        for (String name : headers.keySet()) {
            for (String value : headers.get(name)) {
                list.add(new BasicHeader(name, value));
            }
        }
        return list.toArray(new BasicHeader[0]);
    }

    private HttpResponse forwardRequest(HttpClient httpclient, HttpHost httpHost,
                                        HttpRequest httpRequest) throws IOException {
        return httpclient.execute(httpHost, httpRequest);
    }


    private MultiValueMap<String, String> revertHeaders(Header[] headers) {
        MultiValueMap<String, String> map = new LinkedMultiValueMap<String, String>();
        for (Header header : headers) {
            String name = header.getName();
            if (!map.containsKey(name)) {
                map.put(name, new ArrayList<String>());
            }
            map.get(name).add(header.getValue());
        }
        return map;
    }

    private InputStream getRequestBody(HttpServletRequest request) {
        InputStream requestEntity = null;
        try {
            requestEntity = request.getInputStream();
        }
        catch (IOException ex) {
            // no requestBody is ok.
        }
        return requestEntity;
    }

    private void setResponse(HttpResponse response) throws IOException {
        this.helper.setResponse(response.getStatusLine().getStatusCode(),
                response.getEntity() == null ? null : response.getEntity().getContent(),
                revertHeaders(response.getAllHeaders()));
    }

    private HttpResponse forward(HttpClient httpclient, String verb, String uri,
                                 HttpServletRequest request, MultiValueMap<String, String> headers,
                                 MultiValueMap<String, String> params, InputStream requestEntity)
            throws Exception {
        Map<String, Object> info = this.helper.debug(verb, uri, headers, params,
                requestEntity);
        URL host = new URL( uri );
        HttpHost httpHost = getHttpHost(host);

        HttpRequest httpRequest;
        int contentLength = request.getContentLength();
        InputStreamEntity entity = new InputStreamEntity(requestEntity, contentLength,
                request.getContentType() != null
                        ? ContentType.create(request.getContentType()) : null);
        switch (verb.toUpperCase()) {
            case "POST":
                HttpPost httpPost = new HttpPost(uri);
                httpRequest = httpPost;
                httpPost.setEntity(entity);
                break;
            case "PUT":
                HttpPut httpPut = new HttpPut(uri);
                httpRequest = httpPut;
                httpPut.setEntity(entity);
                break;
            case "PATCH":
                HttpPatch httpPatch = new HttpPatch(uri );
                httpRequest = httpPatch;
                httpPatch.setEntity(entity);
                break;
            default:
                httpRequest = new BasicHttpRequest(verb, uri);

        }
        try {
            httpRequest.setHeaders(convertHeaders(headers));
            HttpResponse zuulResponse = forwardRequest(httpclient, httpHost, httpRequest);

            return zuulResponse;
        }
        finally {
        }
    }

    /*
     * 根据权重，判断是否转发到替代服务
     */
    public boolean useSpecialRoute(AbTestingRoute testRoute){
        Random random = new Random();

        if (testRoute.getActive().equals("N")) return false;

        int value = random.nextInt((10 - 1) + 1) + 1;

        if (testRoute.getWeight()<value) return true;

        return false;
    }

    private void forwardToSpecialRoute(String route) {
        RequestContext context = RequestContext.getCurrentContext();
        HttpServletRequest request = context.getRequest();

        MultiValueMap<String, String> headers = this.helper
                .buildZuulRequestHeaders(request);
        MultiValueMap<String, String> params = this.helper
                .buildZuulRequestQueryParams(request);
        String verb = getVerb(request);
        InputStream requestEntity = getRequestBody(request);
        if (request.getContentLength() < 0) {
            context.setChunkedRequestBody();
        }

        this.helper.addIgnoredHeaders();
        CloseableHttpClient httpClient = null;
        HttpResponse response = null;

        try {
            httpClient  = HttpClients.createDefault();
            response = forward(httpClient, verb, route, request, headers,
                    params, requestEntity);
            setResponse(response);
        }
        catch (Exception ex ) {
            ex.printStackTrace();

        }
        finally{
            try {
                httpClient.close();
            }
            catch(IOException ex){}
        }
    }
}
```
* 与微服务调用相整合，可直接通过与Zuul服务的调用进行具体业务逻辑服务调用。
***
## 五，Spring Cloud 和 OAuth2 
* OAuth2是一个基于令牌的安全验证和授权框架，具有以下四种类型的授权：
   * 密码；
   * 客户端凭证；
   * 授权码；
   * 隐式。
### 使用 Spring Cloud 和 OAuth2来保护单个端点
* OAuth2验证服务的maven依赖
```
<!-- 引入oauth2依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
```
* 引导类
```
package com.thoughtmechanix.authentication;
import ...

@SpringBootApplication
@RestController
@EnableResourceServer
@EnableAuthorizationServer
public class Application {
    @RequestMapping(value = { "/user" }, produces = "application/json")
    public Map<String, Object> user(OAuth2Authentication user) {
        Map<String, Object> userInfo = new HashMap<>();
        userInfo.put("user", user.getUserAuthentication().getPrincipal());
        userInfo.put("authorities", AuthorityUtils.authorityListToSet(user.getUserAuthentication().getAuthorities()));
        return userInfo;
    }


    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }


}
```
* 使用OAuth2服务注册客户端应用程序
```
package com.thoughtmechanix.authentication.security;
import ...

@Configuration
public class OAuth2Config extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private AuthenticationManager authenticationManager;

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("eagleeye")
                .secret(new BCryptPasswordEncoder().encode("thisissecret"))
                .authorizedGrantTypes("refresh_token", "password", "client_credentials")
                .scopes("webclient", "mobileclient");
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception {
      endpoints
        .authenticationManager(authenticationManager)
        .userDetailsService(userDetailsService);
    }
}
```
* 配置用于【为了方便，这里基于内存进行配置】
```java
package com.thoughtmechanix.authentication.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;


@Configuration
public class WebSecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Bean
    PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    @Override
    @Bean
    public UserDetailsService userDetailsServiceBean() throws Exception {
        return super.userDetailsServiceBean();
    }


    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder())
                .withUser("john.carnell").password(new BCryptPasswordEncoder().encode("password1"))
                .roles("USER").and()
                .withUser("william.woodward").password(new BCryptPasswordEncoder().encode("password2"))
                .roles("USER", "ADMIN");
    }
}
```
### 使用OAuth2保护目标微服务
* 向服务中添加依赖
```xml
<!-- 引入oauth2依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
</dependency>
```
* 配置指向OAuth2验证服务【application.yml】
```yaml
security:
  oauth2:
    resource:
      userInfoUri: http://localhost:8901/auth/user
```
* 配置引导类
```java
/*
 * @EnableResourceServer 告诉微服务，其是一个受保护的资源。   
 */
@SpringBootApplication
@RefreshScope
@EnableEurekaClient
@EnableResourceServer
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
* 定义谁可以访问【通过特定角色保护起来了】
```java
package com.thoughtmechanix.organization.security;


import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.oauth2.config.annotation.web.configuration.ResourceServerConfigurerAdapter;

@Configuration
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {


    @Override
    public void configure(HttpSecurity http) throws Exception{
        http
        .authorizeRequests()
          .antMatchers(HttpMethod.DELETE, "/v1/organizations/**")
          .hasRole("ADMIN")
          .anyRequest()
          .authenticated();
    }
}
```
* OAuth2令牌的传播
   * Zuul路由服务配置传播【application.yml】
```yaml
# 黑名单，阻止Cookie,Set-Cookie的传播[如果不设置，zuul自动阻止Cookie,Set-Cookie,Authorization的传播]
zuul:
  sensitive-headers: Cookie,Set-Cookie
```
* * 服务调用时使用OAuth2RestTemplate
```java
package com.thoughtmechanix.licenses.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.oauth2.client.OAuth2ClientContext;
import org.springframework.security.oauth2.client.OAuth2RestTemplate;


@Configuration
public class ConfigBeans {
    @Bean
    public OAuth2RestTemplate oauth2RestTemplate(OAuth2ClientContext oauth2ClientContext,
                                                 OAuth2ProtectedResourceDetails details) {
        return new OAuth2RestTemplate(details, oauth2ClientContext);
    }
    // ...
    
}
```
```java
package com.thoughtmechanix.licenses.clients;

import com.thoughtmechanix.licenses.model.Organization;
import com.thoughtmechanix.licenses.repository.OrganizationRedisRepository;
import com.thoughtmechanix.licenses.utils.UserContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.security.oauth2.client.OAuth2RestTemplate;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

@Component
public class OrganizationRestTemplateClient {

    // OAuth2RestTemplate为RestTemplate的替代品。可以处理Authorization的传播
    @Autowired
    OAuth2RestTemplate restTemplate;

    @Autowired
    OrganizationRedisRepository orgRedisRepo;

    @Autowired
    UserContext userContext;

    private static final Logger logger = LoggerFactory.getLogger(OrganizationRestTemplateClient.class);
    

    public Organization getOrganization(String organizationId){

        // 通过zuul进行路由调用
        ResponseEntity<Organization> restExchange =
                restTemplate.exchange(
                        "http://zuulservice/api/organization/v1/organizations/{organizationId}",
                        HttpMethod.GET,
                        null, Organization.class, organizationId);

        /*Save the record from cache*/
        Organization org = restExchange.getBody();

        return org;
    }
}
```
* 拓展：使用JWT扩展令牌内容
### 关于微服务安全的总结
* 在构建用于生产级别的微服务时，应该围绕以下实践构建微服务安全：
   * 对所有服务通信使用https/安全套接字层；
   * 所有服务调用都应通过API网关；
   * 将服务划分到公共API和私有API；
   * 通过封锁不需要的网络端口来限制微服务的攻击面。
***
## 六，Spring Cloud Stream
* 实现事件驱动【消息驱动】，基于消息传递的解决方案。
* 使用消息队列作为中介的好处：
   * 松耦合；
   * 耐久性；
   * 可伸缩性；
   * 灵活性。
* 使用消息队列作为中介需要关注的方面：
   * 消息处理语义；
   * 消息可见行；
   * 消息编排。
* Spring Cloud Stream是一个由注解驱动的框架，其允许开发人员在Spring应用程序中轻松地构建消息发布者和消费者。
* Spring Cloud Stream架构包括：
   * 发射器：接收一个pojo，发布到通道；
   * 通道：对列的一个抽象，与目标队列相关联；队列名称不会在代码中直接使用，通道名称会；
   * 绑定器：与特定消息平台对话；
   * 接收器：接收消息并反序列化为pojo，进行处理。
### 在组织服务中编写消息生产者
* 引入依赖
```
<!--Spring Cloud Stream Dependencies-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-kafka</artifactId>
</dependency>
```
* 引导类，@EnableBinding绑定消息代理
```
@SpringBootApplication
@RefreshScope
@EnableEurekaClient
@EnableResourceServer
@EnableBinding(Source.class)
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```
* 向消息代理发布消息
```
package com.thoughtmechanix.organization.events.source;

import com.thoughtmechanix.organization.events.models.OrganizationChangeModel;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;
import com.thoughtmechanix.organization.utils.UserContext;


@Component
public class SimpleSourceBean {
    private Source source;
    @Autowired
    private UserContext userContext;

    private static final Logger logger = LoggerFactory.getLogger(SimpleSourceBean.class);

    // spring cloud stream将注入一个Source接口，以供服务使用。
    @Autowired
    public SimpleSourceBean(Source source) {
        this.source = source;
    }

    public void publishOrgChange(String action, String orgId) {
        logger.info("Sending Kafka message {} for Organization Id: {}", action, orgId);
        // 将要发布的消息[pojo]。
        OrganizationChangeModel change = new OrganizationChangeModel(
                OrganizationChangeModel.class.getTypeName(),
                action,
                orgId,
                userContext.getCorrelationId());

        // 使用Source类中定义的通道的send()方法准备发送消息。
        source.output().send(MessageBuilder.withPayload(change).build());
    }
}
```
* 配置用于发布消息的平台以及主题【application.yml】
```
spring:
  cloud:
    stream:
      bindings:
        output: // 映射到output通道
          destination:  orgChangeTopic   // 消息主题
          content-type: application/json // 消息格式
      kafka:
        binder:
          zkNodes: localhost  // zookeeper位置
          brokers: localhost  // kafka位置
```
* 在服务中发布消息
```java
package com.thoughtmechanix.organization.services;

import com.thoughtmechanix.organization.repository.OrganizationRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import com.thoughtmechanix.organization.events.source.SimpleSourceBean;
import com.thoughtmechanix.organization.model.Organization;

import java.util.UUID;

@Service
public class OrganizationService {
    @Autowired
    private OrganizationRepository orgRepository;

    @Autowired
    SimpleSourceBean simpleSourceBean;

    public Organization getOrg(String organizationId) {
        return orgRepository.findById(organizationId).orElse(null);
    }

    public void saveOrg(Organization org) {
        org.setId(UUID.randomUUID().toString());

        orgRepository.save(org);
        simpleSourceBean.publishOrgChange("SAVE", org.getId());
    }

    public void updateOrg(Organization org) {
        orgRepository.save(org);
        simpleSourceBean.publishOrgChange("UPDATE", org.getId());

    }

    public void deleteOrg(String orgId) {
        orgRepository.deleteById(orgId);
        simpleSourceBean.publishOrgChange("DELETE", orgId);
    }
}
```
### 在许可证服务中编写消息消费者实现分布式缓存刷新
* 引入依赖
```
<!--Spring Cloud Stream Dependencies-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-stream</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-stream-kafka</artifactId>
</dependency>

<!--Spring Data Redis dependencies-->
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.7.4.RELEASE</version>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.0</version>
</dependency>
```
* 引导类
```
// 省略...
@EnableBinding(Sink.class)
public class Application {
    private static final Logger logger = LoggerFactory.getLogger(Application.class);

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    // 每次收到消息，就执行该方法 for test。
    @StreamListener(Sink.INPUT)
    public void loggerSink(OrganizationChangeModel orgChange) {
        logger.debug("Received an event for organization id {}", orgChange.getOrganizationId());
    }
}
```
* 配置用于发布消息的平台以及主题【application.yml】
```
spring:
  cloud:
    stream:
      bindings:
        inboundOrgChanges: // 自定义的input通道[可用官方的：input]
          destination: orgChangeTopic
          content-type: application/json
          group: licensingGroup         // group属性用于保证服务只处理一次 
      kafka:
        binder:
          zkNodes: localhost
          brokers: localhost
```
* 构建一个redis数据库连接
```java
package com.thoughtmechanix.licenses.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.security.oauth2.client.OAuth2ClientContext;
import org.springframework.security.oauth2.client.OAuth2RestTemplate;
import org.springframework.security.oauth2.client.resource.OAuth2ProtectedResourceDetails;
import org.springframework.web.client.RestTemplate;

@Configuration
public class ConfigBeans {
    @Autowired
    private ServiceConfig serviceConfig;

    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }

    @Bean
    public OAuth2RestTemplate oauth2RestTemplate(OAuth2ClientContext oauth2ClientContext,
                                                 OAuth2ProtectedResourceDetails details) {
        return new OAuth2RestTemplate(details, oauth2ClientContext);
    }

    @Bean
    public JedisConnectionFactory jedisConnectionFactory() {
        JedisConnectionFactory jedisConnFactory = new JedisConnectionFactory();
        jedisConnFactory.setHostName(serviceConfig.getRedisServer());
        jedisConnFactory.setPort(serviceConfig.getRedisPort());
        return jedisConnFactory;
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(jedisConnectionFactory());
        return template;
    }
}
```
* 配置到sping cloud config
```java
package com.thoughtmechanix.licenses.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class ServiceConfig {

    @Value("${example.property}")
    private String exampleProperty;

    @Value("${redis.server}")
    private String redisServer="";

    @Value("${redis.port}")
    private String redisPort="";

    public String getRedisServer(){
        return redisServer;
    }

    public Integer getRedisPort(){
        return new Integer( redisPort ).intValue();
    }

    public String getExampleProperty() {
        return exampleProperty;
    }
}
```
* 定义spring data redis存储库
```java
package com.thoughtmechanix.licenses.repository;

import com.thoughtmechanix.licenses.model.Organization;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.HashOperations;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.stereotype.Repository;

import javax.annotation.PostConstruct;

@Repository
public class OrganizationRedisRepositoryImpl implements OrganizationRedisRepository {
    private static final String HASH_NAME ="organization";

    private RedisTemplate<String, Organization> redisTemplate;
    private HashOperations hashOperations;

    public OrganizationRedisRepositoryImpl(){
        super();
    }

    @Autowired
    private OrganizationRedisRepositoryImpl(RedisTemplate redisTemplate) {
        this.redisTemplate = redisTemplate;
    }

    @PostConstruct
    private void init() {
        hashOperations = redisTemplate.opsForHash();
    }


    @Override
    public void saveOrganization(Organization org) {
        hashOperations.put(HASH_NAME, org.getId(), org);
    }

    @Override
    public void updateOrganization(Organization org) {
        hashOperations.put(HASH_NAME, org.getId(), org);
    }

    @Override
    public void deleteOrganization(String organizationId) {
        hashOperations.delete(HASH_NAME, organizationId);
    }

    @Override
    public Organization findOrganization(String organizationId) {
       return (Organization) hashOperations.get(HASH_NAME, organizationId);
    }
}
```
```java
package com.thoughtmechanix.licenses.repository;

import com.thoughtmechanix.licenses.model.Organization;

public interface OrganizationRedisRepository {
    void saveOrganization(Organization org);
    void updateOrganization(Organization org);
    void deleteOrganization(String organizationId);
    Organization findOrganization(String organizationId);
}
```
* 缓存读取
```java
package com.thoughtmechanix.licenses.clients;

import com.thoughtmechanix.licenses.model.Organization;
import com.thoughtmechanix.licenses.repository.OrganizationRedisRepository;
import com.thoughtmechanix.licenses.utils.UserContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpMethod;
import org.springframework.http.ResponseEntity;
import org.springframework.security.oauth2.client.OAuth2RestTemplate;
import org.springframework.stereotype.Component;
import org.springframework.web.client.RestTemplate;

@Component
public class OrganizationRestTemplateClient {
    // OAuth2RestTemplate为RestTemplate的替代品。可以处理Authorization的传播
    @Autowired
    OAuth2RestTemplate restTemplate;

    @Autowired
    OrganizationRedisRepository orgRedisRepo;

    @Autowired
    UserContext userContext;

    private static final Logger logger = LoggerFactory.getLogger(OrganizationRestTemplateClient.class);

    private Organization checkRedisCache(String organizationId) {
        try {
            return orgRedisRepo.findOrganization(organizationId);
        }
        catch (Exception ex){
            logger.error("Error encountered while trying to retrieve organization {} check Redis Cache.  Exception {}",
                    organizationId, ex);
            return null;
        }
    }

    private void cacheOrganizationObject(Organization org) {
        try {
            orgRedisRepo.saveOrganization(org);
        }catch (Exception ex){
            logger.error("Unable to cache organization {} in Redis. Exception {}", org.getId(), ex);
        }
    }

    public Organization getOrganization(String organizationId){
        logger.debug("In Licensing Service.getOrganization: {}", userContext.getCorrelationId());

        Organization org = checkRedisCache(organizationId);

        if (org!=null){
            logger.debug("I have successfully retrieved an organization {} from the redis cache: {}", organizationId, org);
            return org;
        }

        logger.debug("Unable to locate organization from the redis cache: {}.", organizationId);

        // 通过zuul进行路由调用
        ResponseEntity<Organization> restExchange =
                restTemplate.exchange(
                        "http://zuulservice/api/organization/v1/organizations/{organizationId}",
                        HttpMethod.GET,
                        null, Organization.class, organizationId);

        /*Save the record from cache*/
        org = restExchange.getBody();

        if (org!=null) {
            cacheOrganizationObject(org);
        }

        return org;
    }
}
```
* 自定义通道
```java
package com.thoughtmechanix.licenses.events;


import org.springframework.cloud.stream.annotation.Input;
import org.springframework.messaging.SubscribableChannel;

public interface CustomChannels {
    @Input("inboundOrgChanges")
    SubscribableChannel orgs();
}
```
* 消息处理：收到消息时清除缓存
```java
package com.thoughtmechanix.licenses.events.handlers;

import com.thoughtmechanix.licenses.events.CustomChannels;
import com.thoughtmechanix.licenses.events.models.OrganizationChangeModel;
import com.thoughtmechanix.licenses.repository.OrganizationRedisRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;


@EnableBinding(CustomChannels.class)
public class OrganizationChangeHandler {

    @Autowired
    private OrganizationRedisRepository organizationRedisRepository;

    private static final Logger logger = LoggerFactory.getLogger(OrganizationChangeHandler.class);

    @StreamListener("inboundOrgChanges")
    public void loggerSink(OrganizationChangeModel orgChange) {
        logger.debug("Received a message of type " + orgChange.getType());
        switch(orgChange.getAction()){
            case "GET":
                logger.debug("Received a GET event from the organization service for organization id {}", orgChange.getOrganizationId());
                break;
            case "SAVE":
                logger.debug("Received a SAVE event from the organization service for organization id {}", orgChange.getOrganizationId());
                break;
            case "UPDATE":
                logger.debug("Received a UPDATE event from the organization service for organization id {}", orgChange.getOrganizationId());
                organizationRedisRepository.deleteOrganization(orgChange.getOrganizationId());
                break;
            case "DELETE":
                logger.debug("Received a DELETE event from the organization service for organization id {}", orgChange.getOrganizationId());
                organizationRedisRepository.deleteOrganization(orgChange.getOrganizationId());
                break;
            default:
                logger.error("Received an UNKNOWN event from the organization service of type {}", orgChange.getType());
                break;

        }
    }

}
```
***
## 七，Spring Cloud Sleuth 和 Zipkin
* 在服务中添加以下依赖，即可替换之前相关构建所有关联ID的基础设施代码
```
<!--Spring Cloud Sleuth dependency-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>
```
* 引入以上依赖，服务便会完成以下的功能：
   * 检查每个传入的HTTP服务，并确定调用中是否存在Spring Cloud Sleuth跟踪信息。如果存在，则将捕获并提供给服务进行日志记录和处理；
   * 将Spring Cloud Sleuth跟踪信息添加到Spring MDC，以便微服务创建的每个日志语句都添加到日志中；
   * 将Spring Cloud Sleuth跟踪信息注入服务发出的每个出战HTTP调用以及Spring消息传递通道的消息中。
* 日志聚合与Spring Cloud Sleuth
   * 将所有服务实例的日志实时流到一个集中的聚合点，在聚合点对日志数据进行索引并进行搜索；
   * 可选用Papertrail平台进行聚合实现以上功能。
* 使用Zuul将关联ID添加到HTTP响应
   * 将Spring Cloud Sleuth依赖加入到Zuul服务；
   * 在Zuul后置过滤器中添加Spring Cloud Sleuth的跟踪ID
```java
package com.thoughtmechanix.zuulsvr.filters;

import brave.Tracer;
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class ResponseFilter extends ZuulFilter{
    private static final int  FILTER_ORDER=1;
    private static final boolean  SHOULD_FILTER=true;
    private static final Logger logger = LoggerFactory.getLogger(ResponseFilter.class);

    @Autowired
    Tracer tracer;
    
    @Autowired
    FilterUtils filterUtils;

    @Override
    public String filterType() {
        return FilterUtils.POST_FILTER_TYPE;
    }

    @Override
    public int filterOrder() {
        return FILTER_ORDER;
    }

    @Override
    public boolean shouldFilter() {
        return SHOULD_FILTER;
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();

        logger.info("Adding the correlation id to the outbound headers. {}", filterUtils.getCorrelationId());
        ctx.getResponse().addHeader(FilterUtils.CORRELATION_ID, filterUtils.getCorrelationId());

        // 通过sleuth获取tracerId返回到response中。
        ctx.getResponse().addHeader("tmx-correlation-id", tracer.currentSpan().context().traceIdString());

        logger.info("Completing outgoing request for {}.", ctx.getRequest().getRequestURI());

        return null;
    }
}
```
### 使用Open Zipkin进行分布式跟踪
* Zipkin是一个分布式跟踪平台,可提供可视化界面，可用于跟踪跨度多个服务调用的事务。建立Spring Cloud Sleuth和Zipkin涉及以下四个操作：
   * 引入对应依赖到捕获跟踪数据的服务中；
   * 在每个服务中配置Spring属性以指向收集跟踪数据的Zipkin服务器；
   * 安装和配置Zipkin服务器以收集数据；
   * 定义每个客户端所使用的采样策略，便于向Zipkin发送跟踪信息。
* 在Zuul服务，许可证服务和组织服务中引入对应依赖。
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-sleuth</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-sleuth-zipkin</artifactId>
</dependency>
```
* 配置Zuul服务，许可证服务和组织服务指向Zipkin服务器。
```
spring:
  zipkin:
    baseUrl: localhost:9411
```
* 安装和配置Zipkin服务器
```
/*
 * pringboot 在2.0后就不推荐创建springboot项目启动zipkin server了,推荐自己搭建zipkin server【可直接运行官方jar包】
 * 因为你会发现spring boot 2.0 之后，你在引入依赖时必须要指定zipkin-server的版本号，而自己指定版本号会出现非常多的问题
 */
```
* 发生服务调用后，访问http://localhost:9411,查看Zipkin已经捕获的跟踪结果。
* 可添加自定义跨度，例如可添加跨度跟踪Redis和数据库查询相关的附加跟踪和计时信息。

