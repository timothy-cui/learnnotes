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
* 配置profile bean
   * Spring为环境相关的bean等运行时根据激活的profile再确定，使用@Profile注解可以指定某个bean属于哪一个profile。
   * 激活profile
      * 依赖spring.profiles.active和spring.profiles.default来确定哪个profile处于激活状态（可通过多种方式设置）；
         * 使用DispatcherServlet设置到web.xml中；
         * 使用@ActiveProfiles注解在单元测试中激活指定profiles。
* 条件化的bean 
   * 使用@Conditional注解，通过实现了Condition接口的matches方法，该方法返回true则创建该bean，为false则不创建；
* 处理自动装配的歧义性
   * 通过@Primary注解标明首选bean；
   * 通过@Qualifier注解限定自动装配的bean - 组合@Autowired或者Component使用（通过声明自定义的限定符注解，可以同时使用多个限定符）。
* bean的作用域
   * Spring的作用域包括： 
      * 单例（Singleton）：在整个应用中，只创建bean的一个实例；
      * 原型（Prototype）: 每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例；
      * 会话（Session）：在web应用中，为每个会话创建一个bean实例；
      * 请求（Request）：在web应用中，为每个请求创建一个bean实例。
   * 单例是默认的作用域，可通过@Scope注解选择其他作用域，其可与@Component和@Bean一起使用；
   * 使用会话和请求作用域：
      * 场景举例：单例（购物车），所有用户就都能共享这个购物车；原型（购物车），在应用中的一个地方创建了，另一个地方就看不到了；会话（购物车），与给定的用户相关联。
      * 会话作用域和请求作用域bean的依赖注入都是以作用域代理的方式进行注入的。
* 运行时值注入
   * 注入外部的值
      * 通过@PropertySource注解引用外部文件（文件会加载到Environment中）；
      * 通过注入的Environment，获取文件中的属性。
      * Spring装配中可使用占位符"${...}",可配合@Value注解使用。
   * 使用Spring表达式语言进行装配
      * spEL表达式可做较复杂运算，其中可引用java类型与bean进入计算...
      * spEL表达式要放到"#{...}"中，也可配合@Value注解使用；
### 面向切面的Spring —— AOP
* 在使用切面编程时，我们仍然可以在一个地方定义通用功能，但是可以通过声明的方式定义这个功能要以何种方式在何处应用，而无需修改受影响的类。
* AOP术语：
   * 通知（Advice）：前置通知；后置通知；返回通知；异常通知；环绕通知。
   * 连接点（join point）：在应用执行过程中**能够**插入切面的一个点。
   * 切点（pointcut）：匹配通知所要织入的一个或多个连接点；
   * 切面（Aspect）：通知和切点的结合；
   * 引入（introduction）：允许向现有的类添加新方法或属性；
   * 织入（weaving）：把切面应用到目标对象并创建新的代理对象的过程。
* Spring提供的四种类型的AOP支持（基于动态代理，局限于方法拦截）：
   * 基于代理的经典SpringAOP；
   * 纯POJO切面；
   * @AspectJ注解驱动的切面（推荐）；
   * 注入式AspectJ切面。
* 编写切点：
   * execution()指示器选择目标方法（连接点）作为切点；
   * within()指示器限制匹配切点所在包；
   * bean()指示器可使用beanID或者bean名称作为参数来限制切点只匹配特定的bean。
   * args()指示器限制连接点匹配参数为指定类型的方法，且同时可传递且切点参数到在通知中使用。
* 使用注解创建切面：
   * @Aspect注解标识切面pojo；
   * @After("切点表达式")注解，通知在切点方法返回或者抛出异常后调用；
   * @AfterReturning("切点表达式")注解，通知在切点方法返回后调用；
   * @AfterThrowing("切点表达式")注解，通知在切点方法抛出异常后调用；
   * @Around("切点表达式")注解，通知会将切点方法封装起来；
   * @Before("切点表达式")注解，通知会在切点方法调用之前执行；
   * @Pointcut("切点表达式")注解，在@Aspect切面定义内定义可重用的切点。
* 使用javaConfig配置：
   * @EnableAspectJAutoProxy注解启用AspectJ自动代理；
   * 将切面类配置为bean
```java
import javax.management.MXBean;

@Configuration
@EnableAspectJAutoProxy
@ComponentScan
public class ConcertConfig {

    @Bean
    public Audience audience() {
        // Audience 为使用@Aspect的切面类。
        return new Audience();
    }
}
```
* 创建环绕通知：
   * 环绕通知方法接受ProceedingJoinPoint作为参数，通过调用其proceed方法，触发且切点方法。
   * 不调用proceed方法，会阻塞切点方法；调用多次proceed方法，会多次触发切点方法。
* 通过注解引入新功能：
   * 可通过切面@Aspect和@DeclareParents注解为目标类引入新的接口（功能），不用修改目标类。
* 对于没有源码无法使用AspectJ注解的情形，可使用xml的方式声明切面（此处不做赘述）。
* 若SpringAOP切面无法满足功能需求，可考虑注入AspectJ切面。其功能更强大（具体此处不做赘述）。
***
## 二，Spring在Web中的应用——SpringMVC、Spring Web Flow与Spring Security；



