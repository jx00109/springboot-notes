# Study-Notes  

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
    
4. @GeneratedValue中strategy参数的的一些区别    
    strategy = GenerationType.IDENTITY    
    该策略下，主键的自增由mysql进行维护，**describe table_name**命令可以查看到主键有自增属性   
    
    strategy = GenerationType.AUTO    
    该策略下，主键的自增由hibernate进行维护，**describe table_name**命令可以查看到主键无自增属性   
    
5. Mysql中的数据类型  
    **在 MySQL 中，有三种主要的类型：Text（文本）、Number（数字）和 Date/Time（日期/时间）类型。**    
    Text类型   
     
    |数据类型|描述|
    |:----:|:----|
    |CHAR(size)|保存**固定长度**的字符串（字母、数字以及特殊字符）。size用于指定字符串的长度。最多255个字符|
    |VARCHAR(size)|保存**可变长度**的字符串（字母、数字以及特殊字符）。size用于指定最长长度。最多为255，如果超过了255就被转换为TEXT类型|
    |TINYTEXT |存放最长长度为255个字符的字符串|
    |TEXT|存放最长长度为65535个字符的字符串|   
    |BLOB|Binary Large Objects，最多存放65535字节的数据，典型的有图片和音乐|   
    |MEDIUMTEXT|存放最大长度为16777215个字符的字符串|
    |MEDIUMBLOB|存放最多16777215字节的数据|   
    |LONGTEXT|存放最大长度为4294967295个字符的字符串|  
    |LONGBLOB|存放4294967295字节的数据|   
    |ENUM(x,y,z,etc.)|枚举类型，最多列出65535个值。如果列表中不存在插入的值，则插入空值|   
    |SET(x,y,z,etc.)|与ENUM类似，insert的值可以是列表中的一个或多个值形成的一个集合，如果插入集合中有一个值是列表中不存在的则会报错|   
    **注：markdown制表的时候用的是竖线，不是斜杠。**    
    
    Number类型   
    
    |数据类型|描述|
    |:----:|:---|
    |INT(size)|带符号范围带符号范围-2147483648到2147483647，无符号的范围是0到4294967295。size 默认为 11。**size的作用是如果显示的位数不到11位，会用零补全，仅此而已**|
    |FLOAT(size, d)|带有浮动小数点的小数字。在 size 参数中规定显示最大位数。在 d 参数中规定小数点右侧的最大位数|
    例如，    
    ```text
       int的值为10 （指定zerofill）
       int（9）显示结果为000000010
       int（3）显示结果为010
    ```    
    
    Date类型    
    
    |数据类型|描述|
    |:----:|:---|
    |DATE()|日期。格式：YYYY-MM-DD, 支持的范围是从 '1000-01-01' 到 '9999-12-31'|
    |DATETIME()|*日期和时间的组合。格式：YYYY-MM-DD HH:MM:SS 支持的范围是从 '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'|
    |TIMESTAMP()|*时间戳。TIMESTAMP 值使用 Unix 纪元('1970-01-01 00:00:00' UTC) 至今的秒数来存储。格式：YYYY-MM-DD HH:MM:SS|
    |TIME()|时间。格式：HH:MM:SS, 支持的范围是从 '-838:59:59' 到 '838:59:59'|
    |YEAR()|2 位或 4 位格式的年。4 位格式所允许的值：1901 到 2155。2 位格式所允许的值：70 到 69，表示从 1970 到 2069。|
    
6. where子句中运算符in的用法  
    ```sql
       select * from table where id in (1,2,4);
    ```
    
7. order by多列时，是按照关键字后的列名顺序进行依次进行排序的。   

8. 使用IDE连接MySQL数据库时，出现错误
    ```text
       MySQL版本：8.0.16
       URL：jdbc:mysql://localhost:3306/febs
       错误代码：08001
    ```
    报错原因：IDEA默认使用的高版本数据库链接驱动，需要在URL后加上参数  
    ```text
       URL：jdbc:mysql://localhost:3306/febs?serverTimezone=Hongkong&characterEncoding=utf-8
    ```  
9. 
    