---
layout: post
title:  "使用Spring  AOP切面实现方法执行时间日志打印"
date:   2022-06-25
last_modified_at: 2022-06-25
categories: [Java]
tags: [java8, jdk, aop, springframework]
---

利用Spring AOP切面编程，可以实现对注解的拦截。把自定义拦截注解标注在方法上，可实现对方法的埋点日志、执行时间日志、执行过程日志等功能。以下是具体实现方式。



1、在`pom.xml`文件中，添加AOP依赖。

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```



2、自定义一个注解

`@Retention(RetentionPolicy.RUNTIME)`表示注解加载class文件之后仍然存在，

`@Target(ElementType.METHOD)`表示该注解仅用于修饰方法的。

```
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExecTimeLog {

}
```



3、实现切面

**环绕通知：**proceedingJoinPoint.proceed()表示执行目标方法代码，在执行代码前后插入StopWatch的开启和结束，计算执行时间。

**时间计算：**根据具体需求，可以使用StopWatch输出微秒、毫秒、秒等单位来输出日志，也可以使用StopWatch的prettyPrint()或shortSummary()直接打印日志。

```
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.reflect.MethodSignature;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.util.StopWatch;

@Aspect
@Component
public class ExecTimeLogAspect {

    public static final Logger logger = LoggerFactory.getLogger(ExecTimeLogAspect.class);

    // 用于标注AOP方法拦截的注解包名
    @Around("@annotation(com.gx3s.backenddemo.aop.ExecTimeLog)")
    public Object methodTimeLogger(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        MethodSignature methodSignature = (MethodSignature) proceedingJoinPoint.getSignature();

        // 获取拦截的类名和方法名
        String className = methodSignature.getDeclaringType().getSimpleName();
        String methodName = methodSignature.getName();

        // 配置计时器ID，开始计算时间
        StopWatch stopWatch = new StopWatch(className + "->" + methodName);
        stopWatch.start(methodName);
        Object result = proceedingJoinPoint.proceed();
        stopWatch.stop();

        // 打印方法执行时间
        if (logger.isInfoEnabled()) {
            // 可以替换时间单位
            logger.info("方法：{}，执行时间：{}秒",stopWatch.getId(), stopWatch.getTotalTimeSeconds());
        }

        return result;
    }

}
```



4、使用`@ExecTimeLog`注解标注到方法上。

```
@ExecTimeLog
@ApiOperation(value = "商品统计信息", response = ProductStatsVo.class)
@GetMapping("stats")
    public Res getProductStats() {
    ProductStatsVo vo = productService.getProductStats();
    return Res.ok("返回成功").data(vo);
}
```



5、在日志中，可以看到方法执行时间。

```
2022-06-10 16:09:20.923  INFO 37052 --- [io-12003-exec-1] com.gx3s.backenddemo.aop.LoggingAspect   : 方法：ProductController->getProductStats，执行时间：0.1257083秒
```

