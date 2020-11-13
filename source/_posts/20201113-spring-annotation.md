---
title: spring-annotation
date: 2020-11-13 17:01:15
tags: java
categories: 后端
---


## 基于spring的注解开发

类似python的装饰器可以实现一下功能

- 动态修改属性
- 动态新增功能（不侵入原代码）
- 动态改变执行逻辑

### spring 配置略过

### 开发注解

    1. 定义注解

        ```java
        @Documented
        @Target({ElementType.PARAMETER, ElementType.FIELD, ElementType.LOCAL_VARIABLE})
        @Retention(RetentionPolicy.RUNTIME)
        public @interface Patch {
            String  key() default "";
            String  value() default "";
        }

        ```

    2. 注解实现

        ```java
        import org.aspectj.lang.JoinPoint;
        import org.aspectj.lang.ProceedingJoinPoint;
        import org.aspectj.lang.annotation.*;
        import org.aspectj.lang.reflect.MethodSignature;
        import org.springframework.stereotype.Component;
        import org.springframework.util.StopWatch;

        /**
        * @Author 杨帅
        * @Date 2020/11/12 23:09
        * @Version 1.0
        */

        @Component
        @Aspect
        public class PatchAspect {

            @After("Patch()")
            public void after(JoinPoint joinPoint)  {
                // 记录参数
            }

            @AfterReturning(pointcut = "Patch()",returning="returnValue")
            public void afterJoinPoint(JoinPoint joinPoint ,Object returnValue){
                // 记录返回值
            }


            @Pointcut("@annotation(注解路径)")
            public void patch(){}

            @Around("Patch()")
            public void around(ProceedingJoinPoint joinPoint){
                try {

                    StopWatch sw = new StopWatch();
                    sw.start();
                    //获取方法参数
                    Object[] args = joinPoint.getArgs();
                    MethodSignature sign = (MethodSignature) joinPoint.getSignature();
                    String[] parameterNames = sign.getParameterNames();
                    System.out.println("parameter list: ");
                    for (int i = 0; i < parameterNames.length; i++) {
                        System.out.println(parameterNames[i] + " = " + args[i]);
                        // 这里可以通过参数名配合注解的值直接修改对应的参数 做拦截操作或者参数校验

                    }
                    //获取注解参数
                    Patch annotation =  sign.getMethod().getAnnotation(Patch.class);
                    System.out.println("value = " + annotation.value());
                    System.out.println("key = " + annotation.key());

                    /*
                    // 这里可以不知道注解的情况下获取注解
                    Annotation[][] annotations;
                    try {
                        annotations = pjp.getTarget().getClass().
                                getMethod(methodName, parameterTypes).getParameterAnnotations();
                    } catch (Exception e) {
                        throw new SoftException(e);
                    }

                    */

                    //执行方法
                    joinPoint.proceed();

                    //方法执行后逻辑
                    sw.stop(args);
                    System.out.println(sw.prettyPrint());

                } catch (Throwable throwable) {
                    throwable.printStackTrace();
                }
            }
        }


        ```

### 总结

注解本质意义上就是代码的注释，如果没有实现解析的逻辑那么仅仅是注释。
对于注释有两种方式，一种是编译直接扫描，一种是运行时反射。
注意：基于 java 注解的开发不同于动态语言的简单逻辑，很多时候需要在代码中有效的处理各种类型的判断以及各种未知

### 其他

[JAVA 注解的基本原理](https://www.cnblogs.com/yangming1996/p/9295168.html)
