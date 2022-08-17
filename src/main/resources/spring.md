# Spring实战笔记
***
## 相关知识点综述
* 一，Spring核心——依赖注入（DI）与面向切面编程（AOP）；
* 二，Spring在Web中的应用——SpringMVC、Spring Web Flow与Spring Security；
* 三，Spring在后端中的应用——Spring与JDBC、关系型数据库、NoSQL数据库、缓存以及Spring Security保护方法应用；
* 四，Spring集成应用——使用远程服务、使用SpringMVC创建RESTfulAPI、Spring异步消息、Spring使用WebSocket与STOMP实现异步消息功能、Spring实现邮件发送以及JMX管理Spring bean；
* 五，SpringBoot简介。
***
## 使用Spring简化java开发
* 为了降低java开发的复杂性，Spring采取了以下四种关键策略：
   * 基于POJO的轻量级和最小侵入性编程 —— 基于POJO；
   * 通过依赖注入和面向接口实现松耦合 —— 依赖注入DI；
   * 基于切面和惯例进行声明式编程 —— 面向切面编程AOP；
   * 通过切面和模版减少样板式代码 —— 使用模板消除样板式代码。
* Spring容器
   * 在基于Spring的应用中，应用对象生存于Spring容器（container）中；
   * Spring自带多个容器实现，可以归为两种不同类型：
      * bean工厂 - 最简单的，提供基本的DI支持；
      * 应用上下文 - 提供应用框架级别的服务。
* 依赖注入（DI） 和 AOP 是Spring框架的最核心的部分。
***
## 一，Spring核心——依赖注入（DI）与面向切面编程（AOP）
### Bean的装配 —— DI
* Spring配置的可选方案：
   * 在XML中配置；
   * 在Java中配置（推荐）：
       * 创建配置类（JavaConfig） —— @Configuration注解标识后被组件扫描生成bean；
       * 声明简单的bean —— @Bean注解标识方法返回的对象会被当作bean放入容器。
       * 在JavaConfig中导入其他的java配置 —— @Import注解
       * 在JavaConfig中导入其他的XML配置 —— @ImportResource注解
   * 隐式的bean发现机制和自动装配（推荐）：
       * 组件扫描 —— @ComponentScan注解或者xml配置启动扫描（@Component注解命名bean/@Name-用得少）。
       * 自动装配 —— Spring自动满足bean之间的依赖（@Autowired - 可用于属性/构造器上进行装配）。

