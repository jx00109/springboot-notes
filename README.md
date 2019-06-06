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