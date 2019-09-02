---
layout: blog
istop: true
jishu: true
background-image: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1509503652&di=cf9ab469d47904d5a9ab910f3134110f&imgtype=jpg&er=1&src=http%3A%2F%2Fwww.sz886.com%2Feditor%2Fimage%2F20170507%2F20170507175839_7806.png
category: Spring
title: Spring AOP从入门到放弃之自定义注解收集系统日志
tags:
- AOP
- Spring AOP
- 自定义注解
- 收集系统日志
date: 2017-10-27 20:25:54
---

# 希望的效果为


## 需求
用户点击了某个界面，请求了后台某个接口。接口请求到后台后，记录请求的数据到数据库中。


## 实现方式
1、自定义一个注解，被加注解的方法，请求的数据被保存下来
2、定义一个aop 去拦截被注解的方法
3、写一个线程池、执行拦截后的逻辑。也就是保存到数据库中


## 效果图
![这里写图片描述](http://img.blog.csdn.net/20171027213635164?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

查看到刚刚请求用户列表界面的执行情况

![这里写图片描述](http://img.blog.csdn.net/20171027213649280?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbmppYW5hbmRpeWk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



# 实现步骤

## 1、自定义注解

```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SLog {
	String value() default "";
}

```

## 定义一个aop拦截注解

```

@Aspect
@Component
public class SLogAspect {

    /**
     * 保存日志到数据库的线程池
     */
    ThreadFactory threadFactory = new ThreadFactoryBuilder().setNameFormat("SLogAspect-Thread-%d").build();

    ExecutorService executor = new ThreadPoolExecutor(5,200,0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<Runnable>(1024),
            threadFactory,
            new ThreadPoolExecutor.AbortPolicy());


    @Pointcut("@annotation(com.slife.annotation.SLog)")
    public void logPointCut() {
    }

    @Around("logPointCut()")
    public Object around(ProceedingJoinPoint point) throws Throwable {
        long beginTime = System.currentTimeMillis();
        // 执行方法
        Object result = point.proceed();
        // 执行时长(毫秒)
        long time = System.currentTimeMillis() - beginTime;

        // 获取request
        HttpServletRequest request = ServletUtils.getHttpServletRequest();
        //获取请求的ip
        String ip = IPUtils.getIpAddr(request);
        SaveLogTask saveLogTask = new SaveLogTask(point, time, ip);
        //保存日志到数据库
        executor.execute(saveLogTask);

        return result;
    }


}

```

## 线程池执行保存数据到数据库

```

/**
 *
 * @author chen
 * @date 2017/9/19
 * <p>
 * Email 122741482@qq.com
 * <p>
 * Describe:
 */
public class SaveLogTask implements Runnable {


    private SlifeLogDao slifeLogDao = ApplicationContextRegister.getBean(SlifeLogDao.class);

    private ProceedingJoinPoint joinPoint;
    private long time;
    private String ip;

    public SaveLogTask(ProceedingJoinPoint point, long time, String ip) {
        this.joinPoint = point;
        this.time = time;
        this.ip = ip;
    }

    @Override
    public void run() {
        saveLog(joinPoint, time, ip);
    }

    /**
     * 保存日志 到数据库
     *
     * @param joinPoint
     * @param time
     */
    private void saveLog(ProceedingJoinPoint joinPoint, long time, String ip) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();

        SlifeLog slifeLog = new SlifeLog();
        SLog sLog = method.getAnnotation(SLog.class);

        if (slifeLog != null) {
            // 注解上的描述
            slifeLog.setMsg(sLog.value());
        }
        // 请求的方法名
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = signature.getName();
        slifeLog.setSrc(className + "." + methodName + "()");

        // 请求的参数
        Object[] args = joinPoint.getArgs();
        try {
            String params = JSON.toJSONString(args[0]);
            slifeLog.setParams(params);
        } catch (Exception e) {

        }

        // 设置IP地址
        slifeLog.setIp(ip);
        // 用户名
        ShiroUser currUser = SlifeSysUser.ShiroUser();

        if (null == currUser) {
            if (null != slifeLog.getParams()) {
                slifeLog.setName(slifeLog.getParams());
                slifeLog.setLoginName(slifeLog.getParams());
            } else {
                slifeLog.setName("获取用户信息为空");
                slifeLog.setLoginName("获取用户信息为空");
                slifeLog.setCreateId(-1L);
            }
        } else {
            slifeLog.setName(currUser.getName());
            slifeLog.setLoginName(currUser.getUsername());

        }

        slifeLog.setUseTime(time);

        // 保存系统日志
        slifeLogDao.insert(slifeLog);
    }
}

```


## 给需要的方法加注解

```
    @SLog("获取用户列表数据")
    @ApiOperation(value = "获取用户列表数据", notes = "获取用户列表:使用约定的DataTable")
    @PostMapping(value = "/list")
    @ResponseBody
    public DataTable<SysUser> list(@RequestBody DataTable dt, ServletRequest request) {
        return sysUserService.pageSearch(dt);
    }




  @SLog("获取用户列表数据")

```

简单的一个 记录请求的日志 实现。

# 说明

这里把保存到数据库的逻辑写到了一个线程池中，主要是不希望记录日志的逻辑影响了用户请求数据接口的逻辑，和性能。





# **[点击获取阿里云优惠券](https://promotion.aliyun.com/ntms/yunparter/invite.html?userCode=vf2b5zld)**

