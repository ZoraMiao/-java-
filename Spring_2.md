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
> 用法：<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
          <property name="beanNames" value="myService"/>
          <!-- <property name="interceptorNames" value="myAfter"/> -->
          <property name="interceptorNames" value="myAdvisor"/>
       </bean>

