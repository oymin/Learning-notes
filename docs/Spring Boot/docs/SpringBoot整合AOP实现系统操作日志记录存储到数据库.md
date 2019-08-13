# SpringBoot整合AOP实现系统操作日志记录存储到数据库

采用方案： 使用spring 的 aop 技术切到自定义注解上，针对不同注解标志进行参数解析，记录日志  
缺点是要针对每个不同的注解标志进行分别取注解标志，获取参数进行日志记录输出  

## 1. 需要引用的依赖

```xml
<!--spring切面aop依赖-->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

在application.properties文件里加这样一条配置
spring.aop.auto=true //这个配置我的例子中没有加 也正常运行
```

## 2. 创建实体类

```java
public class SysLog implements Serializable {
    private Long id;

    private String username; //用户名

    private String operation; //操作

    private String method; //方法名

    private String params; //参数

    private String ip; //ip地址

    private Date createDate; //操作时间
    //创建getter和setter方法
}
```  

## 3. 使用spring 的 aop 技术切到自定义注解上，所以先创建一个自定义注解类

```java
import java.lang.annotation.*;

/**
 * 自定义注解类
 */
@Target(ElementType.METHOD) //注解放置的目标位置,METHOD是可注解在方法级别上
@Retention(RetentionPolicy.RUNTIME) //注解在哪个阶段执行
@Documented //生成文档
public @interface MyLog {
    String value() default "";
}
```  
  
## 4. 创建aop切面实现类

```java
import com.alibaba.fastjson.JSON;
import com.qfedu.rongzaiboot.annotation.MyLog;
import com.qfedu.rongzaiboot.entity.SysLog;
import com.qfedu.rongzaiboot.service.SysLogService;
import com.qfedu.rongzaiboot.utils.HttpContextUtils;
import com.qfedu.rongzaiboot.utils.IPUtils;
import com.qfedu.rongzaiboot.utils.ShiroUtils;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.AfterReturning;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;
import java.util.Date;

/**
 * 系统日志：切面处理类
 */
@Aspect
@Component
public class SysLogAspect {

    private SysLog sysLog = new SysLog();

    @Autowired
    private SysLogReposiroty sysLogReposiroty;

    //定义切点 @Pointcut
    //在注解的位置切入代码
    @Pointcut("@annotation(com.koax.entity.system.MyLog)")
    public void logPoinCut() {
    }

    /**
     * 前置通知：获取方法以及请求参数等
     */
    @Before("logPoinCut()")
    public void doBefor(JoinPoint joinPoint) {
        //从切面织入点处通过反射机制获取织入点处的方法
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        //获取切入点所在的方法
        Method method = signature.getMethod();
        //获取操作
        MyLog myLog = method.getAnnotation(MyLog.class);
        if (myLog != null) {
            String value = myLog.value();
            sysLog.setOperation(value);//保存获取的操作
        }

        //获取请求的类名
        String className = joinPoint.getTarget().getClass().getName();
        //获取请求的方法名
        String methodName = method.getName();
        sysLog.setMethod(className + "." + methodName);

        //请求的参数
        Object[] args = joinPoint.getArgs();
        //将参数所在的数组转换成json
        String params = JSON.toJSONString(args);
        sysLog.setParams(params);
        //操作的时间
        sysLog.setCreateDate(new Date());
        //获取用户名
        sysLog.setUsername("admin");
        //获取用户ip地址
        HttpServletRequest request = HttpContextUtils.getHttpServletRequest();
        sysLog.setIp(IPUtils.getIpAddr(request));

    }

    /**
     * 返回通知：获取方法返回值，存入日志
     *
     * @param joinPoint
     * @param rvt       方法返回值
     */
    @AfterReturning(value = "@annotation(com.koax.entity.system.MyLog)", returning = "rvt")
    public void doAfterReturning(JoinPoint joinPoint, Object rvt) {
        sysLog.setReturnData(JSON.toJSONString(rvt));
        sysLog = sysLogReposiroty.save(sysLog);
    }

    /**
     * 异常通知：获取异常信息，存入日志
     *
     * @param joinPoint
     * @param ex
     */
    @AfterThrowing(pointcut = "logPoinCut()", throwing = "ex")
    public void doAfterThrowing(JoinPoint joinPoint, Exception ex) {
        sysLog.setExceptions(ex.toString());
        sysLogReposiroty.save(sysLog);
    }

}
```

## 5. 接下来就可以在需要监控的方法上添加 aop的自定义注解 

格式为 @+自定义注解的类名 ，比如：`@MyLog`

```java
//例如在contoller类的方法上加注解
@RestController
@RequestMapping("/sys/menu")
public class SysMenuController extends AbstractController {

    @Autowired
    private SysMenuService sysMenuService;

    @MyLog(value = "删除菜单记录")  //这里添加了AOP的自定义注解
    @PostMapping("/del")
    public R deleteBatch(@RequestBody Long[] menuIds) {
        for (Long menuId : menuIds) {
            if (menuId <= 31) {
                return R.error("系统菜单，不能删除");
            }
        }
        sysMenuService.deleteBatch(menuIds);

        return R.ok("删除成功");
    }
}
```

## 6. 执行上面的方法操作后，会将记录保存到数据库

![image](https://upload-images.jianshu.io/upload_images/10139744-a0bd94aff9920928.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
