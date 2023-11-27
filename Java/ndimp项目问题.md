# ndimp项目
[toc]
## 出现的问题
### core 模块中的 @configuration 不生效
经排查，发现由于各模块启动类的位置错误，导致配置无法生效
>如：servic-course的启动类所在地址为 com.jmu.ndimp.course 但 core的配置文件地址为 com.jmu.ndimp.config
> 
> 他们所处的文件夹不同所以启动类无法获取到 config 下的所有配置
> 
> 应更改启动类地址为 com.jmu.ndimp 此时启动类可以获取到当前目录和子目录的所有配置
> 
### knife4j 无法获取到接口列表，配置的基本信息也无法显示
在knife4j配置中修改DocumentationType.SWAGGER_2为OAS_30
```java
@Configuration
@EnableKnife4j
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.OAS_30)
                .groupName("1.0版本")
                .useDefaultResponseMessages(false)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.jmu"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .description("knife4j在线API接口文档")
                .contact(new Contact("叶玮", "", "1253578128@qq.com"))
                .version("v1.0.0")
                .title("knife4j在线API接口文档")
                .build();
    }
}
```

### 拦截器 LoginInterceptor 中 redisService 无法自动装配 @Autowired 注解无法生效
原因：只有在 spring boot 的 Bean 下才能自动注入
解决：
在 LoginInterceptor 中添加 @Component 注解
在 WebConfig 中更改原来的 new 方法创建拦截器的方式，使用 @Autowired 自动注入 LoginInterceptor

LoginInterceptor:
```java
@Component
public class LoginInterceptor implements HandlerInterceptor {
    @Autowired
    RedisService redisService;
```
WebConfig:
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    LoginInterceptor loginInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loginInterceptor)
                .addPathPatterns("/**")
                .excludePathPatterns("/user/login")
                //swagger 文档资源的放行
                .excludePathPatterns("/swagger-resources/**", "/webjars/**", "/v3/**", "/swagger-ui/**","/doc.html","/error");
    }
}
```

### Required request body is missing
所有的list接口方法都报此错误
经查看发现所有list都使用的GetMapping，更改为PostMapping即可

### mybatis-plus方法无法绑定
#### TeacherMapper
出现以下报错
```
 Invalid bound statement (not found): com.jmu.ndimp.teacher.mapper.TeacherMapper.listTeacher
```

解决过程：
1. 查看target发现不存在xml文件
解决：
在pom文件中引入以下代码
```xml
<!--    防止target中漏掉xml文件-->
    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <filtering>true</filtering>
                <includes>
                    <include>**/*</include>
                </includes>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.xml</include>
                </includes>
            </resource>
        </resources>
    </build>
```

错误依旧存在

2. 再次查看target发现文件层级出现错误
mapper.xml文件与mapper接口不在一个文件，单独出现了一个com.jmu.ndimp.teacher.mapper文件夹
![屏幕截图 2023-09-17 153038](/assets/屏幕截图%202023-09-17%20153038.png)
mapper.xml文件与mapper接口不在一个文件
解决：
修改xml文件目录即可
![屏幕截图 2023-09-17 154845](/assets/屏幕截图%202023-09-17%20154845.png)
![屏幕截图 2023-09-17 154640](/assets/屏幕截图%202023-09-17%20154640.png)

结果：
![屏幕截图 2023-09-17 155146](/assets/屏幕截图%202023-09-17%20155146.png)
#### GradeMapper
出现以下报错
```
 Invalid bound statement (not found): com.jmu.ndimp.grage.mapper.GradeMapper.listGrade
```
GradeMapper被放在了dao下，更改文件夹名为mapper即可