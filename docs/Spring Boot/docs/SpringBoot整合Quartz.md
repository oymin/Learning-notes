# SpringBoot 整合 Quartz 定时任务

## 1. quartz 的基本实现原理

Quartz 任务调度的核心API接口为：

- **Scheduler** 任务调度器，与调度程序交互的主要API

- **Trigger** 触发器，定义执行给定作业的计划的组件

- **TriggerBuilder** 用于定义、构建触发器实例

- **Job** 任务，调度程序执行的组件实现的接口

- **JobDetail** 用户定义作业的实例

- **JobBuilder** 用于定义、构建 JobDetail 实例

其中 trigger 和 job 是任务调度的元数据，scheduler 是实际执行调度的控制器。

### 1.1 Trigger 触发器

Trigger 是用于定义调度时间的元素，即按照什么时间规则去执行任务。Quartz 中主要提供了四种类型的 trigger：

- `SimpleTrigger`

- `CronTirgger`

- `DateIntervalTrigger`

- `NthIncludedDayTrigger`

这四种 trigger 可以满足企业应用中的绝大部分需求。

### 1.2 Job 任务

Job 用于表示被调度的任务。主要有两种类型的 job：

- stateless（无状态的）

- stateful（有状态的）

对于同一个 trigger 来说，有状态的 job 不能被并行执行，只有上一次触发的任务被执行完之后，才能触发下一次执行。Job 主要有两种属性：volatility 和 durability，其中 volatility 表示任务是否被持久化到数据库存储，而 durability 表示在没有 trigger 关联的时候任务是否被保留。两者都是在值为 true 的时候任务被持久化或保留。一个 job 可以被多个 trigger 关联，但是一个 trigger 只能关联一个 job。

### 1.3 Scheduler 任务调度器

Scheduler 由 scheduler 工厂创建：`DirectSchedulerFactory` 或者 `StdSchedulerFactory`。

第二种工厂 `StdSchedulerFactory` 使用较多，因为 `DirectSchedulerFactory` 使用起来不够方便，需要作许多详细的手工编码设置。

Scheduler 主要有三种：

- RemoteMBeanScheduler

- RemoteScheduler

- StdScheduler

![image](https://upload-images.jianshu.io/upload_images/10139744-5742e5071372cd71.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 1.4 简单示例代码

#### 1.4.1 创建任务类，实现 Job 接口

```java
public class MyJob implements Job {
    @Override
    public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        //任务要做的事情
        System.out.println("执行任务中......");
    }
}
```

#### 1.4.2 调用 job 执行任务

```java
/**
 * 流程：
 * 1. 创建 job 任务，设置任务的标识以及分组
 * 2. 创建 trigger 触发器，设置触发时间规则
 * 3. 创建 sechdule 调度器，将 job 与 trigger 绑定
 * 4. 注册监听器，sechdule 将 jobkey 与 job监听器 绑定
 * 5. 运行调度器
 */
public class MyTrigger {
    public static void main(String[] args) throws SchedulerException {

        //创建 job
        JobBuilder jobBuilder = JobBuilder.newJob(MyJob.class);
        /**
         * @param name 名称
         * @param group 分组
         */
        JobDetail job = jobBuilder.withIdentity("第一个任务", "1组").build();

        /**
         * 创建触发器：每5秒执行一次job，一直执行
         */
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("第一个触发器", "1组")
                .withSchedule(SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(5).repeatForever())
                //.withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ?")) //cron 规则方式
                .startNow()
                .build();

        /**
         * 创建调度器：将 job 与 trigger 绑定，运行调度器
         */
        Scheduler scheduler = new StdSchedulerFactory().getScheduler();
        scheduler.scheduleJob(job, trigger);

        //注册监听器
        MyJobListener myJobListener = new MyJobListener();
        KeyMatcher<JobKey> jobKeyKeyMatcher = KeyMatcher.keyEquals(job.getKey());
        //调度器：将 jobkey 绑定到 job监听器上
        scheduler.getListenerManager().addJobListener(myJobListener, jobKeyKeyMatcher);

        scheduler.start();

    }
}
```

#### 1.4.3 创建 JobListener 监听器

```java
/**
 * Job 监听器
 */
public class MyJobListener implements JobListener {

    @Override
    public String getName() {
        //返回监听器的名称
        return "myJobListener";
    }

    //job 执行前，执行此方法
    //context 可以获取 job 的对象
    @Override
    public void jobToBeExecuted(JobExecutionContext context) {
        System.out.println("Befor: 在job之前执行了....");
    }

    @Override
    public void jobExecutionVetoed(JobExecutionContext context) {
        System.out.println("否决执行时，执行....");
    }

    //job 执行后，执行此方法
    @Override
    public void jobWasExecuted(JobExecutionContext context, JobExecutionException e) {
        System.out.println("After: 在" + context.getJobDetail().getJobClass() + "之后前执行了....");
    }
}
```

---

## 2. Quartz 线程

在 Quartz 中，有两类线程，Scheduler 调度线程和任务执行线程，其中任务执行线程通常使用一个线程池维护一组线程。

### 2.1 Scheduler 调度线程主要有两个：

1、执行常规调度的线程

常规调度线程轮询存储的所有 trigger，如果有需要触发的 trigger，即到达了下一次触发的时间，则从任务执行线程池获取一个空闲线程，执行与该 trigger 关联的任务。

2、执行 misfired trigger 的线程

Misfire 线程是扫描所有的 trigger，查看是否有 misfiredtrigger，如果有的话根据 misfire 的策略分别处理(fire now OR wait for the next fire)

![Scheduler](https://upload-images.jianshu.io/upload_images/10139744-6e33d96c22a114bd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.2 Quartz Job 数据存储

Quartz 中的 trigger 和 job 需要存储下来才能被使用。

Quartz 中有两种存储方式：

- RAMJobStore ：将 trigger 和 job 存储在内存中

- JobStoreSupport ：基于 jdbc 将 trigger 和 job 存储到数据库中

RAMJobStore 的存取速度非常快，但是由于其在系统被停止后所有的数据都会丢失，所以在集群应用中，必须使用JobStoreSupport。

### 2.3 Quartz 集群架构

一个 Quartz 集群中的每个节点是一个独立的 Quartz 应用，它又管理着其他的节点。这就意味着你必须对每个节点分别启动或停止。Quartz 集群中，独立的 Quartz 节点并不与另一其的节点或是管理节点通信，而是通过相同的数据库表来感知到另一 Quartz 应用的，如图2.1所示。

![ Quartz 集群](https://upload-images.jianshu.io/upload_images/10139744-a9e37445079d8e8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2.4 Quartz 集群相关数据库表

因为 Quartz 集群依赖于数据库，所以必须首先创建 Quartz 数据库表，Quartz 发布包中包括了所有被支持的数据库平台的 SQL 脚本。这些 SQL 脚本存放于 `<quartz_home>/docs/dbTables` 目录下。

![Quartz数据库表](https://upload-images.jianshu.io/upload_images/10139744-de3a957cf6b92e11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**QRTZ_JOB_DETAILS：** 存储的是 job 的详细信息，包括：[`DESCRIPTION`]描述，[`IS_DURABLE`]是否持久化，[`JOB_DATA`]持久化对象等基本信息。

**QRTZ_TRIGGERS：** 触发器信息，包含：job 的名，组外键，[`DESCRIPTION`]触发器的描述等基本信息，还有[`START_TIME`]开始执行时间，[`END_TIME`]结束执行时间，[`PREV_FIRE_TIME`]上次执行时间，[`NEXT_FIRE_TIME`]下次执行时间，[`TRIGGER_TYPE`]触发器类型：simple 和 cron，[`TRIGGER_STATE`]执行状态：WAITING，PAUSED，ACQUIRED 分别为：等待、暂停、运行中。

**QRTZ_CRON_TRIGGERS：** 保存 cron 表达式。

**QRTZ_SCHEDULER_STATE：** 存储集群中 note 实例信息，quartz 会定时读取该表的信息判断集群中每个实例的当前状态，INSTANCE_NAME:之前配置文件中 `org.quartz.scheduler.instanceId` 配置的名字，就会写入该字段，如果设置为 AUTO，quartz 会根据物理机名和当前时间产生一个名字。  [`LAST_CHECKIN_TIME`]上次检查时间，[`CHECKIN_INTERVAL`]检查间隔时间。

**QRTZ_PAUSED_TRIGGER_GRPS：** 暂停的任务组信息。

**QRTZ_LOCKS：** 悲观锁发生的记录信息。

**QRTZ_FIRED_TRIGGERS：** 正在运行的触发器信息。

**QRTZ_SIMPLE_TRIGGERS：** 简单的触发器详细信息。

**QRTZ_BLOB_TRIGGERS：** 触发器存为二进制大对象类型(用于 Quartz 用户自己触发数据库定制自己的触发器，然而 JobStore 不明白怎么存放实例的时候)。

**QRTZ_CALENDARS：** 以 Blob 类型存储 Quartz 的 Calendar 信息

数据库相关表介绍：[http://www.cnblogs.com/zhenyuyaodidiao/p/4755649.html](http://www.cnblogs.com/zhenyuyaodidiao/p/4755649.html)

参数配置介绍：[http://blog.csdn.net/zixiao217/article/details/53091812](http://blog.csdn.net/zixiao217/article/details/53091812)

---

## 3. SpringBoot 整合 Quartz

### 创建 quartz 的相关的数据库中的表

1、在官网下载 quartz

[quartz-2.3.0-distribution.tar.gz](http://www.quartz-scheduler.org/downloads/files/quartz-2.3.0-distribution.tar.gz)

![quartz下载](https://upload-images.jianshu.io/upload_images/10139744-63655165587c8c44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2、解压进入到目录 `docs/daTables` 目录里面有数据库表 sql 脚本选择自己对应的数据库脚本，直接执行就可以生成 quartz 相关的表

![image](https://upload-images.jianshu.io/upload_images/10139744-c1d4de6cbfa95b34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image](https://upload-images.jianshu.io/upload_images/10139744-412d363c392e164d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### pom.xml引入依赖

```xml
<!--集成quartz-->
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>${quartz.version}</version>
    <!-- quartz默认使用c3p0连接池，如果项目使用的不是则需要排除依赖包 -->
    <exclusions>
        <exclusion>
            <artifactId>c3p0</artifactId>
            <groupId>c3p0</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
</dependency>
```

### 创建配置文件类

```java
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.quartz.SchedulerFactoryBean;

import javax.sql.DataSource;
import java.util.Properties;

/**
 * 定时任务配置
 *
 */
@Configuration
public class QuartzConfig {

    @Bean
    public SchedulerFactoryBean scheduler(@Qualifier("druidDataSource") DataSource dataSource) {

        //quartz参数
        Properties prop = new Properties();
        //配置实例
        //prop.put("org.quartz.scheduler.instanceName", "MyScheduler");//实例名称
        prop.put("org.quartz.scheduler.instanceId", "AUTO");
        //线程池配置
        prop.put("org.quartz.threadPool.threadCount", "5");
        //JobStore配置
        prop.put("org.quartz.jobStore.class", "org.quartz.impl.jdbcjobstore.JobStoreTX");
        prop.put("org.quartz.jobStore.tablePrefix", "QRTZ_");

        SchedulerFactoryBean factory = new SchedulerFactoryBean();
        factory.setDataSource(dataSource);
        factory.setQuartzProperties(prop);
        factory.setSchedulerName("MyScheduler");//数据库中存储的名字
        //QuartzScheduler 延时启动，应用启动5秒后 QuartzScheduler 再启动
        factory.setStartupDelay(5);

        //factory.setApplicationContextSchedulerContextKey("applicationContextKey");
        //可选，QuartzScheduler 启动时更新己存在的Job，这样就不用每次修改targetObject后删除qrtz_job_details表对应记录了
        factory.setOverwriteExistingJobs(true);
        //设置自动启动，默认为true
        factory.setAutoStartup(true);

        return factory;
    }
}
```

### 创建自定义 Job 任务类，继承 QuartzJobBean

```java
import com.alibaba.fastjson.JSON;
import com.qfedu.rongzaiboot.entity.ScheduleJob;
import com.qfedu.rongzaiboot.entity.ScheduleJobLog;
import com.qfedu.rongzaiboot.service.ScheduleJobLogService;
import com.qfedu.rongzaiboot.utils.SpringContextUtils;
import org.apache.commons.lang.StringUtils;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.quartz.QuartzJobBean;
import org.springframework.util.ReflectionUtils;

import java.lang.reflect.Method;
import java.util.Date;

public class QuartzJob extends QuartzJobBean {
    private Logger logger = LoggerFactory.getLogger(getClass());

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        System.out.println("执行quartz任务。。。。。");

        String json = context.getMergedJobDataMap().getString("JOB_PARAM_KEY");
        //将获取的对象序列化的json 转化为实体类对象
        ScheduleJob scheduleJob = JSON.parseObject(json, ScheduleJob.class);

        Long jobId = scheduleJob.getJobId();
        String beanName = scheduleJob.getBeanName();
        String methodName = scheduleJob.getMethodName();
        String params = scheduleJob.getParams();

        //quartz没有被spring管理 所以通过其它方式获取service
        ScheduleJobLogService scheduleJobLogService = (ScheduleJobLogService) SpringContextUtils.getBean("scheduleJobLogServiceImpl");
        //保存任务记录日志
        ScheduleJobLog scheduleJobLog = new ScheduleJobLog();
        scheduleJobLog.setJobId(jobId);
        scheduleJobLog.setBeanName(beanName);
        scheduleJobLog.setMethodName(methodName);
        scheduleJobLog.setParams(params);
        scheduleJobLog.setCreateTime(new Date());

        long startTime = System.currentTimeMillis();

        try {
            Object targetClass = SpringContextUtils.getBean(beanName);
            Method method = null;
            //通过反射获取方法
            if (StringUtils.isNotBlank(params)) {
                method = targetClass.getClass().getDeclaredMethod(methodName, String.class);
            } else {
                method = targetClass.getClass().getDeclaredMethod(methodName);
            }

            ReflectionUtils.makeAccessible(method);//使方法具有public权限
            //根据反射执行方法
            if (StringUtils.isNotBlank(params)) {
                method.invoke(targetClass, params);
            } else {
                method.invoke(targetClass);
            }

            long endTime = System.currentTimeMillis() - startTime;

            scheduleJobLog.setStatus((byte) 0);//保存日志里的操作状态 0:成功
            scheduleJobLog.setTimes(endTime);//耗时多长时间

            logger.info("任务执行成功，任务ID：" + jobId + "，总耗时：" + endTime + "毫秒");

        } catch (Exception e) {
            long endTime = System.currentTimeMillis() - startTime;
            scheduleJobLog.setError(StringUtils.substring(e.toString(),2000));//错误消息
            scheduleJobLog.setStatus((byte)1);//失败
            scheduleJobLog.setTimes(endTime);//耗时

            e.printStackTrace();
            logger.error("任务执行失败，任务ID："+jobId);
        } finally {
            //最后调用service保存定时任务日志记录
            scheduleJobLogService.save(scheduleJobLog);
        }

    }

}
```

### 创建Scheduler工具类，quartz的操作核心，包括操作quartz在数据库中的表

```java
import com.alibaba.fastjson.JSON;
import com.qfedu.rongzaiboot.entity.ScheduleJob;
import com.qfedu.rongzaiboot.quartz.QuartzJob;
import org.quartz.*;

public class SchedulerUtils {

    /**
     * 创建任务
     */
    public static void createJob(Scheduler scheduler, ScheduleJob scheduleJob) {

        try {
            Long jobId = scheduleJob.getJobId();
            //创建Job对象
            JobDetail job = JobBuilder.newJob(QuartzJob.class).withIdentity("JOB_" + jobId).build();
            //获取cron表达式 并创建对象
            CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(scheduleJob.getCronExpression())
                    .withMisfireHandlingInstructionDoNothing();
            //创建触发器
            CronTrigger trigger = TriggerBuilder.newTrigger()
                    .withIdentity("TRIGGET_" + jobId)
                    .withSchedule(cronScheduleBuilder) //将cron表达式配置到触发器
                    .build();

            //将对象josn序列化存储到Job的getJobDataMap()方法中，为后续根据获取属性执行对应的类的任务
            job.getJobDataMap().put("JOB_PARAM_KEY", JSON.toJSONString(scheduleJob));
            //存数据
            scheduler.scheduleJob(job, trigger);
            scheduler.pauseJob(JobKey.jobKey("JOB_" + jobId));//使任务处于等待状态,创建后不会执行
        } catch (SchedulerException e) {
            throw new RRException("创建任务失败", e);
        }
    }

    /**
     * 更新任务
     */
    public static void updateJob(Scheduler scheduler, ScheduleJob scheduleJob) {
        //获取新的cron表达式
        CronScheduleBuilder cronScheduleBuilder = CronScheduleBuilder.cronSchedule(scheduleJob.getCronExpression())
                .withMisfireHandlingInstructionDoNothing();

        Long jobId = scheduleJob.getJobId();

        try {
            //拿到原有的trigger
            TriggerKey triggerKey = TriggerKey.triggerKey("TRIGGER_" + jobId);
            CronTrigger trigger = (CronTrigger) scheduler.getTrigger(triggerKey);
            //为原有的trigger赋予新的cron表达式
            trigger = trigger.getTriggerBuilder().withIdentity(triggerKey)
                    .withSchedule(cronScheduleBuilder).build();
            //执行原有的trigger更新
            scheduler.rescheduleJob(triggerKey, trigger);
        } catch (SchedulerException e) {
            e.printStackTrace();
            throw new RRException("更新定时任务失败", e);
        }
    }

    /**
     * 删除任务
     */
    public static void deleteJob(Scheduler scheduler, Long jobId) {
        try {
            scheduler.deleteJob(JobKey.jobKey("JOB_" + jobId));
        } catch (SchedulerException e) {
            e.printStackTrace();
            throw new RRException("删除定时任务失败", e);
        }
    }

    /**
     * 恢复任务
     */
    public static void resumeJob(Scheduler scheduler, Long jobId) {
        try {
            scheduler.resumeJob(JobKey.jobKey("JOB_" + jobId));
        } catch (SchedulerException e) {
            e.printStackTrace();
            throw new RRException("恢复定时任务失败", e);
        }
    }

    /**
     * 立即执行定时任务
     */
    public static void run(Scheduler scheduler, Long jobId) {
        try {
            //只执行一次并且不会改变任务的状态
            scheduler.triggerJob(JobKey.jobKey("JOB_" + jobId));
        } catch (SchedulerException e) {
            e.printStackTrace();
            throw new RRException("立即执行定时任务失败", e);
        }
    }

    /**
     * 暂停任务
     *
     * @param scheduler
     * @param jobId
     */
    public static void pauseJob(Scheduler scheduler, Long jobId) {
        try {
            scheduler.pauseJob(JobKey.jobKey("JOB_" + jobId));
        } catch (SchedulerException e) {
            e.printStackTrace();
            throw new RRException("暂停定时任务失败", e);
        }
    }
}
```

### 保存任务的自定义类实体类

```java
public class ScheduleJob implements Serializable {
    private static final long serialVersionUID = 1L;

    private Long jobId;

    private String beanName; //执行的类名

    private String methodName; //方法名

    private String params; //参数

    private String cronExpression; //cron表达式

    private Byte status; //任务状态 0，运行 1，暂停

    private String remark; //备注

    private Date createTime; //创建时间
}
```

![image](https://upload-images.jianshu.io/upload_images/10139744-04bc9b0f312c878d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 创建quartz任务

`controller`

```java
/**
* 保存定时任务
*/
@MyLog("保存定时任务")
@RequestMapping("/save")
@RequiresPermissions("schedule:job:save")
public R save(@RequestBody ScheduleJob scheduleJob){
    scheduleJobService.save(scheduleJob);
    return R.ok();
}
```

`service,impl`

```java

/**
* 接口：保存定时任务
*/
void save(ScheduleJob scheduleJob);

//------------------

@Override
@Transactional(propagation = Propagation.REQUIRED)
public void save(ScheduleJob scheduleJob) {
    //保存实体类的信息
    scheduleJob.setCreateTime(new Date());
    scheduleJob.setStatus(Constant.ScheduleStatus.PAUSE.getValue());
    scheduleJobMapper.insertSelective(scheduleJob);

    //创建定时任务 并保存到对应的quatrz表中
    SchedulerUtils.createJob(scheduler, scheduleJob);
}
```

### 修改任务

`controller`

```java
/**
* 定时任务信息
*/
@GetMapping("/info/{jobId}")
@RequiresPermissions(value={"schedule:job:info"})
public R info(@PathVariable("jobId") Long jobId){

    ScheduleJob scheduleJob = scheduleJobService.queryObject(jobId);
    return R.ok().put("scheduleJob", scheduleJob);

}

//-------------------------------------------

@MyLog("修改定时任务")
@PostMapping("/update")
@RequiresPermissions(value={"schedule:job:update"})
public R update(@RequestBody ScheduleJob scheduleJob){

    scheduleJobService.update(scheduleJob);
    return R.ok();
}
```

`service,impl`

```java
/**
 * 查询
 */
ScheduleJob queryObject(Long jobId);

/**
 * 更新定时任务
 */
void update(ScheduleJob scheduleJob);

//--------------------------------------------------------

@Override
public ScheduleJob queryObject(Long menuId) {
    return scheduleJobMapper.selectByPrimaryKey(menuId);
}

@Override
@Transactional(propagation = Propagation.REQUIRED)
public void update(ScheduleJob scheduleJob) {

    SchedulerUtils.updateJob(scheduler, scheduleJob);

    scheduleJobMapper.updateByPrimaryKeySelective(scheduleJob);
}
```

`dao,mapper`

```xml
<select id="selectByPrimaryKey" resultMap="BaseResultMap" parameterType="java.lang.Long" >
select
    <include refid="Base_Column_List" />
from schedule_job
where job_id = #{jobId,jdbcType=BIGINT}
</select>

<update id="updateByPrimaryKeySelective" parameterType="com.qfedu.rongzaiboot.entity.ScheduleJob" >
update schedule_job
<set >
    <if test="beanName != null" >
    bean_name = #{beanName,jdbcType=VARCHAR},
    </if>
    <if test="methodName != null" >
    method_name = #{methodName,jdbcType=VARCHAR},
    </if>
    <if test="params != null" >
    params = #{params,jdbcType=VARCHAR},
    </if>
    <if test="cronExpression != null" >
    cron_expression = #{cronExpression,jdbcType=VARCHAR},
    </if>
    <if test="status != null" >
    status = #{status,jdbcType=TINYINT},
    </if>
    <if test="remark != null" >
    remark = #{remark,jdbcType=VARCHAR},
    </if>
    <if test="createTime != null" >
    create_time = #{createTime,jdbcType=TIMESTAMP},
    </if>
</set>
where job_id = #{jobId,jdbcType=BIGINT}
</update>
```

### 删除任务

`controller`

```java
/**
 * 删除定时任务
 */
@MyLog("删除定时任务")
@PostMapping("/del")
@RequiresPermissions("schedule:job:delete")
public R delete(@RequestBody Long[] jobIds){

    scheduleJobService.deleteBatch(jobIds);
    return R.ok();
}
```

`service,impl`

```java
/**
 * 批量删除
 */
@Override
@Transactional(propagation = Propagation.REQUIRED)
public void deleteBatch(Long[] jobIds) {

    for(Long jobId : jobIds){
        SchedulerUtils.deleteJob(scheduler, jobId);
    }

    //删除数据
    scheduleJobMapper.deleteBatch(jobIds);
}
```

`dao,mapper`

```xml
<!-- 批量删除任务记录 -->
<delete id="deleteBatch">
    delete from schedule_job where job_id in
    <foreach collection="array" item="jobId" open="(" separator="," close=")">
        #{jobId}
    </foreach>
</delete>
```

### 暂停定时任务

`controller`

```java
/**
 * 暂停定时任务
 */
@MyLog("暂停定时任务")
@PostMapping("/pause")
@RequiresPermissions("schedule:job:pause")
public R pause(@RequestBody Long[] jobIds){

    scheduleJobService.pause(jobIds);
    return R.ok();
}
```

`service,impl`

```java
/**
 * 批量暂停任务
 */
@Override
@Transactional(propagation = Propagation.REQUIRED)
public void pause(Long[] jobIds) {

    for(Long jobId : jobIds){
        SchedulerUtils.pauseJob(scheduler, jobId);
    }

    Map<String, Object> map = new HashMap<>();
    map.put("list", jobIds);
    map.put("status", Constant.ScheduleStatus.PAUSE.getValue());
    scheduleJobMapper.updateBatch(map);
}
```

`dao,mapper`

```xml
<!-- 批量更新状态 -->
<update id="updateBatch">
    update schedule_job set status = #{status} where job_id in
    <foreach collection="list" item="jobId" open="(" separator="," close=")">
        #{jobId}
    </foreach>
</update>
```

### 恢复任务

`controller`

```java
/**
 * 恢复定时任务
 */
@MyLog("恢复定时任务")
@PostMapping("/resume")
@RequiresPermissions("schedule:job:resume")
public R resume(@RequestBody Long[] jobIds){

    scheduleJobService.resume(jobIds);
    return R.ok();
}
```

`service,impl`

```java
/**
 * 恢复定时任务
 */
@Override
@Transactional(propagation = Propagation.REQUIRED)
public void resume(Long[] jobIds) {

    for(Long jobId : jobIds){
        SchedulerUtils.resumeJob(scheduler, jobId);
    }

    Map<String, Object> map = new HashMap<>();
    map.put("list", jobIds);
    map.put("status", Constant.ScheduleStatus.NORMAL.getValue());
    scheduleJobMapper.updateBatch(map);
}
```

`dao,mapper`

```xml
<!-- 批量更新状态 -->
<update id="updateBatch">
    update schedule_job set status = #{status} where job_id in
    <foreach collection="list" item="jobId" open="(" separator="," close=")">
        #{jobId}
    </foreach>
</update>
```

### 立即执行任务（项目里被设置只会执行一次）

`controller`

```java
/**
 * 立即执行定时任务
 */
@MyLog("立即执行定时任务")
@PostMapping("/run")
@RequiresPermissions("schedule:job:run")
public R run(@RequestBody Long[] jobIds){

    scheduleJobService.run(jobIds);
    return R.ok();
}
```

`service,impl`

```java
/**
 * 立即执行定时任务
 */
@Override
@Transactional(propagation = Propagation.REQUIRED)
public void run(Long[] jobIds) {
    for(Long jobId : jobIds){
        SchedulerUtils.run(scheduler, jobId);
    }
}
```
