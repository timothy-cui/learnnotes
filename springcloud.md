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
