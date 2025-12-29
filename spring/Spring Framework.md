# Spring

Spring是一个支持快速开发Java EE应用程序的框架

它提供了一系列底层容器和基础设施，并可以和大量常用的开源框架无缝集成



本节专注于介绍Spring Framework，它是最核心的Spring框架

Spring Framework主要包括几个模块：

- 支持IoC和AOP的容器
- 支持JDBC和ORM的数据访问模块
- 支持声明式事务的模块
- 支持基于Servlet的MVC开发
- 支持基于Reactive的Web开发
- 以及集成JMS、JavaMail、JMX、缓存等其他模块



---



# IoC 容器

**容器**：一种为某种特定组件的运行提供必要支持的一个软件环境。

例如：Tomcat就是一个Servlet容器，它可以为Servlet的运行提供运行环；Docker提供了必要的Linux环境以便运行一个特定的Linux进程



Spring的核心就是提供了一个**IoC容器**，它可以管理所有轻量级的JavaBean组件，提供的底层服务包括组件的生命周期管理、配置和组装服务、AOP支持，以及建立在AOP基础上的声明式事务服务等



## IoC 原理

**IoC (Inversion of Control, 控制反转)** 

```java
public class UserService {
    private HikariConfig config = new HikariConfig();
    private DataSource dataSource = new HikariDataSource(config);
	...
}
```

传统Java设计中，在对象的内部通过new创建组件对象实例，是程序**主动创建组件对象**

```java
// 引入 注入 机制
public class UserService {
    private DataSource dataSource;
    // 不自己创建 而是等待外部通过set()方法来注入DataSource
    public void setDataSource(DataSource dataSource) { this.dataSource = dataSource;}
}
```

引入IoC：所有组件不再由应用程序自己创建和配置，而是由IoC容器负责，对象只是**被动地接收依赖的组件对象**

IoC是一种设计思想，利于松耦合，优美编程



**DI (Dependency Injection, 依赖注入)**

创建组件对象的过程中，将对象的属性值（简单值，集合，对象）通过配置注入给该对象

- 是IoC思想的一种具体实现技术
- 通过外部（通常是容器）将依赖对象注入到使用它们的组件中
- 由IoC容器负责管理组件的生命周期
- 有三种主要形式：构造函数注入、Setter方法注入和接口注入



在Spring的IoC容器中，我们把所有组件统称为JavaBean，即配置一个组件就是配置一个Bean

Spring 提供的IoC容器的实现方式：

1. BeanFactory：IoC 容器基本实现，Spring 内部的使用接口，加载配置文件时候不会创建Bean，在获取时才创建
2. ApplicationContext：继承自 BeanFactory，提供了一些额外的功能，在加载配置文件时一次性创建所有的Bean



## 装配 Bean

```java
public class AService {
    public void doSomething() {
        System.out.println("AService is doing something.");
    }  
}
public class BService {
    private AService aService;
    
    public void setAService(AService aService) {
        this.aService = aService;
    }
    
    public void doSomething() {
        System.out.println("BService is doing something.");
        aService.doSomething();
    }
}
```



#### XML 方式

准备`application.xml`配置文件，指导Spring的IoC容器创建并组装Bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <!-- id Bean的唯一标识 -->
    <bean id="bService" class="org.example.service.BService">
        <!-- 通过<property name="..." ref="..." />注入另一个Bean -->
        <property name="aService" ref="aService" />
        <!-- 注入数据 而非其他Bean -->
        <!-- property name="username" value="root" / -->
    	<!-- property name="password" value="password" / -->
    </bean>

    <bean id="aService" class="org.example.service.AService" />
</beans>
<!-- Bean的顺序不重要，Spring根据依赖关系会自动正确初始化 -->

```

上述XML配置文件用Java代码写等同于：

```java
BService bService = new BService();
AService aService = new AService();
bService.setAService(aService);
```

区别在于 Spring容器是通过读取XML文件后使用**反射**完成的



创建一个**Spring的IoC容器实例**，加载配置文件，创建并装配好配置文件中指定的所有Bean

```java
ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
```

从Spring容器中“取出”装配好的Bean然后使用

```java
BService bService = context.getBean(BService.class);
bService.doSomething();
```



####  Annotation 方式

```java
// @Component 标记普通类为 Spring 管理的 Bean，可选名称，默认是小写开头的类名
// @Service @Controller @Repository是特例 若组件无法归类则使用 @Component
@Component
public class AService { ...}

@Component
public class BService {
    // 使用@Autowired就相当于把指定类型的Bean注入到指定的字段中
    @Autowired
    AService aService;
    ...
}

// @ComponentScan：告诉容器，自动搜索当前类所在的包以及子包，创建所有标注为@Component的Bean，并根据@Autowired进行装配
@Configuration
@ComponentScan
public class AppConfig {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        BService bService = context.getBean(BService.class);
        bService.doSomething();
    }
}
```



#### 条件装配

1. **Profile**
   Spring提供的表示运行环境的概念：开发native、测试test、生产production
   创建某个Bean时，Spring容器可以根据注解`@Profile`来决定是否创建
   在运行程序时，加上JVM参数`-Dspring.profiles.active=test`就可以指定以`test`环境启动
   	...active=test,master表示`test`环境，并使用`master`分支代码

   ```java
   @Configuration
   @ComponentScan
   public class AppConfig {
       @Bean
       @Profile("!test")
       ZoneId createZoneId() {
           return ZoneId.systemDefault();
       }
   
       @Bean
       @Profile({ "test", "master" }) // 满足test或master
       ZoneId createZoneIdForTest() {
           return ZoneId.of("America/New_York");
       }
   }
   ```

2. **Condition**

   ```java
   @Component
   @Conditional(OnSmtpEnvCondition.class)	// 满足 条件，才会创建此Bean
   public class SmtpMailService implements MailService {
       ...
   }
   public class OnSmtpEnvCondition implements Condition {
       public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
           return "true".equalsIgnoreCase(System.getenv("smtp"));
       }
   }
   // 更简单的 结合配置文件判断
   @Component
   @ConditionalOnProperty(name="app.smtp", havingValue="true")
   public class MailService {
       ...
   }
   ```

3. ...



## 定制 Bean

#### Scope

默认情况下，Spring容器初始化时创建Bean**单例(Singleton)**，容器关闭前销毁Bean

容器运行期间，调用`getBean(Class)`获取到的Bean是同一个实例



```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE) // @Scope("prototype")
public class AService { ...}
```

使用`@Scope`注解指定为Prototype（原型）Bean，每次调用`getBean(Class)`，容器都返回一个新的实例



#### 注入 List

现在有一系列接口相同，不同实现类的Bean

```java
// 例如 注册用户时，我们要对email、password和name这3个变量进行验证
public interface Validator {
    void validate(String email, String password, String name);
}
@Component
@Order(1)
public class EmailValidator implements Validator {
    public void validate(String email, String password, String name) {
        if (email...) { ...}
    }
}
@Component
@Order(2)
public class PasswordValidator  implements Validator { ...}
...
// 最后通过一个Validators作为入口进行验证
@Component
public class Validators {
    // Spring会自动把所有类型为Validator的Bean装配为一个List注入
	// 使用@Order注解 指定List中Bean的顺序
    @Autowired
    List<Validator> validators;

    public void validate(String email, String password, String name) {
        for (var validator : this.validators) {
            validator.validate(email, password, name);
        }
    }
}

```



#### 可选注入

默认情况下，如果Spring没有找到标记为`@Autowired`对应类型的Bean，会抛出`NoSuchBeanDefinitionException`异常

给`@Autowired`增加参数`required = false`：表示如果找到一个标记的类型的Bean，就注入，如果找不到，就忽略



#### 第三方Bean

某个Bean不在我们自己的package管理之内，就在`@Configuration`类中编写一个Java方法创建并返回它

给方法标记一个`@Bean`注解

```java
@Configuration
@ComponentScan
public class AppConfig {
    // @Bean: 标注方法，指示该方法返回的对象将被注册为 Spring Bean，交由 IoC 容器管理
    // 后续直接通过方法参数注入
    @Bean
    ZoneId createZoneId() {
        return ZoneId.of("Z");
    }
}
```



#### 初始化和销毁

不管是`@Component`还是`@Bean`注册的 Bean，在 Spring 容器中都会经历以下核心生命周期阶段：

```plaintext
实例化（创建对象）→ 依赖注入（给属性赋值）→ 初始化（自定义的初始化逻辑）→ 就绪（供程序使用）→ 销毁（容器关闭前的清理逻辑）
```

1. @Component
   @PostConstruct：标注在非静态的无参方法上，Spring 会在依赖注入完成后自动执行该方法
   @PreDestroy：标注在非静态的无参方法上，Spring 会在容器关闭前自动执行该方法

2. @Bean

   ```java
   // 配置类
   @Configuration
   public class AppConfig {
       // 注册Bean，并指定初始化方法和销毁方法
       @Bean(initMethod = "init", destroyMethod = "destroy")
       public UserService userService() {
           return new UserService();
       }
   }
   
   // 普通类（无需任何注解/接口，纯POJO）
   public class UserService {
   
       // 初始化方法（方法名可自定义，无参数、无返回值、public）
       public void init() { ...}
       // 销毁方法（方法名可自定义，无参数、无返回值、public）
       public void destroy() { ...}
   }
   ```



#### 别名

有时需要对一种类型的Bean创建多个实例，例如，同时连接多个数据库，就必须创建多个`DataSource`实例

```java
@Configuration
@ComponentScan
public class AppConfig {
    @Bean("z")
    ZoneId createZoneOfZ() {
        return ZoneId.of("Z");
    }

    @Bean("utc8")
    ZoneId createZoneOfUTC8() {
        return ZoneId.of("UTC+08:00");
    }
}
@Component
public class MailService {
	@Autowired(required = false)
	@Qualifier("z") // 指定注入名称为"z"的ZoneId
    // 也可使用 @Primary 注册Bean时指定一个为主要Bean，这样，注入时若未指定Bean的名字就注入主Bean
	ZoneId zoneId = ZoneId.systemDefault();
    ...
}
```



#### 懒加载

@Lazy: **只对单例 Bean 生效**，Prototype Bean 默认懒加载

1. 对`@Component`标注的组件类使用`@Lazy`：容器启动时不创建实例，首次调用时才初始化

2. 标注在 @Bean 方法上

3.  标注在依赖注入的参数上：解决循环依赖

   ```java
   // UserService依赖OrderService
   @Service
   public class UserService {
       private final OrderService orderService;
   
       public UserService(OrderService orderService) { this.orderService = orderService;}
   }
   
   // OrderService依赖UserService
   @Service
   public class OrderService {
       private final UserService userService;
   
       public OrderService(UserService userService) { this.userService = userService;}
   }
   // 此时容器启动报错：BeanCurrentlyInCreationException
   // 解决：构造器参数上添加@Lazy
   // @Lazy会让 Spring 注入一个代理对象（而非真实的 OrderService 实例），当首次调用 OrderService 的方法时，才会创建真实的实例，从而打破循环依赖的死锁
   ...
   public UserService(@Lazy OrderService orderService) {
       this.orderService = orderService;
   }
   ...
   ```

4. 标注在 @Autowired 属性上：延迟注入依赖
   对`@Autowired`标注的属性添加`@Lazy`，表示该依赖的 Bean 为懒加载（即使依赖的 Bean 本身没有标注`@Lazy`）

5. 



#### 使用FactoryBean

```java
@Component
public class ZoneIdFactoryBean implements FactoryBean<ZoneId> {
    String zone = "Z";

    // 当一个Bean实现了FactoryBean接口后，Spring会先实例化这个工厂，然后调用getObject()创建真正的
    @Override
    public ZoneId getObject() throws Exception {
        return ZoneId.of(zone);
    }
	// BeangetObjectType()可以指定创建的Bean的类型，因为指定类型不一定与实际类型一致，可以是接口或抽象类
    @Override
    public Class<?> getObjectType() {
        return ZoneId.class;
    }
}
```



## Resource

使用Spring容器时，可以对配置文件、资源文件等进行注入，方便程序读取

```java
@Component
public class AppService {
    @Value("classpath:/logo.txt")
    private Resource resource;

    private String logo;

    @PostConstruct
    public void init() throws IOException {
        try (var reader = new BufferedReader(
                new InputStreamReader(resource.getInputStream(), StandardCharsets.UTF_8))) {
            this.logo = reader.lines().collect(Collectors.joining("\n"));
        }
    }
}
```



#### 注入配置

配置文件最常用的配置方法是以`key=value`的形式写在`.properties`文件中

可以用`Resource`来读取位于classpath下的一个`app.properties`文件，但这样依然比较繁琐



Spring容器还提供的`@PropertySource`可以自动读取配置文件

```java
@Configuration
@ComponentScan
@PropertySource("app.properties") // 表示读取classpath的app.properties
public class AppConfig {
    @Value("${app.zone:Z}")	// 读取key为app.zone的value，但如果key不存在，就使用默认值Z
    String zoneId;

    @Bean
    ZoneId createZoneId() {
        return ZoneId.of(zoneId);
    }
}
```



项目应用：

```java
// 先通过一个简单的JavaBean持有所有的配置
@Component
public class SmtpConfig {
    @Value("${smtp.host}")
    private String host;

    @Value("${smtp.port:25}")
    private int port;

    public String getHost() {...}
    public int getPort() {...}
}
// 需要读取的地方，使用#{smtpConfig.host}注入
@Component
public class MailService {
    @Value("#{smtpConfig.host}")	// 从名称为smtpConfig的Bean读取host属性，即调用getHost()方法
    private String smtpHost;

    @Value("#{smtpConfig.port}")
    private int smtpPort;
}
```



## IoC 加载流程

1. 创建 BeanFactory 容器
2. 解析配置信息，并将配置信息封装成 BeanDefinition 对象，再将其存入BeanFactory 
3. 执行工厂的后置处理器 BeanFactoryPostProcessor
4. 执行后置处理 BeanPostProcessor，注意: 这里不要和上一步搞混了，一个工厂后置处理器，一个是 Bean 后置处理器
5. 实例化 Bean 对象，这里的对象信息来源就是BeanFactory 中的 BeanDefinition 对象，这里只是单纯的实例化，没有参数赋值
6. Bean 初始化过程
7. 得到一个完整的 Bean 对象放入单例池中，IOC 容器构建结束

<img src="./ioc.png" style="zoom:67%;float:left; margin:10px" />



---



# AOP

**AOP**（Aspect Oriented Programming）：⾯向切⾯编程，通过**预编译**和**运行期间动态代理**来实现程序功能的统一维护的一种技术

AOP思想是OOP(面向对象)的延续：
	OOP中程序的基本单元是 class
	AOP中的基本单元是 Aspect(切面)



举个栗子：
现在有一个业务组件 BookService 其中有业务方法 createBook updateBook deleteBook
对每个业务方法，除了业务逻辑，还需要安全检查、日志记录和事务处理

```java
public class BookService {
    public void createBook(Book book) {
        securityCheck();
        Transaction tx = startTransaction();
        try {
            // 核心业务逻辑
            tx.commit();
        } catch (RuntimeException e) {
            tx.rollback();
            throw e;
        }
        log("created book: " + book);
    }
    ...
}
```

对于安全检查、日志、事务等代码，它们会重复出现在每个业务方法中

虽然可以使用设计模式比如 Proxy 进行模块抽取，但还是麻烦：须先抽取接口，然后，针对每个方法实现Proxy



引入AOP

把权限检查、日志、事务视作切面（Aspect），然后，以某种自动化的方式，让框架把切面织入到核心逻辑中，实现**Proxy模式**

1. 核心逻辑，即BookService
2. 切面逻辑：
   1. 权限检查的Aspect
   2. 日志的Aspect
   3. 事务的Aspect



## AOP 原理

把切面织入到核心逻辑：

1. 编译期：在编译时，由编译器把切面调用编译进字节码，这种方式需要定义新的关键字并扩展编译器，AspectJ就扩展了Java编译器，使用关键字aspect来实现织入
2. 类加载器：在目标类被装载到JVM时，通过一个特殊的类加载器，对目标类的字节码重新“增强”
3. 运行期：目标对象和切面都是普通Java类，通过JVM的动态代理功能或者第三方库实现运行期动态织入



## 装配 AOP

maven引入Spring对AOP的支持：org.springframework:spring-aspects:6.0.0

```java
@Aspect
@Component	// 本身也是一个Bean
public class loggingAspect {
    // 在执行AService的每个方法前后执行:
    @Around("execution(public * org.example.service.AService.*(..))")
    public Object doAroundA(ProceedingJoinPoint pjp) throws Throwable {
        System.err.println("[Around] start " + pjp.getSignature());
        Object retVal = pjp.proceed();
        System.err.println("[Around] done " + pjp.getSignature());
        return retVal;
    }
    // @Before After AfterReturning AfterThrowing
}
@ComponentScan
@EnableAspectJAutoProxy	
// IoC容器自动查找带有@Aspect的Bean，根据每个方法的@Before、@Around等注解把AOP注入到特定的Bean中
// Spring容器启动时自动创建注入了Aspect的子类，取代了原始的组件类（原始的组件实例作为内部变量隐藏在xxxAopProxy中）
// Spring对接口类型使用JDK动态代理，对普通类使用CGLIB创建子类。如果一个Bean的class是final，Spring将无法为其创建子类
public class AppTest extends TestCase {
    @Test
    public void test99() {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppTest.class);
        BService bService = context.getBean(BService.class);
        bService.doSomething();
    }
}
```



#### 拦截器类型

- @Before：这种拦截器先执行拦截代码，再执行目标代码。如果拦截器抛异常，那么目标代码就不执行了
- @After：这种拦截器先执行目标代码，再执行拦截器代码。无论目标代码是否抛异常，拦截器代码都会执行
- @AfterReturning：和@After不同的是，只有当目标代码正常返回时，才执行拦截器代码
- @AfterThrowing：和@After不同的是，只有当目标代码抛出了异常时，才执行拦截器代码
- @Around：能完全控制目标代码是否执行，并可以在执行前后、抛异常后执行任意拦截代码，可以说是包含了上面所有功能

```java
	@Around 前
try {
    @Before
    	业务方法
    @AfterReturning
} catch (Exception e) {
    @AfterThrowing
} finally {
    @After
}
	@Around 后
```



#### 注解装配

前面是用的AspectJ的注解，并配合一个复杂的`execution(* xxx.Xyz.*(..))`语法来定义应该如何装配AOP

```java
// 定义注解 实现监控应用程序的性能
@Target(METHOD)
@Retention(RUNTIME)
public @interface WelcomeAnnotation {
    String value() default "";
}
// 切面
@Aspect
@Component
public class WelcomeAspect {
    // 直接在Advice上定义Pointcut（最简，适用于单一切入点） 切入点逻辑无法复用
    @Around("@annotation(welcomeAnnotation)")
    public void welcome(ProceedingJoinPoint joinPoint, WelcomeAnnotation welcomeAnnotation) throws Throwable {
        String name = welcomeAnnotation.value();
        System.err.println("welcome: " + name + ", signature: " + joinPoint.getSignature());
        joinPoint.proceed();
        System.err.println("byebye: " + name);
    }
    
    // 通过@Pointcut注解定义可复用的切入点（推荐，适用于多Advice复用）
    @Pointcut("@annotation(com.example.annotation.WelcomeAnnotation)") // 注意：替换为实际的注解全类名
    public void welcomeAnnotationPointcut() {}
    @Around("welcomeAnnotationPointcut() && @annotation(welcomeAnnotation)")
    public Object monitorPerformanceReusable(ProceedingJoinPoint joinPoint, WelcomeAnnotation welcomeAnnotation) throws Throwable {
        // 逻辑和上面的direct方法一致，此处省略（可直接复用上述逻辑）
        return monitorPerformanceDirect(joinPoint, welcomeAnnotation);
    }
    
}

@Component
public class BService {
    @WelcomeAnnotation("zhangsan")
    public void doSomething() {
        System.out.println("BService is doing something.");
        aService.doSomething();
    }
}
// 有了@xxxAnnotation，再配合xxxAspect，任何Bean，只要方法标注了@xxxAnnotation，就可以自动实现xxxx
```



#### 启用AOP导致的NPE

```java
@Component
public class UserService {
    public final ZoneId zoneId = ZoneId.systemDefault();

    public UserService() {}
    public ZoneId getZoneId() { return zoneId;}
    public final ZoneId getFinalZoneId() { return zoneId;}
}

@Component
public class MailService {
    @Autowired
    UserService userService;

    public void sendMail() {
        ZoneId zoneId = userService.zoneId;
        System.out.println(zoneId);
    }
}

@Aspect
@Component
public class LoggingAspect {
    @Before("execution(public * com..*.UserService.*(..))")
    public void doAccessCheck() { ...}
}

@Configuration
@ComponentScan
@EnableAspectJAutoProxy
public class AppConfig {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        MailService mailService = context.getBean(MailService.class);
        System.out.println(mailService.sendMail());
    }
}
// 启用AOP 报错: MailService.sendMail.System.out.println(zoneId) null
// 因为织入了切面 注入到MailService中的UserService实例是proxy
// public final ZoneId zoneId = ZoneId.systemDefault(); 未执行
// 因为 proxy 的目的是代理方法 没初始化proxy的成员变量
// Spring 通过 CGLIB 动态创建的代理类 的构造方法中 不会调用super()

// 修复：不要直接访问UserService的字段 用getter
// 注意 代理类无法覆写final方法
```



---



# database

Spring对数据库访问的简化：

- 提供了简化的访问JDBC的模板类，不必手动释放资源
- 提供了一个统一的DAO类以实现Data Access Object模式
- 把`SQLException`封装为`DataAccessException`，这个异常是一个`RuntimeException`，并且让我们能区分SQL异常的原因，例如，`DuplicateKeyException`表示违反了一个唯一约束
- 能方便地集成Hibernate、JPA和MyBatis这些数据库访问框架



## JDBC

过去的Java程序使用JDBC接口访问关系数据库：

- 创建全局`DataSource`实例，表示数据库连接池
- 在需要读写数据库的方法内部，按如下步骤访问数据库：
  - 从全局`DataSource`实例获取`Connection`实例
  - 通过`Connection`实例创建`PreparedStatement`实例
  - 执行SQL语句，如果是查询，则通过`ResultSet`读取结果集，如果是修改，则获得`int`结果

关键是 释放资源 和 事务处理



在Spring使用JDBC

```java
@Configuration
@ComponentScan
@PropertySource("jdbc.properties")
public class JDBCConfig {
    @Value("${jdbc.url}")
    String jdbcUrl;
    @Value("${jdbc.username}")
    String jdbcUsername;
    @Value("${jdbc.password}")
    String jdbcPassword;

    @Bean
    DataSource createDataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl(jdbcUrl);
        config.setUsername(jdbcUsername);
        config.setPassword(jdbcPassword);
        config.addDataSourceProperty("autoCommit", "true");
        config.addDataSourceProperty("connectionTimeout", "5");
        config.addDataSourceProperty("idleTimeout", "60");
        return new HikariDataSource(config);
    }

    @Bean
    JdbcTemplate createJdbcTemplate(@Autowired DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
}
// 通过HSQLDB自带的工具来初始化数据库表，这里写一个Bean，在Spring容器启动时自动创建一个users表
@Component
public class DatabaseInitializer {
    @Autowired
    JdbcTemplate jdbcTemplate;

    @PostConstruct
    public void init() {
        jdbcTemplate.update("CREATE TABLE IF NOT EXISTS users (" //
                + "id BIGINT IDENTITY NOT NULL PRIMARY KEY, " //
                + "email VARCHAR(100) NOT NULL, " //
                + "password VARCHAR(100) NOT NULL, " //
                + "username VARCHAR(100) NOT NULL, " //
                + "UNIQUE (email))");
    }
}
// 使用：只需要在需要访问数据库的Bean中，注入JdbcTemplate即可
@EnableAspectJAutoProxy
public class AppTest extends TestCase {
	@Test
    public void testJDBC() {
        ApplicationContext context = new AnnotationConfigApplicationContext(JDBCConfig.class);
        JDBCService a = context.getBean(JDBCService.class);
        a.addUser("zhangsan","123456", "666");
        a.addUser("lisi","123456", "777");

        System.out.println(a.getUserByEmail("777"));
    }
}
```

[HSQLDB](https://hsqldb.org/)数据库，是一个用Java编写的关系数据库，可以以内存模式或者文件模式运行，本身只有一个jar包，非常适合演示代码或者测试代码

```properties
# 数据库文件名为testdb:
jdbc.url=jdbc:hsqldb:file:testdb
# Hsqldb默认的用户名是sa，口令是空字符串:
jdbc.username=sa
jdbc.password=
```



Spring提供的**`JdbcTemplate`**采用**Template模式**，提供了一系列以**回调**为特点的工具方法，目的是避免繁琐的`try...catch`语句

用法：

- 针对简单查询，优选`query()`和`queryForObject()`，因为只需提供SQL语句、参数和`RowMapper`

  - 对于查询，主要通过`RowMapper`实现了JDBC结果集到Java对象的转换

  - ```java
    public User getUserByEmail(String email) {
        // 传入SQL，参数和RowMapper实例:
        return jdbcTemplate.queryForObject("SELECT * FROM users WHERE email = ?",
                (ResultSet rs, int rowNum) -> {
                    // 将ResultSet的当前行映射为一个JavaBean:
                    return new User( // new User object:
                            rs.getLong("id"), // id
                            rs.getString("email"), // email
                            rs.getString("password"), // password
                            rs.getString("name")); // name
                },
                email);
    }
    // 返回多行记录
    public List<User> getUsers(int pageIndex) {
        int limit = 100;
        int offset = limit * (pageIndex - 1);
        // 数据库表的结构恰好和JavaBean的属性名称一致，Spring提供的BeanPropertyRowMapper直接把一行记录按列名转换为JavaBean
        return jdbcTemplate.query("SELECT * FROM users LIMIT ? OFFSET ?",
                new BeanPropertyRowMapper<>(User.class),
                limit, offset);
    }
    ```

- 针对更新操作，优选`update()`，因为只需提供SQL语句和参数

  - ```java
    public void updateUser(User user) {
        // 传入SQL，SQL参数，返回更新的行数:
        if (1 != jdbcTemplate.update("UPDATE users SET name = ? WHERE id = ?", user.getName(), user.getId())) {
            throw new RuntimeException("User not found by id");
        }
    }
    // 某一列是自增列，这时我们需要获取插入后的自增值：使用 JdbcTemplate 提供的 KeyHolder
    public User register(String email, String password, String name) {
        // 创建一个KeyHolder:
        KeyHolder holder = new GeneratedKeyHolder();
        if (1 != jdbcTemplate.update(
            // 参数1:PreparedStatementCreator
            (conn) -> {
                // 创建PreparedStatement时，必须指定RETURN_GENERATED_KEYS:
                var ps = conn.prepareStatement("INSERT INTO users(email, password, name) VALUES(?, ?, ?)",
                        Statement.RETURN_GENERATED_KEYS);
                ps.setObject(1, email);
                ps.setObject(2, password);
                ps.setObject(3, name);
                return ps;
            },
            // 参数2:KeyHolder
            holder)
        ) {
            throw new RuntimeException("Insert failed.");
        }
        // 从KeyHolder中获取返回的自增值:
        return new User(holder.getKey().longValue(), email, password, name);
    }
    ```

- 任何复杂的操作，最终也可以通过`execute(ConnectionCallback)`实现，因为拿到`Connection`就可以做任何JDBC操作

  - ```java
    // T execute(ConnectionCallback<T> action)
    public User getUserById(long id) {
        // 注意传入的是ConnectionCallback:
        return jdbcTemplate.execute((Connection conn) -> {
            // 可以直接使用conn实例，不要释放它，回调结束后JdbcTemplate自动释放:
            // 在内部手动创建的PreparedStatement、ResultSet必须用try(...)释放:
            try (var ps = conn.prepareStatement("SELECT * FROM users WHERE id = ?")) {
                ps.setObject(1, id);
                try (var rs = ps.executeQuery()) {
                    if (rs.next()) {
                        return new User( // new User object:
                                rs.getLong("id"), // id
                                rs.getString("email"), // email
                                rs.getString("password"), // password
                                rs.getString("name")); // name
                    }
                    throw new RuntimeException("user not found by id.");
                }
            }
        });
    }
    ```

    

---



## 声明式事务

```java
// 手写事务
TransactionStatus tx = null;
try {
    // 开启事务:
    tx = txManager.getTransaction(new DefaultTransactionDefinition());
    // 相关JDBC操作:
    jdbcTemplate.update("...");
    jdbcTemplate.update("...");
    // 提交事务:
    txManager.commit(tx);
} catch (RuntimeException e) {
    // 回滚事务:
    txManager.rollback(tx);
    throw e;
}
```

在Spring中操作事务，没必要手写JDBC事务，可以使用Spring提供的高级接口来操作事务

Spring提供的事务管理器：`PlatformTransactionManager`，负责管理所有事务
而事务用`TransactionStatus`表示



之所以抽象出`PlatformTransactionManager`和`TransactionStatus`，是因为JavaEE除了提供JDBC事务外，它还支持分布式事务JTA（Java Transaction API）
	分布式事务是指多个数据源（比如多个数据库，多个消息系统）要在分布式环境下实现事务的时候，应该怎么实现

Spring为了同时支持JDBC和JTA两种事务模型，就抽象出`PlatformTransactionManager`



``` java
// 这里只需要JDBC事务
@Configuration
@ComponentScan
@EnableTransactionManagement // 启用声明式事务，避免繁琐编程
@PropertySource("jdbc.properties")
public class AppConfig {
    ...
    // 定义一个PlatformTransactionManager对应的Bean
    @Bean
    PlatformTransactionManager createTxManager(@Autowired DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
    
}
// 然后，对需要事务支持的方法，加一个@Transactional注解
@Component
public class UserService {
    // 此public方法自动具有事务支持:
    @Transactional
    public User register(String email, String password, String name) { ...}
}
// 或者直接在Bean的class处注解@Transactional，表示所有public方法都具有事务支持

// @Transactional 属性
// readOnly 只读事务 高效
// timeout 超时等待秒数 默认-1表示永不超时
// propagation 设置传播行为
// isolation 设置底层数据库的事务隔离级别
// rollbackFor rollbackForClassName 设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的编译期异常时，则事务回滚
// noRollbackFor noRollbackForClassName 设置不需要进行回滚的异常类数组
```



Spring对一个声明式事务的方法，通过AOP代理，即通过自动创建Bean的Proxy实现事务支持

```java
public class UserService$$EnhancerBySpringCGLIB extends UserService {
    UserService target = ...
    PlatformTransactionManager txManager = ...

    public User register(String email, String password, String name) {
        TransactionStatus tx = null;
        try {
            tx = txManager.getTransaction(new DefaultTransactionDefinition());
            target.register(email, password, name);
            txManager.commit(tx);
        } catch (RuntimeException e) {
            txManager.rollback(tx);
            throw e;
        }
    }
    ...
}
```



**回滚事务**

默认：在一个事务方法中，如果程序判断需要回滚事务，只需抛出`RuntimeException`
	默认情况下 Spring 不会在业务方法出现编译期异常时执行回滚操作 只会在RuntimeException回滚

```java
@Transactional
public buyProducts(long productId, int num) {
    ...
    if (store < num) {
        // 库存不够，购买失败:
        throw new IllegalArgumentException("No enough products");
    }
    ...
}
```

针对Checked Exception回滚事务：在`@Transactional`注解中写出来

```java
// 抛出RuntimeException或IOException时，事务将回滚
@Transactional(rollbackFor = {RuntimeException.class, IOException.class})
public buyProducts(long productId, int num) throws IOException { ...}
```



**事务边界**

```java
@Component
public class UserService {
    @Transactional
    public User register(String email, String password, String name) { // 事务开始
       ...
    } // 事务结束
}
```

现实中的事务边界往往更加复杂



**事务传播**

事务传播指多个事务方法在嵌套调用时的事务控制方式

现有某功能入口AController，其中调用BService的某事务方法bMethod，在bMethod内部又调用了CService的事务方法cMethod

1. bMethod 和 cMethod 都由 `@Transactional` 注解

2. 我们希望 BService.bMethod 和 CService.cMethod 在一个事务中进行



Spring的声明式事务为事务传播定义了几个级别，默认传播级别就是REQUIRED，它的意思是，如果当前没有事务，就创建一个新事务，如果当前有事务，就加入到当前事务中执行

- BService.bMethod 在 AController 中进行，因为 AController 没有事务，因此 BService.bMethod 自动创建一个事务
- BService.bMethod 方法内部调用 CService.cMethod 时，cMethod 检测到当前已有事务，直接加入到当前事务中执行

- 整个业务流程的事务边界：只有一个事务：BService.bMethod



其他传播级别：

1. `SUPPORTS`：表示如果有事务，就加入到当前事务，如果没有，那也不开启事务执行。这种传播级别可用于查询方法，因为SELECT语句既可以在事务内执行，也可以不需要事务
2. `MANDATORY`：表示必须要存在当前事务并加入执行，否则将抛出异常。这种传播级别可用于核心更新逻辑，比如用户余额变更，它总是被其他事务方法调用，不能直接由非事务方法调用
3. `REQUIRES_NEW`：表示不管当前有没有事务，都必须开启一个新的事务执行。如果当前已经有事务，那么当前事务会挂起，等新事务完成后，再恢复执行
4. `NOT_SUPPORTED`：表示不支持事务，如果当前有事务，那么当前事务会挂起，等这个方法执行完成后，再恢复执行
5. `NEVER`：和`NOT_SUPPORTED`相比，它不但不支持事务，而且在监测到当前有事务时，会抛出异常拒绝执行
6. `NESTED`：表示如果当前有事务，则开启一个嵌套级别事务，如果当前没有事务，则开启一个新事务

定义事务的传播级别也是写在注解里 `@Transactional(propagation = Propagation.REQUIRES_NEW)`



Spring 事务传播的实现：**`ThreadLocal`**

Spring总是把JDBC相关的`Connection`和`TransactionStatus`实例绑定到`ThreadLocal`
	如果一个事务方法从`ThreadLocal`未取到事务，那么它会打开一个新的JDBC连接，同时开启一个新的事务，否则，它就直接使用从`ThreadLocal`获取的JDBC连接以及`TransactionStatus`

因此，事务能正确传播的前提是，方法调用是在一个线程内
	即事务只能在当前线程传播，无法跨线程传播



---



## DAO

Spring提供了一个`JdbcDaoSupport`类，用于简化DAO的实现

```java
public abstract class JdbcDaoSupport extends DaoSupport {
	// 核心：持有一个JdbcTemplate
    private JdbcTemplate jdbcTemplate;

    public final void setJdbcTemplate(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
        initTemplateConfig();
    }
    public final JdbcTemplate getJdbcTemplate() { return this.jdbcTemplate;}
    ...
}
// 因为 JdbcDaoSupport 的 jdbcTemplate 没有自动注入，所以子类需要注入一个到 JdbcDaoSupport 
public abstract class AbstractDao<T> extends JdbcDaoSupport {
    private String table;
    private Class<T> entityClass;
    private RowMapper<T> rowMapper;

    public AbstractDao() {
        // 获取当前类型的泛型类型:
        this.entityClass = getParameterizedType();
        this.table = this.entityClass.getSimpleName().toLowerCase() + "s";
        this.rowMapper = new BeanPropertyRowMapper<>(entityClass);
    }

    public T getById(long id) {
        return getJdbcTemplate().queryForObject("SELECT * FROM " + table + " WHERE id = ?", this.rowMapper, id);
    }

    public List<T> getAll(int pageIndex) {
        int limit = 100;
        int offset = limit * (pageIndex - 1);
        return getJdbcTemplate().query("SELECT * FROM " + table + " LIMIT ? OFFSET ?",
                new Object[] { limit, offset },
                this.rowMapper);
    }

    public void deleteById(long id) {
        getJdbcTemplate().update("DELETE FROM " + table + " WHERE id = ?", id);
    }
    ...
}
@Component
@Transactional
public class UserDao extends AbstractDao<User> {
    // 已经有了:
    // User getById(long)
    // List<User> getAll(int)
    // void deleteById(long)
}
```



---



## Hibernate

`JdbcTemplate`的方法`List<T> query(String, RowMapper, Object...)`
	使用`RowMapper`把`ResultSet`的一行记录映射为Java Bean

这种把关系数据库的表记录映射为Java对象的过程就是**ORM**：Object-Relational Mapping
	ORM既可以把记录转换成Java对象，也可以把Java对象转换为行记录
	ORM框架使用**Proxy模式**，跟踪Java Bean的修改，便在`update()`操作中更新必要的属性

ORM框架通常提供了缓存
	一级缓存是指在一个Session范围内的缓存，常见的情景是根据主键查询时，两次查询可以返回同一实例
	二级缓存是指跨Session的缓存，一般默认关闭，需要手动配置，它增加了数据的不一致性

使用`JdbcTemplate`配合`RowMapper`可以看作是最原始的ORM
	要实现更自动化的ORM，可以使用熟的ORM框架，例如[Hibernate](https://hibernate.org/)



```java
@Configuration
@ComponentScan
@EnableTransactionManagement
@PropertySource("jdbc.properties")
public class AppConfig {
    @Bean
    DataSource createDataSource() {
        ...
    }
    // 启用Hibernate 必须的Bean
    // 这是一个FactoryBean 会再自动创建一个SessionFactory
    // Hibernate中，Session是封装了一个JDBC Connection的实例
    // SessionFactory是封装了JDBC DataSource的实例，即SessionFactory持有连接池
    // 每次需要操作数据库的时候，SessionFactory创建一个新的Session，相当于从连接池获取到一个新的Connection
    @Bean
    LocalSessionFactoryBean createSessionFactory(@Autowired DataSource dataSource) {
        // 首先用Properties持有Hibernate初始化SessionFactory时用到的所有设置
        var props = new Properties();
        props.setProperty("hibernate.hbm2ddl.auto", "update"); // 自动创建数据库的表结构 生产环境勿用
        props.setProperty("hibernate.dialect", "org.hibernate.dialect.HSQLDialect");	// 使用HSQLDB数据库
        props.setProperty("hibernate.show_sql", "true");	// 让Hibernate打印执行的SQL 便于调试
        var sessionFactoryBean = new LocalSessionFactoryBean();
        sessionFactoryBean.setDataSource(dataSource);
        // 扫描指定的package获取所有entity class，自动找出能映射为数据库表记录的JavaBean
        sessionFactoryBean.setPackagesToScan("com.itranswarp.learnjava.entity");
        sessionFactoryBean.setHibernateProperties(props);
        return sessionFactoryBean;
    }
    // 创建HibernateTransactionManager
    @Bean
    PlatformTransactionManager createTxManager(@Autowired SessionFactory sessionFactory) {
        return new HibernateTransactionManager(sessionFactory);
    }
}

// 将数据库表结构映射为Java对象
@Entity
@Table(name="users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false, updatable = false)
    public Long getId() { ... }

    @Column(nullable = false, unique = true, length = 100)
    public String getEmail() { ... }

    @Column(nullable = false, length = 100)
    public String getPassword() { ... }

    @Column(nullable = false, length = 100)
    public String getName() { ... }

    @Column(nullable = false, updatable = false)
    public Long getCreatedAt() { ... }
    // “虚拟”的属性。因为getCreatedDateTime()是计算得出的属性，而不是从数据库表读出的值
    @Transient
    public ZonedDateTime getCreatedDateTime() {
        return Instant.ofEpochMilli(this.createdAt).atZone(ZoneId.systemDefault());
    }
	// 在我们将一个JavaBean持久化到数据库之前（即执行INSERT语句），Hibernate会先执行该方法
    @PrePersist
    public void preInsert() {
        setCreatedAt(System.currentTimeMillis());
    }
}
```

```java
// 使用
@Component
@Transactional
public class UserService {
    @Autowired
    SessionFactory sessionFactory;
    
    // Insert 持久化一个User实例 只需调用persist()
    public User register(String email, String password, String name) {
        // 创建一个User对象:
        User user = new User();
        // 设置好各个属性:
        user.setEmail(email);
        user.setPassword(password);
        user.setName(name);
        // 不要设置id，因为使用了自增主键
        // 保存到数据库:
        sessionFactory.getCurrentSession().persist(user);
        // 现在已经自动获得了id:
        System.out.println(user.getId());
        return user;
    }
    // Delete Hibernate总是用id来删除记录
    public boolean deleteUser(Long id) {
        User user = sessionFactory.getCurrentSession().byId(User.class).load(id);
        if (user != null) {
            sessionFactory.getCurrentSession().remove(user);
            return true;
        }
        return false;
    }
    // Update 先更新User的指定属性，然后调用merge()方法
    public void updateUser(Long id, String name) {
        User user = sessionFactory.getCurrentSession().byId(User.class).load(id);
        user.setName(name);
        sessionFactory.getCurrentSession().merge(user);
    }
    // Select
    // 1. Hibernate内置的HQL查询：
    List<User> list = sessionFactory.getCurrentSession()
        .createQuery("from User u where u.email = ?1 and u.password = ?2", User.class)
        .setParameter(1, email).setParameter(2, password)
        .list();
    // 2. NamedQuery：给查询起个名字，然后保存在注解中
    @NamedQueries(
        @NamedQuery(
            // 查询名称:
            name = "login",
            // 查询语句:
            query = "SELECT u FROM User u WHERE u.email = :e AND u.password = :pwd"
        )
    )
    @Entity
    public class User extends AbstractEntity {
        public User login(String email, String password) {
        List<User> list = sessionFactory.getCurrentSession()
            .createNamedQuery("login", User.class) // 创建NamedQuery
            .setParameter("e", email) // 绑定e参数
            .setParameter("pwd", password) // 绑定pwd参数
            .list();
        return list.isEmpty() ? null : list.get(0);
    }
}
```



---



## JPA

| JDBC       | Hibernate      | JPA                  |
| ---------- | -------------- | -------------------- |
| DataSource | SessionFactory | EntityManagerFactory |
| Connection | Session        | EntityManager        |

JPA是JavaEE的一个ORM标准，他只是接口，还需要一个实现产品

使用JPA时可以选择Hibernate作为底层实现，或者其他JPA提供方

```java
@Configuration
@ComponentScan
@EnableTransactionManagement
@PropertySource("jdbc.properties")
public class AppConfig {
    @Bean
    DataSource createDataSource() { ... }
    
    @Bean
    public LocalContainerEntityManagerFactoryBean createEntityManagerFactory(@Autowired DataSource dataSource) {
        var emFactory = new LocalContainerEntityManagerFactoryBean();
        // 注入DataSource:
        emFactory.setDataSource(dataSource);
        // 扫描指定的package获取所有entity class:
        emFactory.setPackagesToScan(AbstractEntity.class.getPackageName());
        // 使用Hibernate作为JPA实现:
        emFactory.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        // 其他配置项:
        var props = new Properties();
        props.setProperty("hibernate.hbm2ddl.auto", "update"); // 生产环境不要使用
        props.setProperty("hibernate.dialect", "org.hibernate.dialect.HSQLDialect");
        props.setProperty("hibernate.show_sql", "true");
        emFactory.setJpaProperties(props);
        return emFactory;
    }
    // 实例化一个JpaTransactionManager，以实现声明式事务
	@Bean
    PlatformTransactionManager createTxManager(@Autowired EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}
```

Entity Bean的配置和上一节完全相同

```java
@Component
@Transactional
public class UserService {
    // 此处自动注入代理EntityManagerProxy ，该代理会在必要的时候自动打开EntityManager
    // 多线程引用的EntityManager虽然是同一个代理类，但该代理类内部针对不同线程会创建不同的EntityManager实例
    // 标注了@PersistenceContext的EntityManager可以被多线程安全地共享
    @PersistenceContext
    EntityManager em;
    
    public User getUserById(long id) {
        User user = this.em.find(User.class, id);
        if (user == null) {
            throw new RuntimeException("User not found by id: " + id);
        }
        return user;
    }
    
    public User fetchUserByEmail(String email) {
        // JPQL查询:
        TypedQuery<User> query = em.createQuery("SELECT u FROM User u WHERE u.email = :e", User.class);
        query.setParameter("e", email);
        List<User> list = query.getResultList();
        if (list.isEmpty()) {
            return null;
        }
        return list.get(0);
    }
    
    // NamedQuery
    ...
        
    // 对数据库进行增删改的操作，可以分别使用persist()、remove()和merge()方法，参数均为Entity Bean本身
    ...
}
```



---



## MyBatis

全自动ORM：Hibernate

- 完全的对象关系映射：支持实体类与数据库表的自动映射（注解 / XML 配置），UserProxy保存Hibernate的当前Session
  - 引入了Attached/Detached状态，表示当前此Java Bean到底是在Session的范围内，还是脱离了Session变成了一个“游离”对象
- 兼容多种数据库，它使用HQL或JPQL查询，经过一道转换，变成特定数据库的SQL
- 一级缓存（Session）+ 二级缓存（SessionFactory）：提升查询性能
- 事务管理：与 Spring 无缝集成，支持声明式事务



MyBatis：半自动化ORM框架

| JDBC       | Hibernate      | JPA                  | MyBatis           |
| ---------- | -------------- | -------------------- | ----------------- |
| DataSource | SessionFactory | EntityManagerFactory | SqlSessionFactory |
| Connection | Session        | EntityManager        | SqlSession        |

```java
@Configuration
@ComponentScan
@EnableTransactionManagement
@PropertySource("jdbc.properties")
public class AppConfig {
    @Bean
    DataSource createDataSource() { ... }
    @Bean
    SqlSessionFactoryBean createSqlSessionFactoryBean(@Autowired DataSource dataSource) {
        var sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        return sqlSessionFactoryBean;
    }
    // 使用Spring管理的声明式事务
    @Bean
    PlatformTransactionManager createTxManager(@Autowired DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

和Hibernate不同的是，MyBatis使用**Mapper**来实现映射，而且Mapper必须是接口。我们以`User`类为例，在`User`类和`users`表之间映射的`UserMapper`编写如下：

```java
public interface UserMapper {
	@Select("SELECT * FROM users WHERE id = #{id}")
	User getById(@Param("id") long id);
    
    @Select("SELECT * FROM users LIMIT #{offset}, #{maxResults}")
	List<User> getAll(@Param("offset") int offset, @Param("maxResults") int maxResults);
    // MyBatis执行查询后，将根据方法的返回类型自动把ResultSet的每一行转换为User实例，转换规则当然是按列名和属性名对应
    // 如果列名和属性名不同，最简单的方式是编写SELECT语句的别名
    // SELECT id, name, email, created_time AS createdAt FROM users
    
    // 插入 半自动化ORM 须写出完整的INSERT语句
    // @Options: users表的id是自增主键，在SQL中不传入id，但希望获取插入后的主键
    // keyProperty和keyColumn分别指出JavaBean的属性和数据库的主键列名
    @Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "id")
    @Insert("INSERT INTO users (email, password, name, createdAt) VALUES (#{user.email}, #{user.password}, #{user.name}, #{user.createdAt})")
	void insert(@Param("user") User user);
    
    // 更新 删除
    @Update("UPDATE users SET name = #{user.name}, createdAt = #{user.createdAt} WHERE id = #{user.id}")
    void update(@Param("user") User user);

    @Delete("DELETE FROM users WHERE id = #{id}")
    void deleteById(@Param("id") long id);
}
```

MyBatis提供了一个`MapperFactoryBean`来自动创建所有Mapper的实现类

```java
@MapperScan("com.itranswarp.learnjava.mapper")
...其他注解...
public class AppConfig {...}
```

业务类中直接注入

```java
@Component
@Transactional
public class UserService {
    // 注入UserMapper:
    @Autowired
    UserMapper userMapper;

    public User getUserById(long id) {
        // 调用Mapper方法:
        User user = userMapper.getById(id);
        if (user == null) {
            throw new RuntimeException("User not found by id.");
        }
        return user;
    }
}
```



 XML配置

上述在Spring中集成MyBatis的方式，我们只需要用到注解，并没有任何XML配置文件。MyBatis也允许使用XML配置映射关系和SQL语句，例如，更新`User`时根据属性值构造动态SQL：

```xml
<update id="updateUser">
  UPDATE users SET
  <set>
    <if test="user.name != null"> name = #{user.name} </if>
    <if test="user.hobby != null"> hobby = #{user.hobby} </if>
    <if test="user.summary != null"> summary = #{user.summary} </if>
  </set>
  WHERE id = #{user.id}
</update>
```

XML配置的优点是可以组装出动态SQL，并且把所有SQL操作集中在一起。缺点是配置起来太繁琐，调用方法时如果想查看SQL还需要定位到XML配置中。XML配置方式，参考[官方文档](https://mybatis.org/mybatis-3/zh/configuration.html)

使用MyBatis最大的问题是所有SQL都需要全部手写，优点是执行的SQL就是我们自己写的SQL，对SQL进行优化非常简单，也可以编写任意复杂的SQL，或者使用数据库的特定语法，但切换数据库可能就不太容易



MyBatis 缓存：

一级缓存：默认开启，SqlSession级别的缓存，也称为本地缓存
	实质上就是sqlSession级别的一个Map，与数据库同一次会话期间查询到的数据会放在本地缓存中，以后如果需要获取相同的数据，直接从缓存中拿

二级缓存：需要手动开启和配置，基于namespace级别的缓存，通过MyBatis的缓存接口Cache可以自定义实现
	一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中
	如果会话关闭或提交，一级缓存中的数据会被保存到二级缓存中，新的会话查询信息，就可以参照二级缓存中的内容
	不同的namespace查出的数据会放在自己对应的缓存（map）中



---



# Web 开发

Java Web的基础：Servlet容器，以及标准的Servlet组件：

- Servlet：能处理HTTP请求并将HTTP响应返回
- JSP：一种嵌套Java代码的HTML，将被编译为Servlet
- Filter：能过滤指定的URL以实现拦截功能
- Listener：监听指定的事件，如ServletContext、HttpSession的创建和销毁

Servlet容器为每个Web应用程序自动创建一个唯一的`ServletContext`实例，这个实例就代表了Web应用程序本身



## Spring MVC

创建基于Web的Maven工程

引入依赖：

- org.springframework:spring-context:6.0.0
- org.springframework:spring-webmvc:6.0.0
- org.springframework:spring-jdbc:6.0.0
- jakarta.annotation:jakarta.annotation-api:2.1.1
- io.pebbletemplates:pebble-spring6:3.2.0
- ch.qos.logback:logback-core:1.4.4
- ch.qos.logback:logback-classic:1.4.4
- com.zaxxer:HikariCP:5.0.1
- org.hsqldb:hsqldb:2.7.0

`provided`依赖：

- org.apache.tomcat.embed:tomcat-embed-core:10.1.1
- org.apache.tomcat.embed:tomcat-embed-jasper:10.1.1

```ascii art
spring-web-mvc
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── itranswarp
        │           └── learnjava
        │               ├── AppConfig.java
        │               ├── DatabaseInitializer.java
        │               ├── entity
        │               │   └── User.java
        │               ├── service
        │               │   └── UserService.java
        │               └── web
        │                   └── UserController.java
        ├── resources
        │   ├── jdbc.properties
        │   └── logback.xml
        └── webapp
            ├── WEB-INF
            │   ├── templates
            │   │   ├── _base.html
            │   │   ├── index.html
            │   │   ├── profile.html
            │   │   ├── register.html
            │   │   └── signin.html
            │   └── web.xml
            └── static
                ├── css
                │   └── bootstrap.css
                └── js
                    └── jquery.js
```



配置

```java
@Configuration
@ComponentScan
@EnableWebMvc // 启用Spring MVC
@EnableTransactionManagement
@PropertySource("classpath:/jdbc.properties")
public class AppConfig {
    ... // DataSource、JdbcTemplate、PlatformTransactionManager
    @Bean
    WebMvcConfigurer createWebMvcConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addResourceHandlers(ResourceHandlerRegistry registry) {
                // Spring MVC自动处理静态文件，并且映射路径为/static/**
                registry.addResourceHandler("/static/**").addResourceLocations("/static/");
            }
        };
    }
    // 实例化 ViewResolver 指定 对应的模板引擎
    @Bean
    ViewResolver createViewResolver(@Autowired ServletContext servletContext) {
        // 使用Pebble引擎，指定模板文件存放在/WEB-INF/templates/目录
        var engine = new PebbleEngine.Builder().autoEscaping(true)
                // cache:
                .cacheActive(false)
                // loader:
                .loader(new Servlet5Loader(servletContext))
                .build();
        var viewResolver = new PebbleViewResolver(engine);
        viewResolver.setPrefix("/WEB-INF/templates/");
        viewResolver.setSuffix("");
        return viewResolver;
    }
}
// Controller使用@Controller标记 不能@Component
@Controller
public class UserController {
    @Autowired
    UserService userService;

    @PostMapping("/signin")	// @GetMapping("...")
    public ModelAndView doSignin(	// ModelAndView通常包含View的路径和一个Map作为Model
            @RequestParam("email") String email,
            @RequestParam("password") String password,
            HttpSession session) {...}
    ...
}
```

Web应用中启动Spring容器有很多种方法：通过Listener启动，通过Servlet启动，使用XML配置，使用注解配置...
这里使用最简单的：

```xml
<!-- web.xml 中配置Spring MVC提供的DispatcherServlet -->
<?xml version="1.0"?>
<web-app>
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.itranswarp.learnjava.AppConfig</param-value>
        </init-param>
        <load-on-startup>0</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
</web-app>
```

有了这个配置，Servlet容器会首先初始化Spring MVC的`DispatcherServlet`，在`DispatcherServlet`启动时，它根据配置`AppConfig`创建了一个类型是`WebApplicationContext`的IoC容器，完成所有Bean的初始化，并将容器绑到`ServletContext`上

`DispatcherServlet`持有IoC容器，能从IoC容器中获取所有`@Controller`的Bean，因此，`DispatcherServlet`接收到所有HTTP请求后，根据Controller方法配置的路径，就可以正确地把请求转发到指定方法，并根据返回的`ModelAndView`决定如何渲染页面

```java
// 在AppConfig中通过main()方法启动嵌入式Tomcat
public static void main(String[] args) throws Exception {
    Tomcat tomcat = new Tomcat();
    tomcat.setPort(Integer.getInteger("port", 8080));
    tomcat.getConnector();
    Context ctx = tomcat.addWebapp("", new File("src/main/webapp").getAbsolutePath());
    WebResourceRoot resources = new StandardRoot(ctx);
    resources.addPreResources(
            new DirResourceSet(resources, "/WEB-INF/classes", new File("target/classes").getAbsolutePath(), "/"));
    ctx.setResources(resources);
    tomcat.start();
    tomcat.getServer().await();
}
```



---



## 使用 REST

Web应用中，有一类API接口称为**REST**，通常输入输出都是JSON，便于第三方调用或者使用页面JavaScript与之交互

```java
// Spring MVC的@GetMapping和@PostMapping都支持指定输入和输出的格式
// maven 引入依赖 com.fasterxml.jackson.core:jackson-databind:2.14.0
// ↓ 接收JSON，输出JSON
@PostMapping(value = "/rest",
             consumes = "application/json;charset=UTF-8",
             produces = "application/json;charset=UTF-8")
@ResponseBody	// @ResponseBody表示返回的String无需额外处理，直接作为输出内容写入HttpServletResponse
public String rest(@RequestBody User user) { // @RequestBody 输入的JSON直接被Spring反序列化为User这个JavaBean
    return "{\"restSupport\":true}";
}
```

使用`@RestController`代替`@Controller`，每个方法自动变成API接口方法

​	在User类给某属性加上`@JsonIgnore`表示序列化为JSON时忽略该属性
​		允写禁读`@JsonProperty(access = Access.WRITE_ONLY)`



---



## 集成 Filter

在Spring MVC中，`DispatcherServlet`只需要固定配置到`web.xml`中，剩下的工作主要是专注于编写Controller



在Spring MVC中使用Filter：

直接使用Spring MVC自带的`CharacterEncodingFilter`，在全局范围类给`HttpServletRequest`和`HttpServletResponse`强制设置为UTF-8编码

```xml
<!-- web.xml -->
<web-app>
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```

编写一个`AuthFilter`来在HTTP请求中添加头`Authorization: Basic email:password`

```java
@Component
public class AuthFilter implements Filter {
    @Autowired
    UserService userService;

    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        HttpServletRequest req = (HttpServletRequest) request;
        // 获取Authorization头:
        String authHeader = req.getHeader("Authorization");
        if (authHeader != null && authHeader.startsWith("Basic ")) {
            // 从Header中提取email和password:
            String email = prefixFrom(authHeader);
            String password = suffixFrom(authHeader);
            // 登录:
            User user = userService.signin(email, password);
            // 放入Session:
            req.getSession().setAttribute(UserController.KEY_USER, user);
        }
        // 继续处理请求:
        chain.doFilter(request, response);
    }
}
```

这时如果跟`CharacterEncodingFilter`一样，在`web.xml`中声明`AuthFilter`，那么它将不会被Spring容器初始化

使用Spring MVC提供的`DelegatingFilterProxy`：让Servlet容器实例化的Filter，间接引用Spring容器实例化的`AuthFilter`

```xml
<web-app>
    <filter>
        <filter-name>authFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>authFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```



---



## 使用 Interceptor

Web程序中的Filter由Servlet容器管理，拦截范围包括整个处理流程：DispatcherServlet Controller ModelAndView

使用Interceptor，只针对Controller方法拦截
	好处是它本身是Spring管理的Bean，可以更简单地注入任意Bean



```java
@Order(1)
@Component
// Interceptor必须实现HandlerInterceptor接口，可以选择实现preHandle()、postHandle()和afterCompletion()方法
public class LoggerInterceptor implements HandlerInterceptor {

    final Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    // preHandle 执行于Controller方法调用前
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        logger.info("preHandle {}...", request.getRequestURI());
        if (request.getParameter("debug") != null) {
            PrintWriter pw = response.getWriter();
            pw.write("<p>DEBUG MODE</p>");
            pw.flush();
            return false;	// preHandle() 回false表示无需调用Controller方法继续处理
        }
        return true;
    }

    @Override
    // postHandle 执行于Controller方法正常返回后 可以继续加工Controller方法返回的ModelAndView
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        logger.info("postHandle {}.", request.getRequestURI());
        if (modelAndView != null) {
            modelAndView.addObject("__time__", LocalDateTime.now());
        }
    }

    @Override
    // afterCompletion 无论Controller方法是否抛异常都执行 ex即Controller方法抛出的异常
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        logger.info("afterCompletion {}: exception = {}", request.getRequestURI(), ex);
    }
}

@Order(2)	// @Order指定顺序
@Component
public class AuthInterceptor implements HandlerInterceptor { ...}

// 在WebMvcConfigurer中注册所有的Interceptor 使拦截器生效
@Bean
WebMvcConfigurer createWebMvcConfigurer(@Autowired HandlerInterceptor[] interceptors) {
    return new WebMvcConfigurer() {
        public void addInterceptors(InterceptorRegistry registry) {
            for (var interceptor : interceptors) {
                registry.addInterceptor(interceptor);
            }
        }
        ...
    };
}
```



`@ExceptionHandler`：Spring MVC提供的定义异常处理方法的注解

```java
@Controller
public class UserController {
    @ExceptionHandler(RuntimeException.class)
    // 异常处理方法没有固定的方法签名，可以传入Exception、HttpServletRequest等
    // 返回值可以是void，也可以是ModelAndView
    // 这里通过@ExceptionHandler(RuntimeException.class)表示当发生RuntimeException的时候，就自动调用此方法处理
    public ModelAndView handleUnknowException(Exception ex) {
        return new ModelAndView("500.html", Map.of("error", ex.getClass().getSimpleName(), "message", ex.getMessage()));
    }
    ...
}
```

使用`@ExceptionHandler`定义异常处理方法的好处是：
	如果应用程序内部发生了预料之外的异常，可以给用户显示一个出错页面，比如”哔咔被玩坏了，绝对不是哔咔的问题“，而不是简单的500 Internal Server Error或404 Not Found



---



## 处理 CORS

跨域访问访问：A站的JavaScript访问B站API

B站返回给A站返回一个正确的`Access-Control-Allow-Origin`响应头即可，决定权在API服务提供方

1. **`@CrossOrigin`**

```java
// 在@RestController的class级别或方法级别定义一个@CrossOrigin
// 指定只允许来自local.liaoxuefeng.com跨域访问
// 允许多个 origins = {"http://a.com", "https://www.b.com"}) 数组
// 允许所有域访问 origins = "*"
@CrossOrigin(origins = "http://local.liaoxuefeng.com:8080")
@RestController
@RequestMapping("/api")
public class ApiController { ...}
```

2. **`@CorsRegistry`**

```java
// 在WebMvcConfigurer中定义一个全局CORS配置
@Bean
WebMvcConfigurer createWebMvcConfigurer() {
    return new WebMvcConfigurer() {
        @Override
        public void addCorsMappings(CorsRegistry registry) {
            registry.addMapping("/api/**")
                    .allowedOrigins("http://local.liaoxuefeng.com:8080")
                    .allowedMethods("GET", "POST")
                    .maxAge(3600);
            // 可以继续添加其他URL规则:
            // registry.addMapping("/rest/v2/**")...
        }
    };
}
```

3. **`@CorsFilter`**

```java
// 使用Spring提供的CorsFilter
// 将Spring容器内置的Bean暴露为Servlet容器的Filter需要配置web.xml
// 太繁琐 不用
```



---



## i18n

- internationalization 国际化

- localization 本地化 l10n

- globalization 全球化 g11n

对于Web应用程序，要实现国际化功能，主要是渲染View的时候，要把各种语言的资源文件提出来，这样，不同的用户访问同一个页面时，显示的语言就是不同的



在Spring MVC应用程序中实现国际化：

1. **获取 Local**

Web应用程序中，HTTP规范规定了浏览器会在请求中携带`Accept-Language`头，用来指示用户浏览器设定的语言顺序

```
Accept-Language: zh-CN,zh;q=0.8,en;q=0.2
```

- 先选择简体中文，其次选择中文，最后选择英文

- q表示权重，解析后我们可获得一个根据优先级排序的语言列表，把它转换为Java的`Locale`，即获得了用户的`Locale`
- 大多数框架通常只返回权重最高的`Locale`

Spring MVC通过`LocaleResolver`来自动从`HttpServletRequest`中获取`Locale`

```java
@Primary
@Bean
LocaleResolver createLocaleResolver() {
    var clr = new CookieLocaleResolver();
    clr.setDefaultLocale(Locale.ENGLISH);
    clr.setDefaultTimeZone(TimeZone.getDefault());
    return clr;
}
```

2. **提取资源文件**

把写死在模板中的字符串以资源文件的方式存储在外部

​	例如`messages_zh_CN.properties`、`messages_ja_JP.properties`

每个资源文件都有相同的key

```properties
## messages.properties:
language.select=Language
home=Home
signin=Sign In
copyright=Copyright©{0,number,#}
## messages_zh_CN.properties:
language.select=语言
home=首页
signin=登录
copyright=版权所有©{0,number,#}
```

3. **创建 MessageSource**

```java
@Bean("i18n")
MessageSource createMessageSource() {
    // ResourceBundleMessageSource 会自动根据主文件名自动把所有相关语言的资源文件都读进来
    var messageSource = new ResourceBundleMessageSource();
    // 指定文件是UTF-8编码:
    messageSource.setDefaultEncoding("UTF-8");
    // 指定主文件名:
    messageSource.setBasename("messages");
    return messageSource;
}
```

4. **实现多语言**

```java
@Component
public class MvcInterceptor implements HandlerInterceptor {
    @Autowired
    LocaleResolver localeResolver;

    // 注入的MessageSource 名称i18n
    @Autowired
    @Qualifier("i18n")
    MessageSource messageSource;

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        if (modelAndView != null // 返回了ModelAndView
            && modelAndView.getViewName() != null // 设置了View
            && !modelAndView.getViewName().startsWith("redirect:") // 不是重定向
        ) {
            // 解析用户的Locale:
            Locale locale = localeResolver.resolveLocale(request);
            // 放入Model:
            modelAndView.addObject("__messageSource__", messageSource);
            modelAndView.addObject("__locale__", locale);
        }
    }
}
```

```java
@Component
public class MvcInterceptor implements HandlerInterceptor {
    @Autowired
    LocaleResolver localeResolver;

    // 注意注入的MessageSource名称是i18n:
    @Autowired
    @Qualifier("i18n")
    MessageSource messageSource;

    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        if (modelAndView != null // 返回了ModelAndView
            && modelAndView.getViewName() != null // 设置了View
            && !modelAndView.getViewName().startsWith("redirect:") // 不是重定向
        ) {
            // 解析用户的Locale
            Locale locale = localeResolver.resolveLocale(request);
            // 放入Model:
            modelAndView.addObject("__messageSource__", messageSource);
            modelAndView.addObject("__locale__", locale);
        }
    }
}
```

```html
<!-- 在View中调用MessageSource.getMessage()方法来实现多语言 -->
<a href="/signin">{{ __messageSource__.getMessage('signin', null, __locale__) }}</a>
```

在Pebble中，我们可以封装一个国际化函数

```java
// 修改 ViewResolver 的创建
@Bean
ViewResolver createViewResolver(@Autowired ServletContext servletContext, @Autowired @Qualifier("i18n") MessageSource messageSource) {
    var engine = new PebbleEngine.Builder()
            .autoEscaping(true)
            .cacheActive(false)
            .loader(new Servlet5Loader(servletContext))
            // 添加扩展:
            .extension(createExtension(messageSource))
            .build();
    var viewResolver = new PebbleViewResolver();
    viewResolver.setPrefix("/WEB-INF/templates/");
    viewResolver.setSuffix("");
    viewResolver.setPebbleEngine(engine);
    return viewResolver;
}

private Extension createExtension(MessageSource messageSource) {
    return new AbstractExtension() {
        @Override
        public Map<String, Function> getFunctions() {
            return Map.of("_", new Function() {
                public Object execute(Map<String, Object> args, PebbleTemplate self, EvaluationContext context, int lineNumber) {
                    String key = (String) args.get("0");
                    List<Object> arguments = this.extractArguments(args);
                    Locale locale = (Locale) context.getVariable("__locale__");
                    return messageSource.getMessage(key, arguments.toArray(), "???" + key + "???", locale);
                }
                private List<Object> extractArguments(Map<String, Object> args) {
                    int i = 1;
                    List<Object> arguments = new ArrayList<>();
                    while (args.containsKey(String.valueOf(i))) {
                        Object param = args.get(String.valueOf(i));
                        arguments.add(param);
                        i++;
                    }
                    return arguments;
                }
                public List<String> getArgumentNames() {
                    return null;
                }
            });
        }
    };
}
```

多语言页面改为

```html
<a href="/signin">{{ _('signin') }}</a>
```

5. **实现切换 Local**

```java
@Controller
public class LocaleController {
    final Logger logger = LoggerFactory.getLogger(getClass());

    @Autowired
    LocaleResolver localeResolver;

    @GetMapping("/locale/{lo}")
    public String setLocale(@PathVariable("lo") String lo, HttpServletRequest request, HttpServletResponse response) {
        // 根据传入的lo创建Locale实例:
        Locale locale = null;
        int pos = lo.indexOf('_');
        if (pos > 0) {
            String lang = lo.substring(0, pos);
            String country = lo.substring(pos + 1);
            locale = new Locale(lang, country);
        } else {
            locale = new Locale(lo);
        }
        // 设定此Locale:
        localeResolver.setLocale(request, response, locale);
        logger.info("locale is set to {}.", locale);
        // 刷新页面:
        String referer = request.getHeader("Referer");
        return "redirect:" + (referer == null ? "/" : referer);
    }
}
```



---



## 异步处理

Servlet模型中，每个请求都是由某个线程处理，然后，将响应写入IO流，发送给客户端

如果一个请求处理的时间较长，例如几秒钟甚至更长，那么，这种基于线程池的同步模型很快就会把所有线程耗尽，导致服务器无法响应新的请求
如果把长时间处理的请求改为异步处理，那么线程池的利用率就会大大提高
Servlet从3.0规范开始添加了异步支持，允许对一个请求进行异步处理



```xml
<!-- web.xml -->
<web-app>
    <display-name>Archetype Created Web Application</display-name>

    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextClass</param-name>
            <param-value>org.springframework.web.context.support.AnnotationConfigWebApplicationContext</param-value>
        </init-param>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>com.itranswarp.learnjava.AppConfig</param-value>
        </init-param>
        <load-on-startup>0</load-on-startup>
        <!-- true -> Servlet容器支持async处理 -->
        <async-supported>true</async-supported>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/*</url-pattern>
    </servlet-mapping>
</web-app>
```

Controller 中 async 处理逻辑：

1. ```java
   // 返回一个Callable，Spring MVC自动把返回的Callable放入线程池执行，等待结果返回后再写入响应
   @GetMapping("/users")
   public Callable<List<User>> users() {
       return () -> {
           // 模拟3秒耗时:
           try {
               Thread.sleep(3000);
           } catch (InterruptedException e) {}
           return userService.getUsers();
       };
   }
   ```

2. ```java
   // 返回一个DeferredResult对象，然后在另一个线程中，设置此对象的值并写入响应
   @GetMapping("/users/{id}")
   public DeferredResult<User> user(@PathVariable("id") long id) {
       DeferredResult<User> result = new DeferredResult<>(3000L); // 3秒超时
       new Thread(() -> {
           // 等待1秒:
           try {
               Thread.sleep(1000);
           } catch (InterruptedException e) {}
           try {
               User user = userService.getUserById(id);
               // 设置正常结果并由Spring MVC写入Response:
               result.setResult(user);
           } catch (Exception e) {
               // 设置错误结果并由Spring MVC写入Response:
               result.setErrorResult(Map.of("error", e.getClass().getSimpleName(), "message", e.getMessage()));
           }
       }).start();
       return result;
   }
   ```



**async Filter**：

```xml
<!-- web.xml -->
<web-app ...>
    ...
    <filter>
        <filter-name>sync-filter</filter-name>
        <filter-class>com.itranswarp.learnjava.web.SyncFilter</filter-class>
    </filter>

    <filter>
        <filter-name>async-filter</filter-name>
        <filter-class>com.itranswarp.learnjava.web.AsyncFilter</filter-class>
        <!-- 支持<async-supported>的Filter既可以过滤async处理请求，也可以过滤正常的同步处理请求 -->
        <!-- 普通Filter 遇到async请求 直接报错 -->
        <async-supported>true</async-supported>
    </filter>

    <filter-mapping>
        <filter-name>sync-filter</filter-name>
        <url-pattern>/api/version</url-pattern>
    </filter-mapping>

    <filter-mapping>
        <filter-name>async-filter</filter-name>
        <url-pattern>/api/*</url-pattern>
    </filter-mapping>
    ...
</web-app>
```



Servlet 3.0规范添加的异步支持效率不高

Java标准库提供了封装操作系统的异步IO包`java.nio`高效但复杂

大部分需要高性能异步IO的应用程序会选择[Netty](https://netty.io/)这样的框架，它基于NIO提供了更易于使用的API，方便开发异步应用程序



---



## WebSocket

WebSocket是一种基于HTTP的长链接技术

建立TCP连接后，浏览器请求附带：

```plain
GET /chat HTTP/1.1
Host: www.example.com
Upgrade: websocket
Connection: Upgrade
```

表示客户端希望升级连接，变成长连接的WebSocket，服务器返回升级成功的响应：

```plain
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

收到成功响应后表示WebSocket“握手”成功，如此，代表WebSocket的这个TCP连接将不会被服务器关闭，，而是一直保持，服务器可随时向浏览器推送消息，浏览器也可随时向服务器推送消息



在Spring MVC中实现对WebSocket的支持：

1. maven引入依赖：

- org.apache.tomcat.embed:tomcat-embed-websocket:10.1.1	嵌入式Tomcat支持WebSocket的组件
- org.springframework:spring-websocket:6.0.0	Spring封装的支持WebSocket的接口

2. 配置 AppConfig：

```java
@Configuration
@ComponentScan
@EnableWebMvc
@EnableWebSocket // 启用WebSocket支持
public class AppConfig {
    @Bean
    // 这个Bean在内部通过WebSocketHandlerRegistry注册能处理WebSocket的WebSocketHandler，以及可选的WebSocket拦截器HandshakeInterceptor
    WebSocketConfigurer createWebSocketConfigurer(
            @Autowired ChatHandler chatHandler,
            @Autowired ChatHandshakeInterceptor chatInterceptor)
    {
        return new WebSocketConfigurer() {
            public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
                // 把URL与指定的WebSocketHandler关联，可关联多个:
                registry.addHandler(chatHandler, "/chat").addInterceptors(chatInterceptor);
            }
        };
    }
    ...
}
```

3. 处理WebSocket连接

和处理普通HTTP请求不同，没法用一个方法处理一个URL

Spring提供了`TextWebSocketHandler`和`BinaryWebSocketHandler`分别处理文本消息和二进制消息，这里我们选择文本消息作为聊天室的协议，因此，`ChatHandler`需要继承自`TextWebSocketHandler`：

```java
@Component
public class ChatHandler extends TextWebSocketHandler { ...}
```

当浏览器请求一个WebSocket连接后，如果成功建立连接，Spring会自动调用`afterConnectionEstablished()`方法，任何原因导致WebSocket连接中断时，Spring会自动调用`afterConnectionClosed`方法，因此，覆写这两个方法即可处理连接成功和结束后的业务逻辑：

```java
@Component
public class ChatHandler extends TextWebSocketHandler {
    // 保存所有Client的WebSocket会话实例:
    private Map<String, WebSocketSession> clients = new ConcurrentHashMap<>();
    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        // 新会话根据ID放入Map:
        clients.put(session.getId(), session);
        session.getAttributes().put("name", "Guest1");
    }
    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        clients.remove(session.getId());
    }
}
```

每个WebSocket会话以`WebSocketSession`表示，且已分配唯一ID。和WebSocket相关的数据，例如用户名称等，均可放入关联的`getAttributes()`中

用实例变量`clients`持有当前所有的`WebSocketSession`是为了广播，即向所有用户推送同一消息时：

```java
String json = ...
TextMessage message = new TextMessage(json);
for (String id : clients.keySet()) {
    WebSocketSession session = clients.get(id);
    session.sendMessage(message);
}
```

我们发送的消息是序列化后的JSON，可以用`ChatMessage`表示：

```java
public class ChatMessage {
	public long timestamp;
	public String name;
    public String text;
}
```

每收到一个用户的消息后，我们就需要广播给所有用户：

```java
@Component
public class ChatHandler extends TextWebSocketHandler {
    ...
    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String s = message.getPayload();
        String r = ... // 根据输入消息构造待发送消息
        broadcastMessage(r); // 推送给所有用户
    }
}
```

如果要推送给指定的几个用户，那就需要在`clients`中根据条件查找出某些`WebSocketSession`，然后发送消息。

注意到我们在注册WebSocket时还传入了一个`ChatHandshakeInterceptor`，这个类实际上可以从`HttpSessionHandshakeInterceptor`继承，它的主要作用是在WebSocket建立连接后，把HttpSession的一些属性复制到WebSocketSession，例如，用户的登录信息等：

```java
@Component
public class ChatHandshakeInterceptor extends HttpSessionHandshakeInterceptor {
    public ChatHandshakeInterceptor() {
        // 指定从HttpSession复制属性到WebSocketSession:
        super(List.of(UserController.KEY_USER));
    }
}
```

这样，在`ChatHandler`中，可以从`WebSocketSession.getAttributes()`中获取到复制过来的属性

4. 客户端开发

在完成了服务器端的开发后，我们还需要在页面编写一点JavaScript逻辑：

```javascript
// 创建WebSocket连接:
var ws = new WebSocket('ws://' + location.host + '/chat');
// 连接成功时:
ws.addEventListener('open', function (event) {
    console.log('websocket connected.');
});
// 收到消息时:
ws.addEventListener('message', function (event) {
    console.log('message: ' + event.data);
    var msgs = JSON.parse(event.data);
    // TODO:
});
// 连接关闭时:
ws.addEventListener('close', function () {
    console.log('websocket closed.');
});
// 绑定到全局变量:
window.chatWs = ws;
```

用户可以在连接成功后任何时候给服务器发送消息：

```javascript
var inputText = 'Hello, WebSocket.';
window.chatWs.send(JSON.stringify({text: inputText}));
```

最后，连调浏览器和服务器端，如果一切无误，可以开多个不同的浏览器测试WebSocket的推送和广播。

和上一节我们介绍的异步处理类似，Servlet的线程模型并不适合大规模的长链接。基于NIO的Netty等框架更适合处理WebSocket长链接，我们将在后面介绍。
