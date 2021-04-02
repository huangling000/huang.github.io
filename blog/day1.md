1 构建模块

引入了JPA,thymeleaf,mysl,devtools,web,aop等模块

2 application.yml配置

thymeleaf配置

数据库连接配置

日志配置

3 开发环境与生产环境不同配置

logback-spring.xml



### 异常处理

1 定义错误页面(resources/tmplates/error页面下创建三个html页面)

* 404
* 500
* error

2 主程序所在包下(com/zero)新建web包，定义一个控制器indexController,要写上Controller注解（理解Controller注解与GetMapping注解）

```java
@Controller
public class IndexController {
    @GetMapping("/")
    public String index(){
        //int i = 9/0;
        String blog = null;
        if(blog == null){
            throw new NotFoundException("博客不存在");
        }
        return "index";
    }
}

```

出现错误会spring会自己判断如何处理异常（自动拦截异常能够被认识的异常），跳转到对应404，500等页面

3 如何处理自定义异常，跳转到自定义异常界面。需要对异常进行拦截，自定义拦截器

在主程序所在包下新建(handler/ControllerExceptionHandler)，进行统一异常处理。需要使用注解@ControllerAdvice，@ExceptionHandler。返回对象ModeAndView。在异常处理时，需要判断有没有与ResponseStatus相匹配的异常，如果有，抛出对应的异常进行处理。

```java
/**
 * 拦截有Controller注解的控制器，如index
 */
@ControllerAdvice
public class ControllerExceptionHandler {

    //拿到日志对象
    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    /**
     * return：控制返回一个页面，里加一些后台输出到前台的信息
     * @ExceptionHandler:表示方法可以做异常处理，后面参数表示只要是Exception级别的都可以
     */
    @ExceptionHandler(Exception.class)
    public ModelAndView exceptionHandler(HttpServletRequest request,Exception e) throws Exception{
        logger.error("Request URL : {}, Exception : {}",request.getRequestURL(),e);
        //判断异常是否自定义被指定，如果被指定则不为空，抛出被指定异常
        if (AnnotationUtils.findAnnotation(e.getClass(),
                ResponseStatus.class) != null) {
            throw e;
        }
        ModelAndView mv = new ModelAndView();
        mv.addObject("url",request.getRequestURL());
        mv.addObject("Exception",e);
        mv.setViewName("error/error");
        return mv;
    }
}
```

4 处理业务场景时，有时需要处理异常，跳转到对应的错误页面，如访问一个不存在的blog，希望可以跳到404页面。

进行模拟，在出现这种场景时，抛出一个自定义异常进行处理：

```java
String blog = null;
        if(blog == null){
            throw new NotFoundException("博客不存在");
        }
```

定义NotFoundException,使用到@ResponseStatus注解。

```java
/**
 * 处理blog找不到异常
 * @ResponseStatus:跳转到404页面
 */
@ResponseStatus(HttpStatus.NOT_FOUND)
public class NotFoundException extends RuntimeException {
    public NotFoundException() {
    }

    public NotFoundException(String message) {
        super(message);
    }

    public NotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

