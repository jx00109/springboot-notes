# Springboot-Notes  

1. **@RestController和@Controller的区别**  
    首先我使用**@RestController**注解编写了UserController.java
    ```java  
    @RestController
    public class UserController {
        
        @RequestMapping(path = "/")
        public String index() {
            return "index";
        }

        @GetMapping(path = "/greet")
        public String greet() {
            return "hello";
        }

        @RequestMapping(path = "/login")
        public String login() {
            return "login";
        }
    }
    ```  
    启动服务后，通过**localhost:8080/** 无法访问到 **/resources/templates/index.html**  
    只在页面上直接显示了**index**。  
    修改使用 **@Controller** 后问题得到解决。  
    
    **@RestController注解是@ResponseBody + @Controller的组合** 。  
    ```text
    @ResponseBody注解的作用是将controller的方法返回的对象通过适当的转换器转换为指定的格式之后，写入到response对象的body区，
    通常用来返回JSON数据或者是XML数据，需要注意的呢，在使用此注解之后不会再走试图处理器，而是直接将数据写入到输入流中，效果等同于通过response对象输出指定格式的数据。  
    ```  
    
2. 前端无法得到Spring Security抛出的UsernameNotFoundException  

    login.html
    ```html
    <!DOCTYPE html>
    <html xmlns="http://www.w3.org/1999/xhtml"
          xmlns:th="http://www.thymeleaf.org">
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
        <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
        <title>登录页面</title>
    <body>
    <form th:action="@{/login.do}" method="post" id="loginForm">
        <span th:if="${param.error}" th:text="${session.SPRING_SECURITY_LAST_EXCEPTION.message}"></span>
    
        用户名：<input type="text" name="username" class="username" id="username" placeholder="用户名" autocomplete="off"/> <br>
        密　码：<input type="password" name="password" class="password" id="password" placeholder="密码"
                   oncontextmenu="return false"
                   onpaste="return false"/> <br>
        <input type="checkbox" name="remember-me"/>记住我 <br>
        <input id="submit" type="submit" value="登录"/>
    </form>
    </body>
    </html>
    ```  
    **session.SPRING_SECURITY_LAST_EXCEPTION.message**的值始终为**Bad Credentials**  
    调试后发现，在源码中，捕获了UsernameNotFoundException后，由于**hideUsernameNotFoundException = true**对其进行了处理，最后抛出了BadCredentialsException  
    
3. Spring Security中**WebSecurityConfigurerAdapter**的一些理解偏差  
    ```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("/", "/css/**", "/js/**", "/fonts/**", "/users").permitAll() // 都可以访问
                .antMatchers("/h2-console/**").permitAll() // 都可以访问
                .antMatchers("/admin/**").hasRole("ADMIN") // 需要相应的角色才能访问
                .antMatchers("/console/**").hasAnyRole("ADMIN", "USER") // 需要相应的角色才能访问
                .and()
                .formLogin()   //基于 Form 表单登录验证
                .loginPage("/login").loginProcessingUrl("/login.do").failureUrl("/login?error=true") // 自定义登录界面
                .and().rememberMe().key(KEY) // 启用 remember me
                .and().exceptionHandling().accessDeniedPage("/403")// 处理异常，拒绝访问就重定向到 403 页面
                .and().logout().permitAll()
                .and().csrf().disable();
        http.headers().frameOptions().sameOrigin(); // 允许来自同一来源的H2 控制台的请求
    }
    ```  
    loginPage():当用户访问不允许直接访问的页面时需要跳转的登录页  
    loginProcessingUrl():用于处理用户登录信息的Url(**不是页面**)  
    successForwardUrl():登录成功后跳转的url，需要再写controller进行捕获并返回页面，不然会404  
    
    