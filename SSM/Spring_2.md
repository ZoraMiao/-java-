# 3、Spring与AOP
## 3.2、AOP概述
### 3.2.1、AOP简介
* AOP(Aspect Orient Programming)，面向切面编程，是面向对象编程OOP的一种补充。面向对象编程是从静态角度考虑程序的结构，而面向切面编程是从动态考虑程序运行过程。
* AOP底层，就是采用动态代理模式是实现的。采用了两种代理：**JDK的动态代理** 与**CGLIB的动态代理**。
* 面向切面编程，就是将交叉业务逻辑封装成切面，利用AOP容器的功能将切面**织入**到主业务逻辑中。所谓交叉业务逻辑是指，通用的、与主业务逻辑无关的代码，如安全检查、事物、日志等。
* 若不使用AOP，则会出现代码纠缠，即交叉业务逻辑与主业务逻辑混合在一起。这样，会使主业务逻辑变得混杂不清。
### 3.2.2、AOP编程术语
#### （1）切面（Aspect）
* 切面泛指交叉业务逻辑。事务处理、日志处理等就可以理解为切面。常用的切面有通知与顾问。实际就是对主业务逻辑的一种增强。
#### （2）织入（Weaving）
* 是指将切面代码插入到目标对象的过程。
#### （3）连接点（JoinPoint）
* 指可以被切面织入的方法。通常业务接口与中的方法均为连接点。
#### （4）切入点（Pointcut）
* 指切面具体织入的方法。例：在StudentServiceImpl类中，若doSome（）将被增强，而doOther（）不被增强，则doSome（）为切入点，而doOther（）仅为连接点。
* 被标记为final的方法是不能作为连接点与切入点的。因为做种的是不能被修改的，不能被增强的。
#### （5）目标对象（Target）
* 是指将要被增强的对象。即包含主业务逻辑的类的对象。如StudentServiceImp的对象若被增强，则该类称为目标类，该类对象称为目标对象。当然，不能被增强，也就无所谓目标不目标了。
#### （6）通知（Advice）
* 通知是切面的一种实现，可以完成简单织入功能（织入功能就是在这里完成的）。换个角度讲，**通知定义了增强代码切入到目标代码的时间点** 是目标方法执行之前执行，还是之后执行等。通知类型不同，切入时间不同。
* **切入点定义切入得位置，通知定义切入的时间**
#### （7）顾问（Advisor）
* 是切面的另一种实现，能够将通知以更为复杂的方式织入到目标对象中，是将通知包装为更复杂切面的装配器。
## 3.3、通知详解
### 3.3.1、通知Advice
#### （1）前置通知MethodBeforeAdvice
* 定义前置通知，需要实现MethodBeforeAdvice接口。该接口有一个方法before()，会在目标方法执行之前执行
* 前置通知的特点：
  * 在目标方法执行之前先执行
  * 不改变目标方法的执行流程，前置通知代码不能阻止目标方法执行
  * 不改变目标方法执行的结果
#### （2）后置通知AfterReturningAdvice
* 定义后置通知，需要实现接口AfterReturningAdvice。该接口中有一个方法afterReturning()，会在目标方法执行之后执行。
* 后置通知的特点：
  * 在目标方法执行之后执行。
  * 不改变目标方法的执行流程，后置通知代码不能阻止目标方法执行。
  * 不改变目标方法执行的结果
#### （3）环绕通知MethodInterceptor
* 定义环绕通知，需要实现MethodInterceptor接口。环绕通知，也叫方法拦截器，可以在目标方法调用之前及之后做处理，可以改变目标方法的返回值，也可以改变程序执行流程。
#### （4）异常通知ThrowsAdvice
* 定义异常通知，需要实现ThrowsAdvice接口。在程序有异常的时候执行。
### 3.4、顾问Advisor
* 通知（Advice）是Spring提供的一种切面（Aspect）。但其功能过于简单：只能将切面织入到目标类的所有目标方法中，无法完成将切面织入到指定目标方法中。
* 顾问（Advisor）是Spring提供的另一种切面。其可以完成更为复杂的切面织入功能。**PointcutAdvisor是顾问的一种，可以指定具体的切入点。顾问将通知进行了包装，会根据不同的通知类型，在不同的时间点，将切面织入到不同的切入点。**
  * PointcutAdvisor接口有两个较为常用的实现类：
    * NameMatchMethodPointcutAdvisor 名称匹配方法切入点顾问
    * RegexpMethodPointcutAdvisor 正则表达式匹配方法切入点顾问
#### 3.4.1、名称匹配方法切入点顾问
* NameMatchMethodPointcutAdvisor，即名称匹配的方法顾问。容器可以根据目标类中的简单目标方法名来设置切入点。
#### 3.4.2、正则表达式匹配方法切入点顾问
* RegexpMethodPointcutAdvisor，即正则表达式方法顾问。容器可根据正则表达式来设置切入点。注意，**与正则表达式进行匹配的对象是接口中的方法名（全限定方法名），而非目标类（接口的实现类）的方法名。**
* 正则表达式中各符号的含义：
> .（点号）：表示任意单个字符<br>
  +（加号）：表示前一个字符出现一次或多次<br>
  \*（星号）：表示前一个字符出现0次或多次<br>
  eg:(.\*do.\*):表示方法全名（包括包名、接口名、方法名）中包含do的方法作为切入点方法
### 3.5、自动代理生成器
* 前面代码中所使用的代理对象，均是由ProxyFactoryBean代理工具类生成的。而该代理工具类存在着如下缺点：
  * （1）若存在多个对象，就需要使用多次ProxyFactoryBean来创建代理对象，这会使配置文件变得臃肿，不便于管理
  * （2）用户真正想调用的是目标对象，而真正可以调用的却是代理对象，这不符合正常逻辑
* Spring提供了自动代理生成器，用于解决ProxyFactoryBean的问题。常用的自动代理生成器有两个：
> 默认advisor自动代理生成器<br>
  Bean名称自动代理生成器
* 需要注意的是，自动代理生成器均继承自Bean后处理器BeanPostProcessor。容器中所有Bean在初始化时均会自动执行Bean后处理器中的方法，故其无需id属性。所以**自动代理生成器的Bean也没有id属性，客户类直接使用目标对象bean的id **。
#### 3.5.1、默认advisor自动代理生成器DefaultAdvisorAutoProxyCreator
* 用法(XML配置文件中):<bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>
* 这种代理也存在三个问题：
  * 1）不能选择目标方法
  * 2）不能选择切面类型，切面只能是advisor（顾问）
  * 3）不能选择advisor ，所以advisor均将被作为切面织入到目标方法
#### 3.5.2、Bean名称自动代理器
* 根据bean的id，来为符合相应名称的类生成相应代理对象，且切面既可以是顾问Advisor又可以是通知Advice。
```
用法：
  <bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
          <property name="beanNames" value="myService"/>
          <!-- <property name="interceptorNames" value="myAfter"/> -->
          <property name="interceptorNames" value="myAdvisor"/>
  </bean>
```
### AspectJ对AOP的实现（重点）
* 对于AOP这种编程思想，很多框架都进行了实现。Spring就是其中之一，可以完成面向切面编程。然而，AspectJ也实现了AOP的功能，且其实现方式更加简洁，使用更加方便，而且还支持注解式开发。所以，Spring又将AspectJ的对于AOP的实现也引入到了自己的框架中。
* 在Spring中使用AOP开发时，一般使用AspectJ的实现方式。
#### 3.6.1、AspectJ简介
* AspectJ是一个面向切面的框架，它扩展了java语言。AspectJ定义了AOP语法，它有一个专门的编译器用来生成遵守java字节编码规范的Class文件
#### 3.6.2、AspectJ的通知类型
* 常用的通知类型有五种：
  * 1）前置通知
  * 2）后置通知
  * 3）环绕通知
  * 4）异常通知
  * 5）最终通知
* 其中最终通知是指，无论程序执行是否正常，该通知都会执行。类似try...catch中的finally代码块。
#### 3.6.3AspectJ的切入点表达式
* 除了六中通知，还定义了专门的表达式用于指定切入点。表达式原型是：
> execution(访问权限类型，返回值类型**（不可省）**，全限定性类名，方法名（参数名）**（不可省）**，抛出异常类型)
* 表达式中符号的含义：
  * \* : 0至多个任意字符
  * .. : 用在方法参数中，表示任意多个参数，用在包名后，表示当前包及其子包路径
  * + : 用在类名后，表示当前类及其子类，用在接口后，表示当前接口及其实现类
  * eg1：execution（\* \*.service.\*.\*(..)）：指定只有一级包下的service子包下的所有类（接口）中所有方法为切入点
  * eg2：execution(\* \*..ISomeService.\*(..))：指定所有包下的ISomeService接口中所有方法为切入点
#### 3.6.4、AspectJ的开发环境
* **导入两个jar包**
  * AspectJ是专门针对AOP问题的，所以其运行时需要AOP环境的，即需要之前的AOP的两个jar包。另外，还需要AspectJ自身的jar包：在Spring支持库解压目录中的子目录org.aspectj下有两个子包
#### 3.6.5、AspectJ基于注解的AOP实现
* AspectJ提供了一注解方式对于AOP的实现
```java
@Aspect  //表示当前类为切面
public class MyAspect {

	@Before("execution(* *..ISomeService.doFirst(..))")
	public void before() {
		System.out.println("执行前置通知方法");
	}
	@Before("execution(* *..ISomeService.doFirst(..))")
	public void before(JoinPoint jp) {
		System.out.println("执行前置通知方法:jp="+jp);
	}
	
	@AfterReturning("execution(* *..ISomeService.doSecond(..))")
	public void afterReturning() {
		System.out.println("执行后置通知方法");
	}
	@AfterReturning(value="execution(* *..ISomeService.doSecond(..))",returning="result")  //returning内的值任意
	public void afterReturning(Object result) {//参数名必须和returning的值保持一致，才能获取目标方法的返回值
		System.out.println("执行后置通知方法:result="+result);
	}
	
	@Around("execution(* *..ISomeService.doSecond(..))")
	public Object myAround(ProceedingJoinPoint pjp) throws Throwable {
		System.out.println("执行环绕通知：目标方法前");
		Object result = pjp.proceed();
		System.out.println("执行环绕通知：目标方法后");
		if(result != null)
			result = ((String)result).toUpperCase();
		return result;
	}
	
	@AfterThrowing("execution(* *..ISomeService.doThird(..))")
	public void myAfterThrowing() {
		System.out.println("执行异常通知");
	}
	@AfterThrowing(value="doThirdPointcut()", throwing="ex")
	public void myAfterThrowing(Exception ex) {
		System.out.println("执行异常通知:ex="+ex.getMessage());
	}
	
	@After("doThirdPointcut()")
	public void myFinally() {
		System.out.println("执行最终通知");
	}
	
	//定义一个可重用的切入点,名称为doThirdPointcut()
	@Pointcut("execution(* *..ISomeService.doThird(..))")
	public void doThirdPointcut() {}
}
```
#### 3.6.6、AspectJ基于XML的AOP实现
* 切面就是一个POJO类，而用于增强的方法就是普通方法。通过配置文件，将切面中的功能增强织入到了目标类的目标方法中。
  ```
   <!-- 注册切面 -->
       <bean id="myAspect" class="com.bjpowernode.xml.MyAspect"/>
       <aop:config>
       	 <aop:pointcut expression="execution(* *..ISomeService.doFirst(..))" id="doFirstPointcut"/>
       	 <aop:pointcut expression="execution(* *..ISomeService.doSecond(..))" id="doSecondPointcut"/>
       	 <aop:pointcut expression="execution(* *..ISomeService.doThird(..))" id="doThirdPointcut"/>
       	 <aop:aspect ref="myAspect">
       	 <!-- 若只有一个前置通知，method直接等于方法名，若有两个及以上，则就要指定方法中参数的全限定名 -->
       	 	<aop:before method="before" pointcut-ref="doFirstPointcut"/>
       	 	<aop:before method="before(org.aspectj.lang.JoinPoint)" pointcut-ref="doFirstPointcut"/>
       	 
       	 	<aop:after-returning method="afterReturning" pointcut-ref="doSecondPointcut"/>
       	 	<!-- returning="result"切面中方法的参数名称要与result保持一致 -->
       	 	<aop:after-returning method="afterReturning(java.lang.Object)" pointcut-ref="doSecondPointcut" returning="result"/>
       	
       		 <aop:around method="myAround" pointcut-ref="doSecondPointcut"/>
       		
       	 	<aop:after-throwing method="myAfterThrowing" pointcut-ref="doThirdPointcut"/>
       		 <aop:after-throwing method="myAfterThrowing(java.lang.Exception)" pointcut-ref="doThirdPointcut" throwing="ex"/>
       	 
       	 	<aop:after method="myFinally" pointcut-ref="doThirdPointcut"/>
       	 </aop:aspect>
       </aop:config>
  ```



