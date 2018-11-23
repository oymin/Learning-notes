# Jfinal 3.0

## 项目构建

### 使用 Eclipse+Mave 构建 Jfinal 的 HelloWrold 的项目步骤

- 配置 MAVEN
- 安装 eclipse Maven 插件
- 设置 eclipse Maven 插件的相关配置
- pom.xml 中填写 Jfinal 的相关依赖描述，并且等待依赖包下载完成
- 创建 MyConfig 配置类
- web.xml 中进行 Jfinal 的配置
- 建立 Controller ，并且继承 Jfinal 的 Controller
- 建立 Route 映射
- 编写 Jfinal 的 Jetty 启动方法 ，并且启动项目
- 浏览器中输入访问地址，看结果

## Interceptor 的分类

```
jfinal 中的 configEngine(Engine me) 中配置的 Engine 对象是用于 Controller.render(...) 方法的渲染，而 ActiveRecordPlugin.getEngine() 对象是用于 sql 管理功能模块，这两个 Engine 对象是两个不同的实例，互相之间没有干扰，配置方式也不同
```

---

- Method Interceptor 方法级别
- Class Interceptor 类级别 对当前勒种所有的方法都有效
- Router Interceptor 路由级别 Interceptor 对当前路由中所有的方法都有效
- Global Interceptor 全局拦截 Interceptor 对 web 请求中所有的请求方法进行拦截
- Inject Interceptor 业务注入拦截器 对指定需要被注入的方法有效，可以整个业务类，也可是某个方法

拦截器栈，多个 Interceptor 进行组合形成的拦截器栈

---

## Jfinal Template Engine (模板引擎)



### 两个核心理念

- 在模板中可以直接与 java 代码通畅地交互
- 尽可能沿用 java 语法规则，将学习成本降到极致

### 表达式

```
- 算术运算： +   -   *   /   %   ++   --
- 比较运算： >  >=   <   <=  ==   !=  (基本用法相同，后面会介绍增强部分)
- 逻辑运算： !   &&   ||
- 三元表达式： ? :
- Null 值常量: null
- 字符串常量： "jfinal club"
- 布尔常量：true false
- 数字常量： 123  456F  789L  0.1D  0.2E10
- 数组存取：array[i](Map被增强为额外支持 map[key]的方式取值)
- 属性取值：object.field(Map被增强为额外支持map.key 的方式取值)
- 方法调用：object.method(p1, p2…, pn) (支持可变参数)
- 逗号表达式：123, 1>2, null, "abc", 3+6 (逗号表达式的值为最后一个表达式的值)
```

### 六个指令

- #if
- #for
- #define
- #set
- #include
- #(..)

---

## JFinal + Redis 统计接口访问次数

```
RequestStatisticsInterceptor Interceptor{
    intercept(Invocation inv) {
        String className = inv.getController().getClass().getName();
        String methodName = inv.getMethodName();
        Jedis jedis = Redis.().getJedis();
        (jedis != ){
            jedis.hincrBy(,className++methodName,);
            jedis.close();
        }
        inv.invoke();
    }
}
```

## 开放接口IP访问控制拦截器

```
import java.util.Objects;
import com.jfinal.aop.Interceptor;
import com.jfinal.aop.Invocation;
import com.jfinal.kit.PropKit;
import com.jfinal.plugin.redis.Cache;
import com.jfinal.plugin.redis.Redis;
 
 
/**
* @author 作者：qiww
* @createDate 创建时间：2018年6月11日 下午5:01:36
* 
* 对于开放接口进行IP访问控制<br>
* 
* 开发接口是指无需token验证的接口。
*/
public class IPControlInterceptor implements Interceptor {
	static Cache cache = Redis.use("...");
 
	@Override
	public void intercept(Invocation inv) {
		if (isLimitedIP(inv)) {
			((BaseController) inv.getController()).renderAppError("IP已被锁定，请稍后重试");
			return;
		}
		
		inv.invoke();
	}
	
	/**
	 * @param inv
	 * @return false-无限制IP，true-已限制IP
	 */
	public static boolean isLimitedIP(Invocation inv) {
		try {
			//测试环境无限制
			boolean testmode = PropKit.getBoolean("testMode", true);
			if (testmode) {//测试环境无IP限制
				return false;
			}
			
			//获取IP地址
			String IP = StringUtil.getIP(inv.getController().getRequest());
			
			//本地测试
			if (Objects.equals("127.0.0.1", IP)) {
				return false;
			}
			
			//判断IP是否已经被锁定
			if (cache.exists("openapi:lock:" + IP)) {
				//已经锁定
				return true;
			} 
			
			//若没有锁定,记录访问次数
			String keyCounts = "openapi:calltimes:" + IP;
			if (cache.get(keyCounts) != null) {
				Long times = cache.get(keyCounts);
				
				//是否达到最大访问次数：1*60秒内同一最大访问次数为100，锁定时间为1*60秒
				if (times+1 >= 100) {
					cache.del(keyCounts);
					cache.set("openapi:lock:" + IP, times);
					cache.expire("openapi:lock:" + IP, 1*60);
					return true;
				} else {
					cache.set(keyCounts, times+1);
					cache.expire(keyCounts, 1*60);
					return false;
				}
			}
			
			cache.set(keyCounts, 1L);
			
			return false;
		} catch (Exception e) {
			System.out.println(e.getMessage());
			return false;
		}
		
	}
 
}
```

应用实例

```
@Clear(TokenVerificationInterceptor.class) //公开接口
@Before(IPControlInterceptor.class) //IP访问限制
public void register() {//注册
	//TODO
	...
}
```
















