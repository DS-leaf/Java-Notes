# Spring
## Spring Boot
### 为什么要使用SpringBoot
+ 简化配置，SpringBoot的核心思想就是约定大于配置，省去xml配置
+ 简化部署，内置Tomcat，可以快速启动应用，以jar包的形式
+ 简化依赖，开发web应用，我们只需要在pom文件中添加如下一个 starter-web 依赖即可

### Springboot的核心注解是哪个?它主要由哪几个注解组成
@SpringBootApplication主要有三部分组成：
+ @SpringBootConfiguration：声明当前类是SpringBoot应用的配置类，这个注解里面包括了@Configuration注解（全村就一个，其他注解都用@Configuration来修饰）
+ @EnableAutoConfiguration：开启自动配置
+ @ComponentScan：扫描Spring组件，通过basePackageClasses或者basePackages属性来指定要扫描包的路径，没有指定的话，就扫描当前包以及子包

### 如何理解springboot中的starters,常用的有哪些？
stater机制帮我们完成了项目启动所需要的的相关jar包，不用去一个个找jar包pom了，一键式。
比如说我想使用mybatis，直接在pom文件中加mybatis-spring-boot-starter就可以了，非常方便

**常用的starter**：

+ spring-boot-starter-web 嵌入tomcat和web开发的支持
+ spring-boot-starter-data-jpa 数据库支持
+ spring-boot-starter-data-redis redis数据库支持
+ mybatis-spring-boot-starter 第三方的mybatis集成starter

### 如何在SpringBoot启动的时候运行一些特定的代码？
可以实现接口ApplicationRunner或者CommandLineRunner，两个接口都一样，实现run方法就可以

### 如何使用Springboot实现异常处理？
1. 在Controller方法中使用 @ExceptionHandler 处理**局部异常**
    ```java
    @RestController
    public class UserController {

        @GetMapping("/users/{id}")
        public User getUser(@PathVariable int id) {
            User user = userRepository.findById(id);
            if (user == null) {
                throw new UserNotFoundException("User not found");
            }
            return user;
        }

        @ExceptionHandler(UserNotFoundException.class)
        public ResponseEntity<ErrorResponse> handleUserNotFoundException(UserNotFoundException ex) {
            ErrorResponse error = new ErrorResponse(HttpStatus.NOT_FOUND.value(), ex.getMessage());
            return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
        }
    }

    ```
2. 使用@ControllerAdvice + @ExceptionHandler 注解处理全局异常（开发中常用）
    ```java
    //可以使Spring自动把要返回的对象转化成json文本写入到响应体中，比如自定义的ResultBean
    @ResponseBody
    @ControllerAdvice
    public class MyGlobalExceptionHandler 
    {
        // 专门用来捕获和处理Controller层的异常
        @ExceptionHandler(Exception.class)
        public ModelAndView customException(Exception e) 
        {
            ModelAndView mv = new ModelAndView();
            mv.addObject("message", e.getMessage());
            mv.setViewName("myerror");
            return mv;
        }
    
        // 专门用来捕获和处理Controller层的空指针异常
        @ExceptionHandler(NullPointerException.class)
        public ModelAndView nullPointerExceptionHandler(NullPointerException e)
        {
            ModelAndView mv = new ModelAndView(new MappingJackson2JsonView());
            mv.addObject("success",false);
            mv.addObject("mesg","请求发生了空指针异常，请稍后再试");
            return mv;
       }
    }
    ```

### Spring Boot多线程

Spring/Spring Boot只需要在配置类上注解“@EnableAsync”，在需要使用单独线程的方法上使用“@Async”注解即可。

Spring Boot：
```java
@Configuration
@EnableAsync
public class AsyncConfiguration {

    @Bean("doSomethingExecutor")
    public Executor doSomethingExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        // 核心线程数：线程池创建时候初始化的线程数
        executor.setCorePoolSize(10);
        // 最大线程数：线程池最大的线程数，只有在缓冲队列满了之后才会申请超过核心线程数的线程
        executor.setMaxPoolSize(20);
        // 缓冲队列：用来缓冲执行任务的队列
        executor.setQueueCapacity(500);
        // 允许线程的空闲时间60秒：当超过了核心线程之外的线程在空闲时间到达之后会被销毁
        executor.setKeepAliveSeconds(60);
        // 线程池名的前缀：设置好了之后可以方便我们定位处理任务所在的线程池
        executor.setThreadNamePrefix("do-something-");
        // 缓冲队列满了之后的拒绝策略：由调用线程处理（一般是主线程）
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.DiscardPolicy());
        executor.initialize();
        return executor;
    }

}
```
使用的方式非常简单，在需要异步的方法上加@Async注解
```java
@RestController
public class AsyncController {

    @Autowired
    private AsyncService asyncService;

    @GetMapping("/open/something")
    public String something() {
        int count = 10;
        for (int i = 0; i < count; i++) {
            asyncService.doSomething("index = " + i);
        }
        lon
        return "success";
    }
}


@Slf4j
@Service
public class AsyncService {

    // 指定使用beanname为doSomethingExecutor的线程池
    @Async("doSomethingExecutor")
    public String doSomething(String message) {
        log.info("do something, message={}", message);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            log.error("do something error: ", e);
        }
        return message;
    }
}
```

### Spring Boot获取异步方法返回值
当异步方法有返回值时，如何获取异步方法执行的返回结果呢？这时需要异步调用的方法带有返回值CompletableFuture。
```java
@RestController
public class AsyncController {

    @Autowired
    private AsyncService asyncService;

    @SneakyThrows
    @ApiOperation("异步 有返回值")
    @GetMapping("/open/somethings")
    public String somethings() {
        CompletableFuture<String> createOrder = asyncService.doSomething1("create order");
        CompletableFuture<String> reduceAccount = asyncService.doSomething2("reduce account");
        CompletableFuture<String> saveLog = asyncService.doSomething3("save log");

        // 等待所有任务都执行完
        CompletableFuture.allOf(createOrder, reduceAccount, saveLog).join();
        // 获取每个任务的返回结果
        String result = createOrder.get() + reduceAccount.get() + saveLog.get();
        return result;
    }
}


@Slf4j
@Service
public class AsyncService {

    @Async("doSomethingExecutor")
    public CompletableFuture<String> doSomething1(String message) throws InterruptedException {
        log.info("do something1: {}", message);
        Thread.sleep(1000);
        return CompletableFuture.completedFuture("do something1: " + message);
    }

    @Async("doSomethingExecutor")
    public CompletableFuture<String> doSomething2(String message) throws InterruptedException {
        log.info("do something2: {}", message);
        Thread.sleep(1000);
        return CompletableFuture.completedFuture("; do something2: " + message);
    }

    @Async("doSomethingExecutor")
    public CompletableFuture<String> doSomething3(String message) throws InterruptedException {
        log.info("do something3: {}", message);
        Thread.sleep(1000);
        return CompletableFuture.completedFuture("; do something3: " + message);
    }
}
```