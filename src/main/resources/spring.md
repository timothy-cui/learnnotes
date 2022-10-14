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
### SpringMVC
* 基于模型-视图-控制器（MVC）模式实现。
* 跟踪SpringMVC请求：
   * 前端控制器（DispatcherServlet）：请求的第一站，单例的Servlet将请求委托给应用程序的其他组件来实际处理；
   * 处理器映射：请求的第二站，DispatcherServlet会来查询，并根据请求所携带的url信息进行决策；
   * 控制器：请求的第三站，通过处理器映射找到对应的controller后，DispatcherServlet会将请求发送过去；
   * 模型及逻辑视图名：请求的第四站，控制器处理完毕后将模型和逻辑视图名打包发给DispatcherServlet；
   * 视图解析器：请求的第五站，DispatcherServlet根据逻辑视图名到视图解析器获取视图；
   * 视图：请求的第六站，视图渲染数据后输出。
* 搭建SpringMVC简析：
   * 配置DispatcherServlet（一般配置在web.xml，也可以通过javaConfig配置）；
   * 两个上下文：spring上下文1（DispatcherServlet启动时创建，加载包含web组件的bean）；spring上下文2（ContextLoaderListener，加载驱动后端程序与数据库中间层的bean）- Servlet3.0可通过AbstractAnnotationConfigDispatcherServletInitializer同时配置创建。
   * 启用SpringMvc：基于@EnableWebMvc注解搭建配置类；
* 编写控制器：
   * @Controller注解辅助实现控制器组件扫描；
   * 基于@RequestMapping注解实现请求方法的route和处理的请求类型（该注解可用于类上，定义该类中所有方法的基本route），@RequestMapping(value = "/", method = GET)。
   * 传递模型数据到视图中：
```java
@Controller
@RequestMapping("/spittles")
public class SpittlerController {
    private SpittlerRepository spittlerRepository;
    
    @Autowired
    public SpittlerController(SpittlerRepository spittlerRepository) {
        this.spittlerRepository = spittlerRepository;
    }
    
    // 方式1(Model实际是一个key-value集合，会传递到视图"spittles")
    @RequestMapping(method = RequestMethod.GET)
    public String spittles(Model model) {
        // 不指定key时，会根据对象类型推断为"spittleList"
        model.addAttribute(spittlerRepository.findSpittles(Long.MAX_VALUE, 20));
        return "spittles";
    }

    // 方式2(可以使用Map来代替Model)
    @RequestMapping(method = RequestMethod.GET)
    public String spittles(Map model) {
        model.addAttribute("spittleList", spittlerRepository.findSpittles(Long.MAX_VALUE, 20));
        return "spittles";
    }

    // 方式3（返回值会被放入模型中并传递到逻辑视图，key被推断为"spittleList"，逻辑视图名会根据请求路径推断为"spittles"）
    @RequestMapping(method = RequestMethod.GET)
    public List<Spittle> spittles() {
        return spittlerRepository.findSpittles(Long.MAX_VALUE, 20);
    }
}
```
* 接受请求的输入：
  * 处理查询参数(/spittles?max=238900&count=20):@RequestParam
```java
@Controller
@RequestMapping("/spittles")
public class SpittlerController {
    private static final String MAX_LONG_AS_STRING = Long.toString(Long.MAX_VALUE);
    private SpittlerRepository spittlerRepository;

    @Autowired
    public SpittlerController(SpittlerRepository spittlerRepository) {
        this.spittlerRepository = spittlerRepository;
    }
    
    @RequestMapping(method = RequestMethod.GET)
    public List<Spittle> spittles(@RequestParam("max") long max, 
                                  @RequestParam("count") long count) {
        return spittlerRepository.findSpittles(max, count);
    }

    // 加默认值(defaultValue属性为String，进行绑定时会自动转换为Long)
    @RequestMapping(method = RequestMethod.GET)
    public List<Spittle>  spittles(@RequestParam(value = "max", defaultValue = MAX_LONG_AS_STRING) long max,
                                   @RequestParam(value = "count", defaultValue = "20") long count) {
        return spittlerRepository.findSpittles(max, count);
    }
}
```
*  
  * 通过路径参数接受输入(/spittle/12345):@PathVariable
```java
@Controller
@RequestMapping("/spittles")
public class SpittlerController {
    private static final String MAX_LONG_AS_STRING = Long.toString(Long.MAX_VALUE);
    private SpittlerRepository spittlerRepository;

    @Autowired
    public SpittlerController(SpittlerRepository spittlerRepository) {
        this.spittlerRepository = spittlerRepository;
    }
    
    @RequestMapping(value = "/spittleId", method = RequestMethod.GET)
    public String spittle(@PathVariable("spittleId") long spittleId, Model model) {
        // 不指定key时，会根据对象类型推断为"spittle"
        model.addAttribute(spittlerRepository.findOne(spittleId));
        return "spittle";
    }
}
```
* 
   * 处理表单
```java
@Controller
@RequestMapping("/spittles")
public class SpittlerController {
    private static final String MAX_LONG_AS_STRING = Long.toString(Long.MAX_VALUE);
    private SpittlerRepository spittlerRepository;

    @Autowired
    public SpittlerController(SpittlerRepository spittlerRepository) {
        this.spittlerRepository = spittlerRepository;
    }
    
    // "redirect:"会被解析为重定向的规则。
    @RequestMapping(value = "/register", method = RequestMethod.POST)
    public String processRegistration(Spitter spitter) {
        spittlerRepository.save(spitter);
        return "redirect:/spitter/" + spitter.getUserName();
    }
}
```
* 
   * 校验表单：entity中使用java校验API的注解对元素进行注解（如@NotNull、@Max、@Min）；使用@Valid注解controller方法参数告诉Spring，需要确保这个参数满足校验限制（方法依然会被调用，但是会出现校验错误，需要程序处理）。
```java
@Controller
@RequestMapping("/spittles")
public class SpittlerController {
    private static final String MAX_LONG_AS_STRING = Long.toString(Long.MAX_VALUE);
    private SpittlerRepository spittlerRepository;

    @Autowired
    public SpittlerController(SpittlerRepository spittlerRepository) {
        this.spittlerRepository = spittlerRepository;
    }
    
    // "redirect:"会被解析为重定向的规则。
    @RequestMapping(value = "/register", method = RequestMethod.POST)
    public String processRegistration(@Valid Spitter spitter, Errors errors) {
        // 如果校验出现错误，则重新返回表单。
        if (errors.hasErrors()) {
            return "registerFrom";
        }
        
        spittlerRepository.save(spitter);
        return "redirect:/spitter/" + spitter.getUserName();
    }
}
```
* 渲染Web视图
   * JSP视图、Apache Tiles视图、Thymeleaf视图；
   * 根据不同的视图实现配置不同的视图解析器（JSP使用InternalResourceViewResolver，Thymeleaf使用ThymeleafViewResolver）；
   * 使用对应的标签库对模型数据与视图进行绑定。
* 关于SpringMVC的一些高级技术说明
   * SpringMVC的配置方案：
      * Servlet3.0可通过WebApplicationInitializer配置除DispatcherServlet和ContextLoaderListener外的Servlet和Listener；
      * Servlet3.0可通过AbstractAnnotationConfigDispatcherServletInitializer的customizeRegistration方法对DispatcherServlet进行额外配置；
      * 对于不支持Servlet3.0的容器可使用web.xml进行以上相应配置。
   * 处理multipart形式的数据（上传文件）：
      * 配置multipart解析器
         * StandardServletMultipartResolver（依赖Servlet3.0）：声明bean，并在配置DispatcherServlet时通过重载customizeRegistration方法配置multipart的细节（包括文件上传路径、大小限制等）；
         * CommonsMultiMultipartResolver（Spring内置）：可以不显示指定文件上传目录（默认为servelet的临时目录），在声明bean时就可以配置multipart的细节（包括文件上传路径、大小限制等）。
      * 处理multipart请求
         * 在表单中标识属性，告诉浏览器以multipart数据的形式来提交表单；
         * 将控制器的输入添加一个参数（可以为byte数组；也可以为Spring提供的MultipartFile接口，其功能更强大），并为其添加@RequestPart注解；
         * 对于文件的管理，可以将文件保存到Amazon S3中；
         * 在Serclet3.0中，也可以以javax.servlet.http.Part作为控制器方法的参数，代替MultipartFile。
```
@RequestMapping(value = "/register", method = RequestMethod.POST)
public String processRegistration(
                 @RequestPart("profilePicture") byte[] profilePicture,
                 @Valid Spitter spitter,
                 Errors errors) {
   ···
}
```
* 
   * 
      * 处理异常
         * Spring提供了多种方式将异常转换为响应（Servlet请求的输出都是一个Servlet响应）：
            * 特定的Spring异常将会自动映射为指定的HTTP状态码(一些异常会默认映射为HTTP状态码)；
            * 异常上可以添加@ResponseStatus注解，从而将其映射为某一个HTTP状态码；
            * 在方法上可以添加@ExceptionHandler注解，使其用来处理异常。
```
// 将异常自动映射为HTTP状态码；
@RequestMapping(value = "/spittleId", method = RequestMethod.GET)
public String spittle(@PathVariable("spittleId") long spittleId, Model model) {
    Spittle spittle = spittlerRepository.findOne(spittleId);
    if (spittle == null) {
        throw new SpittleNotFoundException();
    }
    model.addAttribute(spittle);
    return "spittle";
}

@ResponseStatus(value = HttpStatus.NOT_FOUND, reason = "Spitte Not Found")
public class SpittleNotFoundException extends RuntimeException{}

// 将同一个控制器中相同的异常处理代码提取，编写成一个异常处理的方法.@ExceptionHandler注解，当抛出DuplicateSpittleException异常时委托该方法处理，该方法返回参数与控制器方法返回参数类型相同。
@ExceptionHandler(DuplicateSpittleException.class)
public String handleDuplicateSpittle(){
    return "error/duplicate";
}
```
*
    *
        * 为控制器添加通知
```
// 为所有控制器定义一个异常处理方法，使用@ControllerAdvie注解的类，则这个类里包含@ExceptionHandler、@InitBinder、@ModelAttribute修饰的方法都会应用到控制器中带有@RequestMapping注解的方法上。
@ControllerAdvice
public class AppWideExceptionHandler {
    @ExceptionHandler(DuplicateSpittleException.class)
    public String handleDuplicateSpittle(){
        return "error/duplicate";
    }
}
```
*
    *
        * 跨重定向请求传递数据
```
// 使用URL模版进行数据传递
return "redirect:/spitter/{username}"

// 使用flash属性发送数据(Spring提供了RedirectAttributes设置flash属性，其提供了Model的所有功能，数据通过会话进行传递)
@RequestMapping(value = "/register", method = RequestMethod.POST)
public String processRegistration(
                 Spitter spitter,
                 RedirectAttributes model) {
   spitterRepository.save(spitter);
   model.addAttribute("username", spitter.getUsername());
   model.addFlashAttribute("spitter", spitter);
   return "redirect:/spitter/{username}"
}

@RequestMapping(value = "/{username}", method = RequestMethod.POST)
public String processRegistration(
                 @PathVariable String username,
                 Model model) {
   if (!model.containsAttribute("spitter")) {
        model.addAttribute(spitterRepository.findByUsername(username));
   }
   return "profile"
}
```
### Spring Web Flow
* 是一个web框架，适用于元素按照规定流程运行的成勋，是Spring MVC的扩展，支持基于流程的应用程序。
* 在Spring中配置Web Flow
   * 暂不支持在java中配置，只能在xml中进行配置；
   * 装配流程执行器->配置流程注册表->处理流程请求
```xml
<!-- 装配流程执行器 -->
<flow:flow-executor id = "flowExecutor" />

<!-- 配置流程注册表，负责创建和执行流程 -->
<flow:flow-registor id = "flowRegistor">
   <flow:flow-location id = "pizza" path = "/WEB-INF/flows/springpizza.xml" />
</flow:flow-registor>

<!-- 处理流程请求，负责加载流程定义 -->
<!-- FlowHandlerMapping装配注册表的引用，其会帮助DispatcherServlet将请求的url匹配到流程上（/pizza）-->
<bean class = "org.springframework.webflow.mvc.servlet.FlowHandlerMapping">
    <property name = "flowRegistor" ref = "flowRegistor" /> 
</bean>
<!-- FlowHandlerMapping仅仅是将请求定向到Spring Web Flow，FlowHandlerAdapter会负责响应 -->
<bean class = "org.springframework.webflow.mvc.servlet.FlowHandlerAdapter">
<property name = "flowExecutor" ref = "flowExecutor" />
</bean>
```
* 流程的组件
   * 流程主要由三个主要元素定义：状态（流程中事件发生的地点）、转移（连接这些地点的公路）、流程数据。
   * 状态包括：行为、决策、结束、子流程、视图（一般为JSP）；
   * 转移：除了结束状态以外，每个状态至少要有一个转移；
   * 流程数据：在流程中进行传递的数据。
* Spring Web Flow能够构建会话式应用程序（对用户进行指引、询问适当的问题并基于他们的响应将其引导到特定的页面），可以与Spring Security结合，保护对视图的访问。
### Spring Security


