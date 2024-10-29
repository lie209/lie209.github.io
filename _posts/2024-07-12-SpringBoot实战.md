---
layout: post
title: SpringBoot实战
date: 2024-7-12
catalog: true
tags: [Java]
---

# SpringBoot总结

## 编写接口流程

### 环境搭建

- 执行资料中的big_event.sql脚本，准备数据库表

- 创建springboot工程，引入对应的依赖（web、mybatis、mysql驱动）

  ```xml
      <dependencies>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter</artifactId>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-validation</artifactId>
              <version>3.1.5</version>
          </dependency>
          <dependency>
              <groupId>org.mybatis.spring.boot</groupId>
              <artifactId>mybatis-spring-boot-starter</artifactId>
              <version>3.0.3</version>
          </dependency>
          <dependency>
              <groupId>com.mysql</groupId>
              <artifactId>mysql-connector-j</artifactId>
              <version>8.0.33</version>
          </dependency>
  <!--        lombok编译时自动给实体类添加getter和setter方法-->
          <dependency>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
          </dependency>
      </dependencies>

- 配置文件application.yml中引入mybatis的配置信息

  ```xml
   datasource:
     driver-class-name: com.mysql.cj.jdbc.Driver
     url: jdbc:mysql://localhost:3306/big_event
     username: root
     password: xxxxxx
  ```

- 创建包结构

  ```mermaid
  graph LR
      A[Controller] --> B[Service]
      B --> C[Mapper]
      C-->D[pojo]
  ```

- 添加实体类，实体类是对数据库表中的映射

  ```java
  public class User {
      private Integer id;//主键ID
      private String username;//用户名
      private String password;//密码
      private String nickname;//昵称
      private String email;//邮箱
      private String userPic;//用户头像地址
      private LocalDateTime createTime;//创建时间
      private LocalDateTime updateTime;//更新时间
  }
  ```

  > 为了方便快速构建Setter和Getter方法，可以使用`lombok`来自动化构建
  >
  > - 引入依赖
  >
  >   ```xml
  >   <!--        lombok编译时自动给实体类添加getter和setter方法-->
  >           <dependency>
  >               <groupId>org.projectlombok</groupId>
  >               <artifactId>lombok</artifactId>
  >           </dependency>
  >   ```
  >
  > - 添加注解
  >
  >   ```java
  >   //lombok自动生成构造方法
  >   @NoArgsConstructor
  >   @AllArgsConstructor
  >   @Data
  >   //自动生成setter,getter方法
  >   public class Result<T> {...}
  >   ```

### 业务逻辑编写

- **对于Controller**

  > Controller中的return是返回给浏览器的

  - 添加`@RestController`注解

  - 使用`@Mapping`来指定请求范围
  
    ```java
    @RestController
    @RequestMapping("/user")
    public class UserController
    {
        @Autowired
        private UserService userService;
        @PostMapping("/register")
        public Result registerUser(String username,String password)
        {
            User u=userService.findUserByUsername(username);
            if(u==null)
            {
                userService.register(username,password);
                return Result.success();
            }
            else
            {
                return Result.error("用户名已存在");
            }
    
    
        }
    }
    ```

**Controller中的具体业务逻辑要调用Service中的方法进行实现**

- **对于Service**

  - 创建接口

    ```java
    public interface UserService {
        User findUserByUsername(String username);
        void register(String username, String password);
    }
    ```

  - 编写实现类，实现类位于`service`目录下的`impl`子目录中，使用`@Service`注解来标记Service的实现类，使用`@Autowired`注解来标记注入用于操作数据库的Mapper

    ```java
    @Service
    public class UserServiceImpl implements UserService {
        @Autowired
        private UserMapper userMapper;
        @Override
        public User findUserByUsername(String username)
        {
            User u = userMapper.findUserByUsername(username);
            return u;
        }
        @Override
        public void register(String username, String password)
        {
            //密码加密
            String md5Pwd =Md5Util.getMD5String(password);
            //添加
            userMapper.add(username,md5Pwd);
    
        }
    }
    ```

- **对于Mapper**

  Service使用`Mapper`来对底层数据库进行操作，使用`@Mapper`注解来标记Mapper，使用`@Select`、`@Insert`等注解对数据库进行增删改查操作

  ```java
  @Mapper
  public interface UserMapper
  {
      @Select("select * from user where username = #{username}")
      User findUserByUsername(String username);
  
      @Insert("insert into user(username, password,create_time,update_time) values(#{username}, #{md5Pwd},now(),now())")
      void add(String username, String md5Pwd);
  }
  ```

## 参数校验

Spring 提供的一个参数校验框架，使用预定义的注解完成参数校验

### 基础流程

- 引入Spring Validation 起步依赖

  ```xml
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-validation</artifactId>
              <version>3.1.5</version>
          </dependency>
  ```

- 在参数前面添加`@Pattern`注解

  ```java
      public Result registerUser(@Pattern(regexp = "^[a-zA-Z0-9_-]{4,16}$",message = "用户名格式不正确") String username, @Pattern(regexp = "^[a-zA-Z0-9]{6,16}$",message = "密码格式不正确")String password)
      {...}
  ```

- 在Controller类上添加`@Validated`注解

  ```java
  @RestController
  @RequestMapping("/user")
  @Validated
  public class UserController
  {
      @Autowired
      private UserService userService;
      @PostMapping("/register")
      public Result registerUser(@Pattern(regexp = "^[a-zA-Z0-9_-]{4,16}$",message = "用户名格式不正确") String username, @Pattern(regexp = "^[a-zA-Z0-9]{6,16}$",message = "密码格式不正确")String password)
      {
          ...
      }
  }
  ```

### 对于实体类的参数校验

- 在实体类对象的定义中使用`@NotNUll`、`@NotEmpty`、`@Email`、`@Pattern`等注解来限定参数

  > 此时仅在校验User对象的时候生效，如果Controller中传入的是普通的参数，则需要在参数处手动添加对应的校验注解
  >
  > ```java
  >     @PatchMapping("/updateAvatar")
  >     public Result updateAvatar(@RequestParam @URL(message = "头像地址格式不正确") String avatar)
  >     {
  >         userService.updateAvatar(avatar);
  >         return Result.success();
  >     }
  > ```

  ```java
  @Data
  public class User {
      @NotNull
      private Integer id;//主键ID
      private String username;//用户名
      private String password;//密码
      @NotEmpty
      @Pattern(regexp = "^[a-zA-Z0-9_-]{1,16}$",message = "昵称格式不正确")
      private String nickname;//昵称
      @Email
      private String email;//邮箱
  }
  ```

- 在Controller方法中，在需要校验的对象前面添加`@Validated`注解，来使实体类中定义的参数校验规则生效

  ```java
      @PutMapping("/update")
      public Result updateUser(@RequestBody @Validated User user)
      {
          userService.update(user);
          return Result.success();
      }
  ```

### 分组校验

对于有些情况，例如，接口1的请求值中的id可以为空，接口2却要求请求值中的id不能为空：
```java
    @PostMapping("/add")
    public Result addCategory(@RequestBody @Validated Category category)
    {
        categoryService.addCategory(category);
        return Result.success();
    }

    @PutMapping("/update")
    public Result updateCategory(@RequestBody @Validated Category category)
    {
        Boolean b = categoryService.udateCategory(category);
        return b?Result.success():Result.error("更新失败");

    }
```

这种情况下就会产生冲突，因为在Category实体类中，id被注解`@NotNull`所修饰了

```java
@Data
public class Category {
    @NotNull
    private Integer id;//主键ID
```

---

这时候就需要使用**分组校验**

1. 定义分组

   ```java
   @Data
   public class Category {
       @NotNull
       private Integer id;
       @NotEmpty
       private String categoryName;
       
       public interface Add{}
       public interface Update{}
   }
   ```

2. 定义校验项时指定归属的分组，指定什么操作的时候需要校验这个项

   ```java
   @Data
   public class Category {
       @NotNull(groups = {Update.class})
       private Integer id;
       @NotEmpty(groups = {Add.class, Update.class})
       private String categoryName;
   
       public interface Add{}
       public interface Update{}
   }
   ```

3. 校验时指定要校验的分组

   ```java
       @Autowired
       private CategoryService categoryService;
       @PostMapping("/add")
       public Result addCategory(@RequestBody @Validated(Category.Add.class) Category category)
       {
           categoryService.addCategory(category);
           return Result.success();
       }
   
       @PutMapping("/update")
       public Result updateCategory(@RequestBody @Validated(Category.Update.class) Category category)
       {
           Boolean b = categoryService.udateCategory(category);
           return b?Result.success():Result.error("更新失败");
       }
   ```

> **定义校验项时如果没有指定分组，则属于Default分组，分组可以继承**，A extends B，则A拥有B中所有的校验项
>
> 以上代码就能这样改写成这样：
> ```java
> @Data
> public class Category {
>     @NotNull(groups = {Update.class})
>     private Integer id;
>     //没有指定分组，默认Default就会校验这个变量
>     @NotEmpty
>     private String categoryName;
>     @NotEmpty
>     private String categoryAlias;
> 	//Add和Update用有Default分组中所有的校验项
>     public interface Add extends Default {}
>     public interface Update extends Default{}
> }
> ```
>

### 自定义校验

有的注解不能满足所有的校验需求，特殊的情况需要自定义校验（自定义校验注解）

1. 新建软件包`anno`，新建自定义注解State

   ![image-20241021151601908](https://s2.loli.net/2024/10/21/8Gn4ePl9Ap7qic3.png)

   ```java
   //元注解,标记注解可以抽取到帮助文档中
   @Documented
   // 标记谁给注解提供校验规则
   @Constraint(
           validatedBy = {StateValidation.class}
   )
   //元注解，标记注解可以用在哪些地方
   @Target(FIELD)
   //元注解，注解在运行时保留
   @Retention(RetentionPolicy.RUNTIME)
   public @interface State {
       //提供校验失败的提示信息
       String message() default "State的参数只能是已发布或者草稿";
   
       //指定分组
       Class<?>[] groups() default {};
   
       //获取到State注解的附加信息
       Class<? extends Payload>[] payload() default {};
   }
   ```

2. 新建软件包`validation`，自定义校验数据的类StateValidation，实现ConstraintValidator接口

   ![image-20241021153939641](https://s2.loli.net/2024/10/21/UQRri7CT4lnXwZB.png)

   ```java
   //ConstraintValidator<State,String>  第一个参数是给哪个注解做校验，第二个参数是校验的参数类型
   public class StateValidation  implements ConstraintValidator<State,String> {
       @Override
       public void initialize(State constraintAnnotation) {
           ConstraintValidator.super.initialize(constraintAnnotation);
       }
       /**
        *
        * @param value 将来要校验的数据
        * @param constraintValidatorContext
        * @return 返回校验是否通过
        */
       @Override
       public boolean isValid(String value, ConstraintValidatorContext constraintValidatorContext) {
           if(value==null) {
               return false;
           }
           if( value.equals("已发布")||value.equals("草稿")) {
               return true;
           }
           return false;
       }
   }
   ```

   > 记得在State类中指定校验类
   >
   > ```java
   > // 标记谁给注解提供校验规则
   > @Constraint(
   >         validatedBy = {StateValidation.class}
   > )
   > ```

3. 在需要校验的地方使用自定义注解

   ```java
       @NotEmpty
       @State
       private String state;//发布状态 已发布|草稿
   ```

## 全局异常处理

- **在项目目录下新建exception软件包，添加全局异常处理器，允许开发者以一种集中化的方式处理应用中可能发生的多种异常情况**

  > 比如说，向浏览器返回参数校验未通过时产生的错误信息

- 使用`@RestControllerAdvice`注解来对全局异常处理器进行标注，使用`@ExceptionHandler`注解来指定异常处理方法要处理的异常

  ```java
  @RestControllerAdvice
  public class GlobalExceptionHandler
  {
      //指定要处理的异常
      @ExceptionHandler({ Exception.class })
      public Result handleException(Exception e) {
          e.printStackTrace();
          return Result.error(StringUtils.hasLength(e.getMessage())?e.getMessage():"操作失败");
      }
  }
  ```

- 将`@Pattern`注解中指定和抛出的异常信息封装到Result对象中

  ```java
  return Result.error(StringUtils.hasLength(e.getMessage())?e.getMessage():"操作失败");
  ```

## 登录认证

### 令牌验证

令牌就是一段字符串，防止非法访问

- 承载业务数据,，减少后续请求查询数据库的次数

- 防篡改，保证信息的合法性和有效性

1. 引入坐标

   ```xml
           <dependency>
               <groupId>com.auth0</groupId>
               <artifactId>java-jwt</artifactId>
               <version>4.4.0</version>
           </dependency>
   ```

2. token生成和验证

   ```xml
   @SpringBootTest
   public class JwtTest {
   
       @Test
       public void testGen()
       {
           Map<String,Object> claims = new HashMap<>();
           claims.put("username","admin");
           claims.put("id", 1);
           String token = JWT.create().withClaim("user", claims)  //添加载荷
                   .withExpiresAt(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 12)) //有效期
                   .sign(Algorithm.HMAC256("asfdsf"));//指定算法和密钥
           System.out.println(token);
       }
   
   
       @Test
       public void testParse()
       {
           String token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE3MjkxMDk2MTMsInVzZXIiOnsiaWQiOjEsInVzZXJuYW1lIjoiYWRtaW4ifX0.7FptG8W7F1oSRLOthIH-7rJmoZTtIGJSHy22ZuLQzfI";
           JWTVerifier jwtVerifier = JWT.require(Algorithm.HMAC256("asfdsf")).build();
           DecodedJWT decodedJWT = jwtVerifier.verify(token);
           Map<String, Claim> claims = decodedJWT.getClaims();
           System.out.println(claims.get("user"));
       }
   }
   ```

- 注意事项
  - JWT验证时使用的签名秘钥，必须和生成JWT令牌时使用的秘钥是配套的。
  - 如果JWT令牌解析时报错，则说明 JWT令牌被篡改 或 失效了，令牌非法。 

### 令牌使用

- 接口给浏览器返回token

  ```java
      @PostMapping("/login")
      public Result<String> login(String username, String password)
      {
          //根据用户名查询用户
          User user=userService.findUserByUsername(username);
          if(user==null)
          {
              return Result.error("用户名不存在");
          }
          else
          {
              if(Md5Util.getMD5String(password).equals(user.getPassword()))
              {
                  Map<String,Object> claims=new HashMap<String, Object>();
                  claims.put("id",user.getId());
                  claims.put("username",user.getUsername());
                  String token= JwtUtil.genToken(claims);
                  return Result.success(token);
              }
              return Result.error("密码错误");
          }
      }
  ```

- 验证token

  ```java
      @GetMapping("/list")
      public Result<String> list(@RequestHeader("Authorization") String token, HttpServletResponse response)
      {
  //        验证token
          try {
              Map<String, Object> parseToken = JwtUtil.parseToken(token);
          } catch (Exception e) {
              response.setStatus(401);
              return Result.error("token无效");
          }
          return Result.success();
      }

### 拦截器

**使用以上方法虽然可行，但如果在每个Controller中都写一次，略显繁琐，于是我们可以使用拦截器来实现登陆时令牌的验证功能**

- 创建拦截器，新建interceptors软件包，在其中创建拦截器

  ```java
  @Component
  public class LoginInterceptor implements HandlerInterceptor
  {
      @Override
      public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler)
      {
          //令牌验证
          //获得token
          String token = req.getHeader("Authorization");
          //验证token
          try {
              Map<String, Object> parseToken = JwtUtil.parseToken(token);
          } catch (Exception e) {
              res.setStatus(401);
              return false;
          }
          return true;
      }
  }

- 在SpringBoot中注册拦截器

  - 新建config软件包

  - 在其中新建WebConfig配置类，在这个配置类中对拦截器进行注册操作

  ```java
  @Configuration
  public class WebConfig implements WebMvcConfigurer
  {
      @Autowired
      private LoginInterceptor loginInterceptor;
  
      @Override
      public void addInterceptors(InterceptorRegistry registry)
      {
          //排除登录接口和注册接口
          registry.addInterceptor(loginInterceptor).excludePathPatterns("/user/login","/user/register");
      }
  }
  ```

  > 注意事项
  >
  > - 使用拦截器统一验证令牌
  > - 登录和注册接口需要放行

### 通过token获取用户信息

通过请求头中的token获得用户信息

```java
    @GetMapping("/userInfo")
    public Result<User> userInfo(@RequestHeader(name="Authorization") String token)
    {
        Map<String,Object> claims=JwtUtil.parseToken(token);
        User user=userService.findUserByUsername((String) claims.get("username"));
        return Result.success(user);
    }
```

### 通过自定义脚本自动添加请求头信息

```mermaid
graph LR
    A[ApiFox] --> B[前置操作]
    B --> C[自定义脚本]
```

使用自定义脚本中自动添加token：

```javascript
pm.request.addHeader("Authorization:eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJjbGFpbXMiOnsiaWQiOjQsInVzZXJuYW1lIjoid3NzcyJ9LCJleHAiOjE3MjkxNzQxMTh9.3iIIAnAiZYsEK99vOnE3Db6kJofOFZo4GSyWX-yuGZs");
```

### 使用ThreadLocal优化代码

ThreadLocal：**提供线程局部变量**

- 用来存取数据: set()/get()

- 使用ThreadLocal存储的数据, 线程安全

来一个用户开一个线程，Controller使用同一个键从不同的线程中获得不同的数据

- 从拦截器中获得token，存放到ThreadLocal中

  ```java
  @Component
  public class LoginInterceptor implements HandlerInterceptor
  {
      @Override
      public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler)
      {
          //令牌验证
          //获得token
          String token = req.getHeader("Authorization");
          //验证token
          try {
              Map<String, Object> claims = JwtUtil.parseToken(token);
              //存到ThreadLocal中
              ThreadLocalUtil.set(claims);
          } catch (Exception e) {
              res.setStatus(401);
              return false;
          }
          return true;
      }
  }
  ```

- Controller通过ThreadLocal获得所需的claims信息

  ```java
      @GetMapping("/userInfo")
      public Result<User> userInfo(/*@RequestHeader(name="Authorization") String token*/)
      {
          Map<String,Object> claims= ThreadLocalUtil.get();
          User user=userService.findUserByUsername((String) claims.get("username"));
          return Result.success(user);
      }
  ```

- **使用结束时需要手动清空ThreadLocalMap中存储的信息，防止内存泄漏**

  在登陆拦截器中使用`afterCompletion`方法，在Controller中的方法执行完毕，并且视图渲染完毕之后清理线程局部变量

  ```java
      @Override
      public void afterCompletion(HttpServletRequest req, HttpServletResponse res, Object handler, Exception ex)
      {
          //手动清理，防止内存泄露
          ThreadLocalUtil.remove();
      }
  ```


## 更新数据

- 前端传递过来的JSON语句，Controller使用`@RequestBody`自动转换成对应的对象

      @PutMapping("/update")
      public Result update(@RequestBody User user)
      {
          userService.update(user);
          return Result.success();
      }

- 在Mapper进行数据库操作时，通过属性映射，自动在 SQL 语句中，使用 `#{}` 占位符来引用对象的属性

      @Update("UPDATE user SET nickname = #{nickname}, email = #{email},update_time=#{updateTime} WHERE id = #{id}")
      void update(User user);

- 需要正确指定Controller的返回值

  如果不正确指定返回指定的类型，有可能会返回错误的数据

  ```java
      @GetMapping("/list")
      public Result<List<Category>> getCategory()
      {
          List<Category> categoryList = categoryService.getCategory();
          return Result.success(categoryList);
      }
  ```

## 查询数据

### Mybatis数据库中字段与实体类属性值之间的映射

查询数据要注意SQL返回的每行数据的列名称与实体类属性值没有一一对应的情况

例如数据库中的列名是`create_time`，实体类中的属性为`createTime`，这种情况，在查询时，不一一对应的属性值就会出现`null`值

```jso
{
    "code": 0,
    "message": "操作成功",
    "data": [
        {
            "id": 1,
            "categoryName": null,
            "categoryAlias": null,
            "createUser": null,
            "createTime": null,
            "updateTime": null
        },
        {
            "id": 2,
            "categoryName": null,
            "categoryAlias": null,
            "createUser": null,
            "createTime": null,
            "updateTime": null
        }
    ]
}
```

解决方法是，在`application.yml`配置文件中添加以下语句，开启驼峰映射：

```yaml
mybatis:
  configuration:
   map-underscore-to-camel-case: true
```

### 指定JSON中日期显示格式

可以在实体类定义中使用`JsonFormat`注解手动指定封装到JSON后的日期显示格式：
```java
@Data
public class Category {
    private Integer id;//主键ID
    @NotEmpty
    private String categoryName;//分类名称
    @NotEmpty
    private String categoryAlias;//分类别名
    private Integer createUser;//创建人ID
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createTime;//创建时间
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime updateTime;//更新时间
}
```

### 分页查询

当需要接口返回值为以下类型时，则需要使用动态SQL进行分页查询：
![image-20241021171036874](https://s2.loli.net/2024/10/21/MqdEHVpKzlUYN6X.png)

1. 定义PageBean实体类对象

   ```java
   //分页返回结果对象
   @Data
   @NoArgsConstructor
   @AllArgsConstructor
   public class PageBean <T>{
       private Long total;//总条数
       private List<T> items;//当前页数据集合
   }
   ```

2. 开启分页查询

   - 导入`PageHelper`依赖

     ```xml
             <dependency>
                 <groupId>com.github.pagehelper</groupId>
                 <artifactId>pagehelper-spring-boot-starter</artifactId>
                 <version>2.1.0</version>
             </dependency>
     ```
   
   - 编写Service层代码
   
     ```java
         @Override
         public PageBean<Article> list(Integer pageNum, Integer pageSize, Integer categoryId, String state) {
             //创建pageBean对象
             PageBean<Article> pageBean = new PageBean<>();
             //开启分页查询，使用PageHelper插件，会自动将pageNum和pageSize传入到sql语句中
             PageHelper.startPage(pageNum,pageSize);
             //调用Mapper
             Map<String,Object> claims= ThreadLocalUtil.get();
             Integer userId = (Integer) claims.get("id");
             List<Article> articles =articleMapper.list(userId,categoryId,state);
             // Page提供了方法，可以得到PageHelper查询后获得的总记录数和当前页数据
             Page<Article> page = (Page<Article>) articles;
             //把数据填充到pageBean中
             pageBean.setTotal(page.getTotal());
             pageBean.setItems(page.getResult());
             return pageBean;
         }
     ```
   
   - 使用配置文件编写动态Sql
   
     - 在`resources`目录下新建目录mapper
   
       ![image-20241022084532793](https://s2.loli.net/2024/10/22/eJFrxdvtNmWjq5k.png)
   
     - 新建映射配置文件，映射配置文件需要与对应Mapper接口处在同一个目录层级下，并且需要与对应接口名称相同
   
       ```xml
       <?xml version="1.0" encoding="UTF-8" ?>
       <!DOCTYPE mapper
               PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
               "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
       <!--namespace中的属性值与对应映射的类名要一致-->
       <mapper namespace="com.aqdx.bigevent.mapper.ArticleMapper">
       <!--    动态sql-->
           <select id="list" resultType="com.aqdx.bigevent.pojo.Article">
               select * from article
               <where>
       --             如果满足条件，则拼接sql
                   <if test="categoryId != null">
                       category_id = #{categoryId}
                   </if>
                   <if test="state != null">
                       and state = #{state}
                   </if>
                   and create_user = #{userId}
               </where>
           </select>
       </mapper>
       ```
   
       
   
       
   
       
