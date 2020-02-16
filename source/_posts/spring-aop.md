---
title: Spring Aop AspectJ 摘录笔记
date: 2020-02-16 12:41:18
tags: 
	- AspectJ
	- Aop
categories: Spring
---

原文链接：https://juejin.im/post/5a55af9e518825734d14813f

## @AspectJ

**AspectJ是一个AOP框架，它能够对java代码进行AOP编译（一般在编译期进行），让java代码具有AspectJ的AOP功能（当然需要特殊的编译器），可以这样说AspectJ是目前实现AOP框架中最成熟，功能最丰富的语言，更幸运的是，AspectJ与java程序完全兼容，几乎是无缝关联，因此对于有java编程基础的工程师，上手和使用都非常容易.** 



其实**AspectJ单独就是一门语言**，它需要专门的编译器(ajc编译器). Spring AOP 与ApectJ的目的一致，都是为了统一处理横切业务，但与AspectJ不同的是，Spring AOP并不尝试提供完整的AOP功能(即使它完全可以实现)，Spring AOP 更注重的是与Spring IOC容器的结合，并结合该优势来解决横切业务的问题，因此在AOP的功能完善方面，相对来说AspectJ具有更大的优势，同时,Spring注意到AspectJ在AOP的实现方式上依赖于特殊编译器(ajc编译器)，因此Spring很机智回避了这点，**转向采用动态代理技术的实现原理来构建Spring AOP的内部机制（动态织入），这是与AspectJ（静态织入）最根本的区别**。在AspectJ 1.5后，引入@Aspect形式的注解风格的开发，Spring也非常快地跟进了这种方式，因此Spring 2.0后便使用了与AspectJ一样的注解。请注意，**Spring 只是使用了与 AspectJ 5 一样的注解，但仍然没有使用 AspectJ 的编译器，底层依是动态代理技术的实现，因此并不依赖于 AspectJ 的编译器**。



所以，Spring AOP虽然是使用了那一套注解，其实实现AOP的底层是使用了动态代理(JDK或者CGLib)来动态植入。



> - 切点:定位到具体方法的一个表达式
> - 切面: 切点+建言
> - 建言(增强):定位到方法后干什么事



## 前置通知 @Before

前置通知通过@Before注解进行标注，并可直接传入切点表达式的值，该通知在目标函数执行前执行，注意JoinPoint，是Spring提供的静态变量，通过joinPoint 参数，可以获取目标对象的信息,如类名称,方法参数,方法名称等，该参数是可选的。

```java
@Before("execution(* com.iogogogo.aop.service.*.*(..))")
public void before(JoinPoint joinPoint) {
  log.info("前置通知 CLASS_METHOD : {}", joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
  log.info("前置通知 ARGS : {}", Arrays.toString(joinPoint.getArgs()));
}
```



## 后置通知 @AfterReturning

通过@AfterReturning注解进行标注，该函数在目标函数执行完成后执行，并可以获取到目标函数最终的返回值returnVal，当目标函数没有返回值时，returnVal将返回null，必须通过returning = “returnVal”注明参数的名称而且必须与通知函数的参数名称相同。请注意，在任何通知中这些参数都是可选的，需要使用时直接填写即可，不需要使用时，可以完成不用声明出来。**当出现异常则不执行。**

```java
@AfterReturning(value = "execution(* com.iogogogo.aop.service.*.*(..))", returning = "returnVal")
public void AfterReturning(JoinPoint joinPoint, Object returnVal) {
  log.info("后置通知 CLASS_METHOD : {}", joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
  log.info("后置通知 ARGS : {}", Arrays.toString(joinPoint.getArgs()));
  log.info("后置通知 returnVal {} ", returnVal);
}
```





## 异常通知 @AfterThrowing

该通知只有在异常时才会被触发，并由throwing来声明一个接收异常信息的变量，同样异常通知也用于Joinpoint参数，需要时加上即可。

```java
@AfterThrowing(value = "execution(* com.iogogogo.aop.service.*.*(..))", throwing = "e")
public void afterThrowable(Throwable e) {
  log.error("异常: ", e);
}
```





## 最终通知 @After

该通知有点类似于finally代码块，只要应用了无论什么情况下都会执行。

```java
@After("execution(* com.iogogogo.aop.service.*.*(..))")
public void after(JoinPoint joinPoint) {
  log.info("最终通知 CLASS_METHOD : {}", joinPoint.getSignature().getDeclaringTypeName() + "." + joinPoint.getSignature().getName());
  log.info("最终通知 ARGS : {}", Arrays.toString(joinPoint.getArgs()));
}
```





## 环绕通知 @Around

环绕通知既可以在目标方法前执行也可在目标方法之后执行，更重要的是环绕通知可以控制目标方法是否指向执行，但即使如此，我们应该尽量以最简单的方式满足需求，在仅需在目标方法前执行时，应该采用前置通知而非环绕通知。第一个参数必须是ProceedingJoinPoint，通过该对象的proceed()方法来执行目标函数，proceed()的返回值就是环绕通知的返回值。同样的，ProceedingJoinPoint对象也是可以获取目标对象的信息、如类名称、方法参数、方法名称等等。

```java
@Around("execution(* com.iogogogo.aop.service.*.*(..))")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
  log.info("环绕通知前....");
  // 执行目标函数
  Object obj = joinPoint.proceed();
  log.info("环绕通知后....{}", obj);
  return obj;
}
```



## execution 语法

```java
// scope ：方法作用域，如public,private,protect
// returnt-type：方法返回值类型
// fully-qualified-class-name：方法所在类的完全限定名称
// parameters 方法参数
execution(<scope> <return-type> <fully-qualified-class-name>.*(parameters))
```

- `execution(* com.iogogogo.*.*(..))` com.iogogogo包下所有类的所有方法
- `execution(* com.iogogogo.Dog.*(..))` Dog类下的所有方
- `execution(* com.iogogogo.Dog*.*(..))` Dog开头的类下的所有方法





## 切点 @Pointcut

在使用切入点时，还可以抽出来一个`@Pointcut`来供使用，可以避免重复的execution在不同的注解里写很多遍。

```java
@Pointcut("execution(public * com.iogogogo.aop.service.*.*(..))")
public void pointcut() {
}
```

使用

```java
@Before(value = "pointcut()")
public void before() {
  log.info("使用@Pointcut 的前置通知");
}
```



## AOP切面的优先级

有时候，我们对一个方法会有多个切面的问题，这个时候还会涉及到切面的执行顺序的问题。

我们可以定义每个切面的优先级， Spring 中提供注解 `@Order(i)` ，当 `i` 的值越小，优先级越高。



## 环绕和前后置通知的区别

对于有变量缓存需求，线程安全的应用场景，前后置通知实现比较困难，而环绕通知实现就非常容易；

https://www.cnblogs.com/yaphetsfang/articles/11378821.html