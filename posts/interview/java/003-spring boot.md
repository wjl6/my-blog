---
title: "003-spring boot.md"
date: 2025-12-26 14:59:36
tags: []
---

### 1. 什么是 Spring？
Spring 是一个 轻量级 IOC + AOP 容器，核心目标是：
- 解耦
- 提高可维护性

---

### 2. 什么是 IOC？什么是 DI？
- IOC（控制反转） 是一种设计思想，核心是：将对象的创建、管理、依赖关系的维护等 “控制权”，从应用程序代码本身转移到第三方容器（如 Spring IOC 容器）
- DI（依赖注入） 是 “落地手段”，核心是容器在创建对象时，自动将依赖注入到对象中
- **核心价值**：解耦、简化开发、便于维护，是 Spring 框架的基石，几乎所有 Spring 功能（如 AOP、事务）都基于 IOC 容器实现
---

### 3. AOP是什么？ 有什么应用场景？
- AOP（Aspect-Oriented Programming）即面向切面编程，是与 OOP（面向对象编程）互补的编程思想：
> - OOP 关注 “业务逻辑的纵向划分”（如用户模块、订单模块、支付模块）；
> - AOP 关注 “横切逻辑的横向抽取”（如日志、事务、权限、性能监控等，这些逻辑横跨多个业务模块） I
- 把分散在各个业务方法中的通用逻辑（如日志记录）抽离出来，形成一个独立的 “切面”，通过 “动态织入” 的方式应用到目标方法上，实现通用逻辑与业务逻辑的解耦，避免重复代码

| 术语 | 通俗解释 |
|------|----------|
| 切面（Aspect） | 抽离出来的通用逻辑模块（如日志切面、事务切面），包含 “通知” 和 “切点” |
| 通知（Advice） | 切面的具体逻辑（如日志的 “记录入参”“记录返回值”），分 5 种类型（前置 / 后置 / 环绕 / 异常 / 最终） |
| 切点（Pointcut） | 定义切面要 “织入” 到哪些目标方法上（如所有 com.service 包下的方法） |
| 织入（Weaving） | 将切面逻辑应用到目标方法的过程（Spring 中默认是运行时织入） |
| 连接点（JoinPoint） | 程序执行过程中可被切面拦截的点（如方法调用、异常抛出），切点是连接点的筛选结果 |


- **总结**
- **AOP 核心定义**：面向切面编程，抽离横切逻辑（日志、事务等）形成切面，动态织入目标方法，解耦通用逻辑与业务逻辑；
- **核心应用场景**：日志记录、事务管理、权限校验、性能监控、异常处理、缓存控制、接口限流；
- **核心价值**：避免重复代码，统一通用逻辑，提升代码可维护性和扩展性；
- **底层原理**：Spring AOP 基于动态代理（JDK/CGLIB）实现，运行时织入切面逻辑。
---

### 4. AOP 的核心原理（简化版，面试够用）
- Spring AOP 底层基于动态代理实现，分为两种方式：
- JDK 动态代理：目标类实现了接口时，通过生成接口的代理类来织入切面逻辑；
- CGLIB 动态代理：目标类未实现接口时，通过生成目标类的子类来织入切面逻辑。


- **核心流程**：
> - 定义切面（Aspect），指定切点（Pointcut）和通知（Advice）； 
> - 程序运行时，Spring 为目标类创建代理对象； 
> - 调用目标方法时，代理对象先执行切面的通知逻辑，再执行目标方法； 
> - 最终返回结果，完成切面的织入。


---

### 5. Spring AOP 和 AspectJ 区别？
- 本质差异：
> - Spring AOP 是 **Spring 内置的动态代理实现**，轻量级、仅支持方法级拦截、**运行期织入**、仅作用于 Spring Bean；
> - AspectJ 是**完整的 AOP 标准，静态织入**、支持所有连接点、可作用于任意类、性能更高，但使用复杂度高。
- 核心选择原则：
> - 90% 的 Spring 项目场景，用 Spring AOP 足够（方法级切面、简单易用）；
> - 若需拦截字段 / 构造器、非 Spring 类，或追求极致性能，用 AspectJ；
> - 语法复用：Spring AOP 借用 AspectJ 的注解语法（@Aspect/@Before），但底层实现无关，无需混淆。
- 简单记：Spring AOP 是 “够用就好” 的轻量级方案，AspectJ 是 “无所不能” 的专业级方案。

---

### 6. Bean 的生命周期是什么？
- 本质是Spring 容器从创建 Bean 实例开始，到 Bean 实例最终被销毁的整个过程
  ![](../../../assets/images/interview/java/002/01.png)


```text
1. 实例化（new Bean()）→ 
2. 属性填充（@Autowired 注入依赖）→ 
3. 执行 Aware 方法（setBeanName/setApplicationContext 等）→ 
4. 执行初始化前处理器（BeanPostProcessor.postProcessBeforeInitialization）→ 
5. 执行初始化方法（@PostConstruct / init-method / InitializingBean.afterPropertiesSet）→ 
6. 执行初始化后处理器（BeanPostProcessor.postProcessAfterInitialization）→ 
7. Bean 就绪（可被使用）→ 
8. 容器销毁时执行销毁方法（@PreDestroy / destroy-method / DisposableBean.destroy）
``` 

---

### 7. Spring Boot 为什么能自动配置？
- **核心原理**：Spring Boot 自动配置基于 @EnableAutoConfiguration 开启总开关，通过 SPI 加载自动配置类，再通过 @Conditional 系列注解按需生效，最终实现 “按需配置、开箱即用”；
- **核心精髓**：条件注解（@Conditional）保证配置 “按需生效”，@ConditionalOnMissingBean 保证 “用户配置优先”；
- **核心价值**：消除传统 Spring 繁琐的 XML/Java 配置，简化开发，同时保留灵活的定制能力。
---

### 8. 自动配置的完整流程
- **启动触发**：Spring Boot 主类的 @SpringBootApplication 包含 @EnableAutoConfiguration，开启自动配置；
- **加载配置类**：AutoConfigurationImportSelector 通过 SPI 机制，加载 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports 中的所有自动配置类；
- **条件筛选**：Spring 容器根据自动配置类上的 @Conditional 注解，筛选出满足条件的配置类；
- **创建 Bean**：满足条件的配置类生效，自动创建对应的 Bean 到容器中；
- **用户覆盖**：如果用户自定义了相同类型的 Bean，Spring Boot 会优先使用用户的 Bean（@ConditionalOnMissingBean）；
- **配置绑定**：自动配置类通过 @EnableConfigurationProperties 绑定配置文件中的属性，覆盖默认值。

---

### 9. Spring Boot 如何简化配置？
- Spring Boot 并非 “消除配置”，而是通过**约定大于配置、自动配置、简化配置格式、内置默认值**等核心手段，将传统 Spring 繁琐的配置简化到 “开箱即用” 的程度
  
| 核心手段          | 具体解释 | 解决的传统 Spring 痛点 |
|:--------------|:---------|:----------------------|
| 自动配置（核心）      | 基于依赖自动推断并配置 Bean（如引入 web 依赖自动配 Tomcat/MVC） | 需手动编写 XML/Java 配置定义 Bean（如 DispatcherServlet、数据源） |
| 约定大于配置        | 内置默认规则（如配置文件默认路径、Bean 扫描路径），无需手动指定 | 需手动配置扫描包、视图解析器前缀、端口号等 |
| 简化配置格式        | 支持 application.yml/yaml 简洁格式，替代繁琐的 XML/Properties | XML 配置冗余（如 <bean> 标签嵌套），Properties 格式不支持层级、易写错 |
| 起步依赖（Starter） | 一键引入场景化依赖（如 spring-boot-starter-web），自动管理依赖版本 | 需手动引入多个依赖（如 spring-web、spring-mvc、tomcat），且需手动解决版本冲突 |
| 配置绑定          | 配置文件属性自动绑定到 Java 类（@ConfigurationProperties），无需手动读取 | 需手动通过 @Value 逐个注入属性，或编写代码读取 Properties 文件 |
| 内置容器 / 组件     | 内置 Tomcat/Jetty、日志框架等，无需手动部署 / 配置 | 需手动下载、配置容器，编写日志配置文件（如 log4j.xml） |
---

### 10. Spring 事务的传播行为？

- (1) REQUIRED（默认）：支持当前事务，无则新建
> - 核心逻辑：   
> **如果外层有事务，内层复用外层事务**；   
> **如果外层无事务，内层新建自己的事务**。   
> - 通俗解释：“能复用就复用，不能就自己建”，这是最常用的传播行为（默认值）。  
> - **关键特点**：内外层共用一个事务，要么一起提交，要么一起回滚（原子性）

- (2) REQUIRES_NEW：新建事务，暂停当前事务
> - 核心逻辑：   
> **无论外层是否有事务，内层都新建独立事务**；  
> **外层事务会被暂停，内层事务执行完成后，外层事务继续**。
> - 通俗解释：“老子自己单干，不跟你玩”，内外层事务完全隔离，互不影响。
> - **关键特点**：内层事务独立提交 / 回滚，不受外层事务影响；外层事务异常不回滚内层，内层异常也不回滚外层（除非手动捕获）。

- (3) SUPPORTS：支持当前事务，无则以非事务执行
> - 核心逻辑：   
> **如果外层有事务，内层复用外层事务**；  
> **如果外层无事务，内层不开启事务，以非事务方式执行**。
> - 通俗解释：“有事务就跟你走，没事务就裸奔”，适合 “可选事务” 的场景。

- (4) NOT_SUPPORTED：以非事务执行，暂停当前事务
> - 核心逻辑：   
> **无论外层是否有事务，内层都以非事务方式执行**；  
> 如果外层有事务，外层事务会被暂停，内层执行完成后恢复。
> - 通俗解释：“拒绝事务，就算你有我也不用”，适合无需事务的操作（如日志记录）。
> - 适用场景：记录操作日志、发送消息等，即使外层有事务，这些操作也无需纳入事务（避免事务范围过大，影响性能）.

- (5) MANDATORY：必须在事务中执行，否则抛异常
> - 核心逻辑：   
> **如果外层有事务，内层复用外层事务**；  
> **如果外层无事务，直接抛出 IllegalTransactionStateException 异常**。
> - 通俗解释：“必须有事务，没事务就报错”，强制要求调用方必须开启事务
> - 适用场景：核心业务方法（如转账、支付），必须在事务中执行，防止调用方漏加 @Transactional。

- (6) NEVER：以非事务执行，有事务则抛异常
> - 核心逻辑：   
> **如果外层无事务，内层以非事务方式执行**；  
> **如果外层有事务，直接抛出异常**。
> - 通俗解释：“绝对不要事务，有事务就报错”，与 MANDATORY 完全相反。
> - 适用场景：纯查询且绝对不能有事务的方法（如统计报表查询，避免事务锁表）。

- (7) NESTED：嵌套事务（基于保存点）
> - 核心逻辑：   
> 如果外层有事务，内层创建嵌套事务（基于数据库保存点 Savepoint）；    
> 如果外层无事务，内层等同于 REQUIRED（新建事务）。
> - 通俗解释：“外层事务的子事务”，内层事务依赖外层事务，且可独立回滚。。
> - 适用场景：NESTED 依赖数据库支持，适合复杂业务的部分回滚。

| 传播行为 | 外层有事务 | 外层无事务 | 核心特点 | 常用场景 |
|----------|------------|------------|----------|----------|
| REQUIRED | 复用外层事务 | 新建事务 | 原子性（一起提交 / 回滚） | 大部分业务方法（默认） |
| REQUIRES_NEW | 新建独立事务 | 新建事务 | 完全隔离（互不影响） | 日志、消息发送 |
| NESTED | 嵌套事务（保存点） | 新建事务 | 子事务（内层回滚不影响外层） | 复杂业务的部分回滚 |
| SUPPORTS | 复用外层事务 | 非事务执行 | 可选事务 | 查询方法 |
| MANDATORY | 复用外层事务 | 抛异常 | 强制事务 | 核心业务（支付 / 转账） |

---

### 11. @Transactional 失效场景？

- (1) 注解修饰非 public 方法（最常见）
> Spring 事务基于动态代理实现，而 JDK/CGLIB 动态代理仅对 public 方法生效
- (2) 同类方法内部调用（无代理介入）
> Spring 事务的本质是 “代理对象调用方法”，如果在同一个类中，非事务方法调用事务方法（或事务方法调用另一个事务方法），会直接调用目标对象的方法，而非代理对象的方法，导致事务注解失效
- (3) 未捕获异常（或捕获后未抛出）
> Spring 事务默认仅在抛出 RuntimeException/Error 时触发回滚，且如果异常被 try-catch 捕获但未重新抛出，事务管理器无法感知异常，导致事务不回滚。
- (4) 异常类型不匹配（默认仅回滚运行时异常）
> @Transactional 默认仅对 RuntimeException（非检查型异常）和 Error 回滚，对 IOException、SQLException 等检查型异常不回滚，若业务抛出这类异常，事务会失效。
- (5) 数据源未配置事务管理器
> Spring Boot 单数据源：自动配置 DataSourceTransactionManager，无需手动配置；  
> 多数据源：手动配置事务管理器，并通过 @Transactional(value = "txManager1") 指定；  
> 传统 Spring：添加 @EnableTransactionManagement 注解开启事务支持
- (6) 传播行为配置错误
> 如果事务方法的传播行为配置为 NOT_SUPPORTED/NEVER（拒绝事务），即使加了 @Transactional，也会以非事务方式执行，导致事务失效
- (7) 数据库不支持事务
> 如果底层数据库不支持事务（如 MySQL 的 MyISAM 引擎），即使配置了 @Transactional，事务也无法生效（MyISAM 不支持事务，InnoDB 支持）。
- (8) 事务超时配置不合理（隐性失效）
> 如果事务方法执行时间超过 @Transactional(timeout = n) 配置的超时时间，事务会被强制回滚，看似 “失效”（实际是超时触发回滚）

---

### 12. Spring Boot 和 Spring Cloud 的区别？你项目中是怎么用的？

- Spring Boot 解决的是单体应用的快速开发和配置问题，而 Spring Cloud 是在此基础上解决微服务架构下的服务治理问题
- 并不是一上来就 Cloud，业务复杂度不够时反而会增加运维和排错成本

---

### 13. @Async 本质？
- @Async 本质上是让被注解的方法脱离当前线程，交由 Spring 管理的线程池来执行（而非 JVM 直接创建新线程）
- 关键细节：@Async 的线程池配置（避坑重点）
- 默认线程池的问题：Spring 自带的默认异步线程池（SimpleAsyncTaskExecutor）有坑 —— 它不是真正的线程池，而是每次调用都创建新线程（这是唯一 “创建新线程” 的情况），高并发下会导致线程爆炸，必须自定义线程池

---

### 14. Spring Boot 启动过程？

Spring Boot 的启动过程是 **“封装 + 自动配置 + 上下文初始化 + Bean 生命周期管理”** 的闭环流程，核心是通过 SpringApplication.run() 触发，从应用启动到最终 Bean 就绪，可拆解为 8 个核心阶段，同时依托自动配置、上下文刷新等机制实现 “约定大于配置” 的核心特性

```text
main() → SpringApplication.run()
  ↓
初始化 SpringApplication（推断应用类型、加载监听器/初始化器）
  ↓
环境准备（加载配置、激活环境）
  ↓
创建 ApplicationContext（按应用类型）
  ↓
加载 BeanDefinition（扫描组件 + 自动配置）
  ↓
上下文刷新（核心）：
  ├─ BeanFactory 初始化
  ├─ BeanPostProcessor 注册
  ├─ Web 容器初始化（Web 应用）
  ├─ Bean 实例化 + 依赖注入 + 初始化
  └─ 上下文刷新完成
  ↓
启动 Web 容器（Web 应用） + 执行 ApplicationRunner/CommandLineRunner
  ↓
应用就绪（ApplicationReadyEvent）
```

- Spring Boot 的启动过程本质是 **“Spring 上下文的初始化 + 自动配置的落地 + 嵌入式容器的启动”**，核心围绕 ApplicationContext 的创建和刷新展开，通过 “约定大于配置” 的自动配置、Starter 机制，大幅简化了 Spring 应用的开发和部署。
- 理解启动流程的关键是抓住 “**上下文刷新（refresh）**” 这个核心阶段，它涵盖了 Bean 的全生命周期管理，也是自动配置、依赖注入、AOP 等核心特性的落地环节。



---

### 15. Spring Boot 中常见的设计模式？

| 序号 | 设计模式       | 英文名称           | 描述                                                                 | 在 Spring Boot 中的典型应用                                                                 | 示例                          |
|------|----------------|--------------------|----------------------------------------------------------------------|--------------------------------------------------------------------------------------------|-------------------------------|
| 1    | **单例模式**       | Singleton Pattern  | 确保一个类只有一个实例，并提供全局访问点                             | Spring Bean 默认作用域为 singleton，整个应用上下文共享一个实例                             | @Service、@Component 注解的类 |
| 2    | **工厂模式**       | Factory Pattern    | 定义创建对象的接口，让子类或配置决定实例化哪一个类                   | BeanFactory / ApplicationContext 负责根据配置动态创建 Bean；FactoryBean 接口自定义创建逻辑 | applicationContext.getBean()  |
| 3    | **代理模式**       | Proxy Pattern      | 为对象提供代理以控制访问，常用于添加额外行为                         | Spring AOP 使用 JDK 动态代理或 CGLIB 实现事务、日志、安全等横切关注点                       | @Transactional 注解的方法被代理 |
| 4    | 模板方法模式   | Template Method Pattern | 定义算法骨架，将部分步骤延迟到子类或回调中实现                     | 各种 Template 类封装固定流程（如连接获取、资源释放），用户实现核心逻辑                       | JdbcTemplate、RestTemplate、RedisTemplate |
| 5    | 依赖注入       | Dependency Injection | 控制反转的一种实现，将依赖从外部注入而非内部创建                   | Spring 核心机制，通过构造函数、字段或 Setter 注入依赖                                       | @Autowired 注解注入 Service/Repository |
| 6    | 观察者模式     | Observer Pattern   | 一对多依赖，当主体状态变化时自动通知所有观察者                       | 事件发布/订阅机制                                                                          | ApplicationEventPublisher + @EventListener |
| 7    | **策略模式**       | Strategy Pattern   | 定义一系列算法并封装，使它们可以相互替换                             | 根据条件选择不同实现类（如不同支付方式、缓存策略）                                         | 将多个策略 Bean 放入 Map，根据类型动态选择 |
| 8    | 适配器模式     | Adapter Pattern    | 将不兼容的接口转换为客户端期望的接口                                 | Spring MVC 中的 HandlerAdapter 将不同类型的 Handler（如 @Controller）适配到统一处理流程     | WebMvcConfigurer 自定义适配    |
| 9    | 装饰者模式     | Decorator Pattern  | 动态地给对象添加额外职责                                             | 某些 Wrapper 类或增强器（如事务装饰、缓存装饰）                                             | 数据源切换、ResponseBodyAdvice 等 |

---

### 16. Spring Boot 一个接口多个实现类如何依赖注入？

#### 一、 指定 Bean 名称注入（@Qualifier）
- 核心：通过 @Qualifier 指定目标实现类的 Bean 名称，是最常用的基础方式

用法 1：字段注入（简洁，推荐小项目 / 非核心代码）
```java
@Service
public class OrderService {
    // @Qualifier 指定 Bean 名称（与实现类的默认名称/自定义名称匹配）
    @Autowired
    @Qualifier("wechatPayService") 
    private PayService payService;

    public void checkout(double amount) {
        payService.pay(amount);
    }
}
```

用法 2：构造器注入（推荐，符合 Spring 最佳实践，便于测试）
```java
@Service
public class OrderService {
    private final PayService payService;

    // 构造器注入 + @Qualifier
    @Autowired
    public OrderService(@Qualifier("alipayService") PayService payService) {
        this.payService = payService;
    }

    public void checkout(double amount) {
        payService.pay(amount);
    }
}
```

扩展：自定义 Bean 名称（避免默认名称冲突）
```java
// 自定义 Bean 名称为 "wxPay"
@Service("wxPay")
@Order(1) // 指定优先级，数字越小越优先
public class WechatPayService implements PayService { ... }

// 注入时匹配自定义名称
@Autowired
@Qualifier("wxPay")
private PayService payService;
```

#### 二、 指定主实现类（@Primary）
- 核心：给某一个实现类标记 @Primary，当接口注入时无明确指定，默认注入该实现类

```java
// 标记为默认实现类
@Service
@Primary 
public class AlipayService implements PayService { ... }

// 注入时无需 @Qualifier，默认注入 AlipayService
@Service
public class OrderService {
    @Autowired
    private PayService payService; // 自动注入 AlipayService

    public void checkout(double amount) {
        payService.pay(amount); // 输出：支付宝支付
    }
}
```
⚠️ 注意：一个接口只能有一个 @Primary 实现类，否则仍会报 “无唯一 Bean” 异常。

#### 三、 方式 3：按类型批量注入（List/Map）
- 核心：将接口的所有实现类注入到 List/Map 中，适用于 “动态选择实现类” 场景（如根据参数切换支付方式）

```java
@Service
public class OrderService {
    // 注入所有 PayService 实现类：Key=Bean 名称，Value=实现类实例
    @Autowired
    private Map<String, PayService> payServiceMap;

    // 或注入到 List：按 Bean 加载顺序存储 @Order(1)
     @Autowired
     private List<PayService> payServiceList;

    public void checkout(double amount, String payType) {
        // 根据支付类型动态获取实现类
        PayService payService = switch (payType) {
            case "wechat" -> payServiceMap.get("wechatPayService");
            case "alipay" -> payServiceMap.get("alipayService");
            default -> throw new IllegalArgumentException("不支持的支付类型");
        };
        payService.pay(amount);
    }
}

// 调用示例
// orderService.checkout(100.0, "wechat"); → 微信支付
// orderService.checkout(100.0, "alipay"); → 支付宝支付
```

#### 四、 通过配置类手动注册 Bean（@Bean）
- 核心：在配置类中手动定义 Bean，指定名称和实现类，适用于 “第三方实现类”“需自定义初始化逻辑” 的场景

```java
// 配置类
@Configuration
public class PayConfig {
    // 手动注册 WechatPayService，指定 Bean 名称为 "myWechatPay"
    @Bean("myWechatPay")
    public PayService wechatPayService() {
        // 可自定义初始化逻辑（如设置参数、依赖其他 Bean）
        return new WechatPayService();
    }

    // 手动注册 AlipayService，指定 Bean 名称为 "myAlipay"
    @Bean("myAlipay")
    public PayService alipayService() {
        return new AlipayService();
    }
}

// 注入时指定手动注册的 Bean 名称
@Service
public class OrderService {
    @Autowired
    @Qualifier("myWechatPay")
    private PayService payService;
}
```

#### 五、 使用 @Resource 按名称注入（JDK 注解，替代 @Autowired + @Qualifier）
- 核心：@Resource 是 JDK 自带注解，默认按 Bean 名称注入，无需搭配 @Qualifier，更简洁

```java
@Service
public class OrderService {
    // @Resource(name = "Bean 名称") 按名称注入
    @Resource(name = "wechatPayService")
    private PayService payService;

    // 若省略 name，默认按字段名匹配 Bean 名称（如字段名是 wechatPayService，则匹配同名 Bean）
    // @Resource
    // private PayService wechatPayService;
}
```

#### 六、 动态注入（ApplicationContext 手动获取）
- 核心：通过 ApplicationContext 手动获取 Bean，适用于 “运行时动态选择” 且无法提前注入的场景（如工具类、非 Spring 管理的类）

```java
@Service
public class OrderService implements ApplicationContextAware {
    private ApplicationContext applicationContext;

    // 实现 ApplicationContextAware，Spring 自动注入 ApplicationContext
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    public void checkout(double amount, String payType) {
        // 手动获取 Bean
        PayService payService = switch (payType) {
            case "wechat" -> applicationContext.getBean("wechatPayService", PayService.class);
            case "alipay" -> applicationContext.getBean("alipayService", PayService.class);
            default -> throw new IllegalArgumentException("不支持的支付类型");
        };
        payService.pay(amount);
    }
}
```

#### 总结
| 注入方式                | 适用场景                                   | 优先级         |
|-------------------------|--------------------------------------------|----------------|
| @Qualifier + @Autowired | 固定使用某一个实现类                       | 最高（基础通用）|
| @Primary                | 大部分场景默认用某实现类，少数场景指定其他 | 高             |
| List/Map 批量注入       | 动态切换实现类（如根据参数选择）| 高             |
| @Bean 手动注册          | 第三方实现类、需自定义初始化逻辑           | 中             |
| @Resource               | 习惯 JDK 注解，替代 @Autowired+@Qualifier  | 中             |
| ApplicationContext 手动获取 | 非 Spring 管理类、极端动态场景             | 低（尽量避免）|

---

### 17. @Resource 和 @Autowired有什么区别？
| 对比维度         | @Autowired                                                                 | @Resource                                                                 |
|------------------|----------------------------------------------------------------------------|---------------------------------------------------------------------------|
| 所属规范         | Spring 框架原生注解（org.springframework.beans.factory.annotation）        | JDK 自带注解（javax.annotation，JDK9 + 需手动引入依赖）                   |
| **注入核心规则**     | 先按类型（Type） 匹配 Bean，类型匹配失败再按名称（Name）（需配合 @Qualifier） | 先按名称（Name） 匹配 Bean，名称匹配失败再按类型（Type）                  |
| 依赖必选性       | 默认必须找到匹配 Bean，否则抛 NoSuchBeanDefinitionException（可通过 required = false 关闭） | 默认必须找到匹配 Bean，否则抛 NoSuchBeanDefinitionException（可通过 name="" 或 type=Object.class 宽松匹配） |
| 支持的注解搭配   | 需配合 @Qualifier 指定 Bean 名称；配合 @Primary 解决类型冲突                | 无需额外注解，直接通过 name 属性指定 Bean 名称                            |
| **支持的注入方式**   | 字段注入、构造器注入、方法注入（setter / 任意方法）                        | 字段注入、setter 方法注入（构造器注入不支持）                             |
| **泛型注入支持**     | 支持泛型类型匹配（如 List<PayService> 注入所有 PayService 实现类）          | 泛型支持弱（仅能按名称 / 原始类型匹配）                                   |
| 属性配置         | 仅支持 required（是否必须注入）                                            | 支持 name（指定 Bean 名称）、type（指定 Bean 类型）                       |
| 循环依赖处理     | 与 Spring 容器一致，支持字段注入的循环依赖（构造器注入不支持）              | 同 Spring 容器规则，依赖 Spring 底层处理                                |

---

### 18. 什么是循环依赖？如何解决循环依赖？

- 循环依赖**指两个 / 多个 Bean 互相依赖对方（如 A→B，B→A），导致容器初始化 Bean 时陷入 “先有鸡还是先有蛋” 的死循环**；Spring 仅通过「**三级缓存**」解决单例 Bean 的构造器之外的循环依赖，**多例 Bean / 构造器注入的循环依赖无法解决**。

Spring 解决循环依赖的核心是「提前暴露未完成初始化的 Bean 实例」，通过三级缓存实现，这是 Spring 最核心的源码考点之一。

#### 1. 三级缓存的定义（DefaultSingletonBeanRegistry 中）
| 缓存级别 | 名称                | 类型                     | 作用                                                                 |
|----------|---------------------|--------------------------|----------------------------------------------------------------------|
| 一级缓存 | singletonObjects    | Map<String, Object>      | 存储完全初始化完成的单例 Bean（最终可用的实例）|
| 二级缓存 | earlySingletonObjects | Map<String, Object>    | 存储提前暴露的未完成初始化的 Bean（已实例化，未属性注入 / 初始化）|
| 三级缓存 | singletonFactories  | Map<String, ObjectFactory<?>> | 存储 Bean 的创建工厂，用于生成提前暴露的 Bean 实例（解决 AOP 代理问题） |

#### 2. 三级缓存解决循环依赖的完整流程（A→B→A）
- 初始化 A：
  - 先 new 出 A 的空对象（实例化），但还没注入 B、没执行 @PostConstruct；
  - **把 A 的 “创建工厂” 丢到三级缓存，目的是 “提前暴露 A 的引用”**；
  - 开始给 A 注入 B，发现 B 还没创建，转去创建 B。
- 初始化 B：
  - 先 new 出 B 的空对象，把 B 的工厂丢到三级缓存；
  - 开始给 B 注入 A，此时去查缓存：
  - 一级缓存（完成的 A）：无；
  - 二级缓存（早期 A）：无；
  - **三级缓存（A 的工厂）：有 → 用工厂生成 A 的早期实例，丢到二级缓存，删除三级缓存；**
  - 把二级缓存里的 “早期 A” 注入给 B；
  - B 完成初始化（注入 A + 执行 @PostConstruct），丢到一级缓存。
- 回到初始化 A：
  - 从一级缓存拿到完成的 B，注入给 A；
  - **A 完成初始化，丢到一级缓存，删除二级缓存里的早期 A。**

#### 3. 为什么需要三级缓存？（核心是解决 AOP 代理）

如果 Bean 需要生成 AOP 代理（如加了 @Transactional），三级缓存的 ObjectFactory 会在 “提前暴露” 时生成代理对象，而非原始对象 —— 这样注入给依赖方的是代理对象，而非原始对象，保证 AOP 生效。

如果只有二级缓存，无法动态生成代理对象，会导致循环依赖场景下 AOP 失效。

> 新手最容易困惑的点：为什么提前暴露的是代理对象，而不是原始对象？
>  - 核心在 getEarlyBeanReference 方法：Spring 会遍历所有 BeanPostProcessor，其中 AbstractAutoProxyCreator（AOP 代理创建器）会在这里判断当前 Bean 是否需要代理：
>  - 如果需要（比如有 @Transactional）：立即生成代理对象并返回；
>  - 如果不需要：返回原始对象。
> 
> 这样，注入给 B 的 A 是代理对象，而非原始对象 → 当 B 调用 A 的方法时，会触发代理的增强逻辑（比如事务），保证 AOP 生效。

---

### 19. Spring 框架与 Java 原生开发的深度理解

本质是「**手动搭建轮子**」与「**使用成熟封装框架**」的区别，**核心在于「生产力、规范性、可维护性」的权衡** —— 原生开发是基础，保证灵活性和底层可控性；Spring 框架是原生开发的高度抽象与封装，专注于提升企业级应用的开发效率和工程化能力。

#### 一. 什么是 Java 原生开发？什么是 Spring 开发？

###### 1. Java 原生开发（纯 Java 开发）

指不依赖任何第三方框架（仅依赖 JDK 核心 API），直接使用 Java 基础语法、集合、IO、多线程、反射、JDBC 等原生 API 进行开发的方式。

- 核心特点：无额外封装，直接操作底层 API，所有逻辑（如对象创建、资源管理、流程控制）都需要开发者手动实现

###### 2. Spring 框架开发
指**基于 Spring 生态（Spring Core、Spring Boot、Spring Cloud 等）** 进行开发，依托 Spring 提供的「IoC 容器、AOP、声明式事务、自动配置」等核心能力，简化开发流程的方式。

- 核心特点：站在原生 Java 的基础上，对常用的企业级开发场景进行高度封装、抽象和自动化，开发者无需关注底层实现细节，专注于业务逻辑

#### 二. 核心差异：Spring 框架 vs Java 原生开发（10 大关键维度）

| 对比维度                     | Java 原生开发                                                                 | Spring 框架开发                                                                 |
|------------------------------|------------------------------------------------------------------------------|--------------------------------------------------------------------------------|
| 核心思想                     | 手动实现、底层可控、灵活自由                                                 | 约定优于配置、依赖注入、面向切面、自动化封装                                   |
| 对象管理                     | 手动new创建对象，管理对象生命周期（创建、销毁、依赖），耦合度高               | 由IoC 容器统一管理 Bean 的创建、依赖注入、生命周期，实现对象解耦               |
| 业务解耦                     | 硬编码依赖，如UserService userService = new UserServiceImpl()，修改实现类需改动大量代码 | 基于接口编程 + 依赖注入，通过@Autowired注入接口，修改实现类仅需配置调整，耦合度极低 |
| 事务处理                     | 手动编写 JDBC 事务：conn.setAutoCommit(false)，try-catch中手动commit()/rollback()，代码冗余，易出错 | 用@Transactional声明式事务，一行注解实现事务管控，底层自动完成提交 / 回滚，支持多种事务传播特性 |
| 横切逻辑处理（日志、权限、监控） | 硬编码嵌入业务逻辑中，如每个方法开头写日志，代码冗余，难以维护               | 基于AOP（面向切面编程），将横切逻辑抽离为切面，与业务逻辑解耦，统一管控，无需侵入业务代码 |
| 资源管理（数据库、连接池、MQ） | 手动创建和关闭资源（如Connection、Socket），易出现资源泄露（忘记关闭）         | 内置资源管理机制，配合第三方组件（Druid、RabbitMQ）实现资源池化、自动关闭，避免资源泄露 |
| 开发效率                     | 低，大量重复代码（如 JDBC、Servlet 配置），专注于底层实现，而非业务           | 极高，Spring Boot 自动配置消除 90% 的冗余配置，注解式开发简化代码，开发者专注于业务逻辑，提升数倍开发效率 |
| 工程化与可维护性             | 差，无统一规范，全靠开发者个人编码习惯，大型项目易出现「代码混乱」，难以迭代和维护 | 强，有统一的开发规范和生态标准，代码结构清晰，横切逻辑与业务逻辑分离，大型项目（微服务、分布式）易迭代、易维护 |
| 生态支持                     | 仅依赖 JDK，无额外生态，应对复杂场景（微服务、分布式事务）需手动搭建轮子       | 拥有完善的生态体系（Spring Boot、Spring Cloud、Spring Data、Spring Security），覆盖所有企业级场景，开箱即用 |
| 底层可控性                   | 极高，开发者可以精准控制每一行代码的执行逻辑，便于调试底层问题               | 较低，Spring 封装了底层实现，出现问题时需要理解 Spring 的底层原理（如 IoC 容器、AOP 动态代理），难以直接调试底层代码 |

---


















