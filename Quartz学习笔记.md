---
title: Quartz学习笔记
date: 2020-05-20 22:45:40
tags:
- Quartz
- Java
categories:
- Java
- 任务调度
---

[官网](http://www.quartz-scheduler.org/)

## 1.基本使用

### 1.1 导入依赖
```xml
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.3.0</version>
</dependency>
```

<!-- more -->

### 1.2 创建自定义 Job 类实现任务具体执行逻辑。

`MyJob.java`
```java
public class MyJob implements Job {
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        JobDetail detail = context.getJobDetail();
        String name = detail.getKey().getName();
        String group = detail.getKey().getGroup();
        String job = detail.getJobDataMap().getString("data");
        System.out.println("Job执行，Job名:" + name + " | Group: " + group + " | data:" + job + " | " + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
    }
}
```

### 1.3 创建任务调度器、任务触发器、任务细节

#### SimpleSchedule

```java
// 1.创建 Scheduler 任务调度器
Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
// 2.创建任务触发器
TriggerBuilder<Trigger> triggerBuilder = TriggerBuilder.newTrigger();
Trigger trigger = triggerBuilder
    .withIdentity("trigger1", "group1")
    .startNow()
    .withSchedule(SimpleScheduleBuilder
                  .simpleSchedule()
                  .withIntervalInSeconds(2)
                  .repeatForever())
    .endAt(new GregorianCalendar(2020, 4, 21, 20, 50, 30).getTime())
    .build();

// 3.定义一个 JobDetail
JobDetail job = JobBuilder.newJob(MyJob.class)
    .withIdentity("测试任务1", "test")
    .usingJobData("data", "JobData_coderhuye")
    .build();

// 4.在任务调度器中加入任务和触发器
scheduler.scheduleJob(job, trigger);
// 5.启动任务调度器
scheduler.start();
```

#### CalendarSchedule

```java
Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
TriggerBuilder<Trigger> triggerBuilder = TriggerBuilder.newTrigger();
Trigger trigger = triggerBuilder
    .withIdentity("trigger1", "group1")
    .startNow()
    .withSchedule(CalendarIntervalScheduleBuilder
                  .calendarIntervalSchedule()
                  .withIntervalInSeconds(1)
                 )
    .endAt(new GregorianCalendar(2020, 4, 21, 20, 50, 30).getTime())
    .build();

JobDetail job = JobBuilder.newJob(MyJob.class)
    .withIdentity("测试任务1", "test")
    .usingJobData("data", "JobData_coderhuye")
    .build();

scheduler.scheduleJob(job, trigger);
scheduler.start();
```

#### DailySchedule

```java
Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
TriggerBuilder<Trigger> triggerBuilder = TriggerBuilder.newTrigger();
Trigger trigger = triggerBuilder
    .withIdentity("trigger1", "group1")
    .startNow()
    .withSchedule(DailyTimeIntervalScheduleBuilder
                  .dailyTimeIntervalSchedule()
                  .startingDailyAt(TimeOfDay.hourAndMinuteOfDay(9, 0))
                  .endingDailyAt(TimeOfDay.hourAndMinuteOfDay(12, 0))
                  .onDaysOfTheWeek(MONDAY, TUESDAY, WEDNESDAY)
                  .withIntervalInSeconds(2)
                 )
    .build();

JobDetail job = JobBuilder.newJob(MyJob.class)
    .withIdentity("测试任务1", "test")
    .usingJobData("data", "JobData_coderhuye")
    .build();

scheduler.scheduleJob(job, trigger);
scheduler.start();
```

#### CronSchedule (重点)

##### Cron 表达式
范例：0/10 * * * *

**参数细节**说明：
- 5 个占位符说明

| 位置 | 含义       | 范围 | 特殊值        |
| ---- | ---------- | ---- | ------------- |
| 1    | 秒         | 0-59 | , - * /       |
| 2    | 分         | 0-59 | , - * /       |
| 3    | 时         | 0-23 | , - * /       |
| 4    | 日         | 1-31 | , - * ? / L W |
| 5    | 月         | 1-12 | , - * /       |
| 6    | 星期       | 1-7  | , - * ? / L W |
| 7    | 年（可选） |      | , - * /       |

- 特殊符号说明

| 符号 | 含义                                                         |
| :--: | ------------------------------------------------------------ |
|  *   | 表示**所有可能值**。                                         |
|  -   | 表示一个**范围**，如小时字段中使用 “10-12”，表示从 10 点 到 12 点。 |
|  ,   | 表示一个**列表值**，如星期字段中使用 “MON,WED,FRI” 表示星期一、星期三和星期五。 |
|  /   | 表示一个**等步长序列**，x/y 中 x 为起始值，y 为增量值。      |
|  ?   | 表示**不确定值**，只用于**日期、星期**。                     |
|  L   | 表示**最后 “Last**”，只用于**日期、星期**。用于日期表示该月最后一天；用于星期表示星期六(7)，若前面还有一个数字则表示该月最后一个星期几，如 6L 表示该月最后一个周五。 |
|  W   | 表示**离该日期最近的工作日**，只能出现在**日期**，是对前导日期的修饰。例如 15W 表示离该月 15 号最近的工作日，若该月 15 号是星期六，则匹配 14 号星期五；如果 15 日是星期日，则匹配 16 号星期一；如果 15 号是星明二，那结果就是 15 号星明二。注意**关联的匹配日期不能够跨月**，如指定 1W，如果 1 号是星期六，结果匹配的是 3 号星期一，而非上个月最后的那天。W 字符串只能指定单一日期，而不能指定日期范围。 |
|  LW  | 表示当月最后一个工作日，用于**日期**。                       |

- 示例

| 表达式                   | 含义                                                         |
| ------------------------ | ------------------------------------------------------------ |
| 秒 分 时 日 月 周        |                                                              |
| 0 0 12 * * ?             | 每天 12 点执行                                               |
| 0 15 10 * * ?            | 每天 10:15 执行                                              |
| 0 15 10 * * ? 2020       | 2020 年 每天 10:15 执行                                      |
| 0 * 14 * * ?             | 每天 14 点到 15 点之间，每分钟执行一次，start:14:00,end:14:59 |
| 0 0/5 14 * * ?           | 每天 14 点到 15 点之间，每 5 分钟执行一次，start:14:00,end:14:55 |
| 0 0/5 14,18 * * ?        | 每天 14 点到 15 点，18 点到 19 点之间，每 5 分钟执行一次     |
| 0 0-5/2 14 * * ?         | 每天 14:00 到 14:05 点之间，每 2 分钟执行一次，              |
| 0 10,44 14 ? 3 WED       | 3 月每周三的 14:10 和 14:44，执行一次                        |
| 0 15 10 ? * MON-FRI      | 每周一，二，三，四，五的 10:15 执行                          |
| 0 15 10 15 * ?           | 每月 15 日 10:15 执行                                        |
| 0 15 10 L * ?            | 每月最后一天 10:15 执行                                      |
| 0 15 10 ? * 6L           | 每月最后一个星期五 10:15 执行（此时天必须是"?"）             |
| 0 15 10 ? * 6L 2020-2022 | 在 2020，2021，2022 年每月最后一个星期五 10:15 执行          |

[Cron在线生成](https://qqe2.com/cron)
## 2.Job并发

Job 是有可能并发执行的，如一个任务执行需要 10s，但任务执行间隔只有 5s，则有可能存在多个任务并发执行的情况，这是可使用 `@DisallowConcurrentExecution` 注解来解决该问题。
代码示例：
```java
@DisallowConcurrentExecution //不允许并发，如果任务间隔为1s，则每个任务执行需要5s
public class MyJob implements Job {
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        JobDetail detail = context.getJobDetail();
        String name = detail.getKey().getName();
        String group = detail.getKey().getGroup();
        String job = detail.getJobDataMap().getString("data");
        System.out.println("Job执行，Job名:" + name + " | Group: " + group + " | data:" + job + " | "
                + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
    }
}
```

## 3.Spring Boot整合Quartz
### 3.1 导入依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```

### 3.2 创建自定义 Job 类实现任务具体执行逻辑。

`job/MyQuartzJob.java`
```java
public class MyQuartzJob extends QuartzJobBean {
    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        JobDetail detail = context.getJobDetail();
        String name = detail.getKey().getName();
        String group = detail.getKey().getGroup();
        String job = detail.getJobDataMap().getString("data");
        System.out.println("MyQuartzJob执行，Job名:" + name + " | Group: " + group + " | data:" + job + " | "
                + LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
    }
}
```

### 3.3 配置Quartz
`config/QuartzConfig.java`
```java
@Configuration
@EnableScheduling
public class SchedulerCustomizer implements SchedulerFactoryBeanCustomizer {
    @Override
    public void customize(SchedulerFactoryBean schedulerFactoryBean) {
        schedulerFactoryBean.setStartupDelay(2);
        schedulerFactoryBean.setAutoStartup(true);
        schedulerFactoryBean.setOverwriteExistingJobs(true);
    }
}
```

### 3.4 启动任务
`controller/QuartzController.java`
```java
@RestController
public class Controller {
    @Autowired
    private Scheduler scheduler;

    @RequestMapping(value = "/test")
    public String save() throws Exception {
        buildMyQuartzJob();
        return "Success";
    }

    private void buildMyQuartzJob() throws SchedulerException {
        TriggerBuilder<Trigger> triggerBuilder = TriggerBuilder.newTrigger();
        Trigger trigger = triggerBuilder
                .withIdentity("trigger1", "group1")
                .startNow()
                .withSchedule(CronScheduleBuilder
                        .cronSchedule("* * * * * ?")
                )
                .build();

        JobDetail job = JobBuilder.newJob(MyQuartzJob.class)
                .withIdentity("测试任务1", "test")
                .usingJobData("data", "JobData_coderhuye")
                .build();

        scheduler.scheduleJob(job, trigger);
    }
}
```