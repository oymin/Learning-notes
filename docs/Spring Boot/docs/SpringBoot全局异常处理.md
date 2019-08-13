# SpringBoot全局异常处理

## 方式一：继承BasicErrorController

1、自定义错误处理 controller 继承 BasicErrorController

```java
import org.springframework.boot.autoconfigure.web.ErrorProperties;
import org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController;
import org.springframework.boot.autoconfigure.web.servlet.error.ErrorViewResolver;
import org.springframework.boot.web.servlet.error.ErrorAttributes;

/**
 * 自定义错误处理controller
 */
public class MyErrorController extends BasicErrorController {

    //继承后创建的构造方法
    public MyErrorController(ErrorAttributes errorAttributes, ErrorProperties errorProperties, List<ErrorViewResolver> errorViewResolvers) {
        super(errorAttributes, errorProperties, errorViewResolvers);
    }

    /*
    springboot默认返回的错处信息格式
    {
    "timestamp": "2018-08-23 20:33:12",
    "status": 500,
    "error": "Internal Server Error",
    "message": "编号不可为空",
    "path": "/manager/products",
    }
    */

    /**
     * 重写 getErrorAttributes()方法:
     * 可以获取返回的错误属性内容
     * 也可以设置返回哪些内容
     */
    @Override
    protected Map<String, Object> getErrorAttributes(HttpServletRequest request, boolean includeStackTrace) {
        Map<String, Object> attrMap = super.getErrorAttributes(request, includeStackTrace);
        //去除掉默认返回的错误信息
        attrMap.remove("timestamp");
        attrMap.remove("status");
        attrMap.remove("error");
        attrMap.remove("exception");
        attrMap.remove("path");
        //返回自定义的错误信息
        String errorCode = (String) attrMap.get("message");
        ErrorEnum errorEnum = ErrorEnum.getByCode(errorCode);
        attrMap.put("message", errorEnum.getMessage());
        attrMap.put("code", errorEnum.getCode());
        attrMap.put("success", errorEnum.isSuccess());
        return attrMap;
    }
}
```

2、将自定义的错误处理类注入 spring 容器中

```java
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.boot.autoconfigure.web.ServerProperties;
import org.springframework.boot.autoconfigure.web.servlet.error.ErrorViewResolver;
import org.springframework.boot.web.servlet.error.ErrorAttributes;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.List;

/**
 * 错误处理相关配置
 */
@Configuration
public class ErrorConfiguration {

    //注入：自定义错误处理 MyErrorController
    @Bean
    public MyErrorController basicErrorController(ErrorAttributes errorAttributes,
                                                  ServerProperties serverProperties,
                                                  ObjectProvider<List<ErrorViewResolver>> errorViewResolversProvider) {
        return new MyErrorController(errorAttributes, serverProperties.getError(), errorViewResolversProvider.getIfAvailable());
    }

}
```

3、错误枚举

```java
/**
 * 错误种类
 */
public enum ErrorEnum {

    ID_NOT_NULL("F001", "编号不可为空", false),
    UNKNOWN("999", "未知异常", false);

    private String code;
    private String message;
    private boolean success;

    ErrorEnum(String code, String message, boolean success) {
        this.code = code;
        this.message = message;
        this.success = success;
    }

    public static ErrorEnum getByCode(String code) {
        for (ErrorEnum errorEnum : ErrorEnum.values()) {
            if (errorEnum.code.equals(code)) {
                return errorEnum;
            }
        }
        return UNKNOWN;
    }

    public String getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }

    public boolean isSuccess() {
        return success;
    }
}
```

---
