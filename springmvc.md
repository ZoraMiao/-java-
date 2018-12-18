### 路径问题
* 一个URL例如：`http://127.0.0.1:8080/01-primary/index.jsp`，最后一个/后面的叫做资源名称（index.jsp），最后一个/前面所有的路径叫做访问路径(`http://127.0.0.1:8080/01-primary`)
* **绝对路径** ：根据给出的请求路径可以准确定位到资源的路径。对于计算机中的web应用的绝对路径，则是**指带请求协议的路径**。例：`http://127.0.0.1:8080/01-primary/index.jsp`。
* **相对路径** ： 指仅根据请求路径无法准确定位资源的路径。相对路径必须要结合其参照路径才可组成可以准确定位资源的绝对路径。**在进行资源访问时，必须要将相对路径转换为绝对路径才可完成资源的准确定位。** 绝对路径 = 相对路径 + 参照路径
  * 以`/`开头的：①如果出现在java代码或配置文件中，这个路径叫做后台路径。②如果出现在静态页面或动态页面的静态部分（jsp页面中除了java代码块其他都叫静态部分），这个路径叫做前台路径。
      * 前台路径的参照路径是**web服务器的根（`http://127.0.0.1:8080`）**
      * 后台路径的参照路径是**web应用的根（`http://127.0.0.1:8080/01-primary`）**
  * 不以`/`开头的：无论是前台路径还是后台路径，它的参照路径是 **当前的访问路径(去掉资源名称后的路径)** ，例如：`http://127.0.0.1:8080/01-primary/index.jsp`，它的当前访问路径是`http://127.0.0.1:8080/01-primary`
  * **后台路径的特例：** 当代码中使用response的sendRedirect()方法进行重定向时，其参照路径不是web应用的根路径，而是web服务器的根路径。
* **注：jsp页面中的base标签会自动在当前页面的不以/开头的路径前加上basePath路径，使其变为绝对路径**
### SpringMVC也叫Spring web mvc，属于表现层的框架，在三层架构中属于View层。
## 2、配置式开发
* 是指“处理器类是程序员手工定义的、实现了特定接口的类，然后再在SpringMVC配置文件中对该类进行显示的、明确的注册”的开发方式。
### 2.1.1 处理器映射器HandlerMapping
* HandlerMapping接口负责根据request请求找到对应的Handler处理器及Interceptor拦截器，并将它们封装在HandlerExecutionChain对象中，返回给中央调度器。其常用的实现类有两种：
  * BeanNameUrlHandlerMapping
  * SimpleUrlHandlerMapping
#### (1)BeanNameUrlHandlerMapping（默认）
* 会根据请求的url与spring容器中定义的处理器bean的name（id）属性值进行匹配，从而在spring容器中找到处理器bean实例。
* 根据源码，对于处理器的Bean的名称，必须以“/”开头，否则无法加入到urls数组中。而urls数组中的url则是中央调度器用于判断“该url所对应的类是否作为处理器交给处理器适配器进行适配”的依据。这也是处理器与其它普通Bean的区别。
* 使用BeanNameUrlHandlerMapping映射器有两点明显不足：
  * ①处理器Bean的id为一个url请求路径，而不是Bean的名称，有些不伦不类；
  * ②处理器Bean的定义与请求url绑定在一起。若出现对个url请求同一个处理器的情况，就需要在Spring容器中配置对个处理器的`</bean>`。这将导致容器会创建多个该处理器类实例。
#### (2)SimpleUrlHandlerMapping
* 不仅可以将url与处理器的定义分离，还可以对url进行统一映射管理。
* 会根据请求的url与Spring容器中定义的处理器映射器子标签的key属性进行匹配。匹配上后，再将该key的value值与处理器bean的id进行匹配。从而在Spring容器中找到处理器bean。
```
  <bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
        	<property name="mappings">
        		<props>
        			<prop key="/my.do">myController</prop>
        		</props>
        	</property>
  </bean>
  <bean id="myController" class="com.zm.handler.MyController"/>
```
### 2.1.2处理器适配器HandlerAdapter
#### (1) SimpleControllerHandlerAdapter处理器适配器，需要用户自定义的处理器实现Controller接口<br>
#### (2) 所有实现了HttpRequestHandlerAdapter接口的处理器Bean，均是通过此适配器进行适配、执行的
* spring默认组建设置为：
  * 处理器映射器：BeanNameURLHandlerMapping和DefaultAnnotationHandlerMapping
  * 处理器适配器：HttpRequestHandlerAdapter、SimpleControllerHandlerAdapter和AnnotationMethodHandlerAdapter
  * 视图解析器：InternalResourceViewResolver
### 2.1.3 处理器
* 处理器除了实现Controller接口外，还可以继承自一些其它的类来完成一些特殊的功能。
#### （1）继承自AbstractController类
* 因为AbstractController类还继承自一个父类WebContentGenerator，所以具有supportMethods属性，可以设置支持的HTTP数据提交方式。默认支持GET、POST。**只有表单和AJAX可以进行POST提交，其他都是GET提交** 。
#### （2）继承MultiActionController
* ①这个类中有一个属性methodNameResolver，其具有默认值InternalPathMethodNameResolver（方法名解析器）,该解析器将方法名作为资源名称进行解析，那就意味着，提交请求时，要将方法名作为资源名称出现。
* ②PropertiesMethodNameResolver方法名解析器
  * 该方法名解析器中的 **方法名是作为URL资源名称中的一部分出现的**，即方法名并非单独作为一种资源名称出席那。例如请求时可以写成/xxx_doFirst，则会访问xxx所映射的处理器的doFirst()方法。
* ③ParameterMethodNameResolver方法名解析器
  * 该方法名解析器中的 **方法名作为请求参数的值出现**。例如请求时可以写为/xxx?ooo=doFirst，则会访问xxx所映射的处理器的doFirst()方法。其中ooo为该请求所携带的参数名，而doFirst则作为其参数值出现。
### 2.1.4 ModelAndView
* 即模型与视图，通过addObject()方法向模型中添加数据，通过setViewName()方法向模型添加视图名称。
#### （1）模型
* 本质上是HashMap
* HashMap是一个单向查找数组
* LinkedHashMap是一个双向查找数组
#### （2）视图
* 通过setViewName()指定视图名称。注意，这里的视图名称将会对应一个视图对象，一般是不会在这里直接写上要跳转的页面的。这个视图对象，将会被封装在ModelAndView中，传给视图解析器来解析，最终转换为相应的页面。但需要注意的是，这里的view对象本质仅仅是一个String而已。
* 当然若处理器方法返回的ModelAndView中并没有数据要携带，则可直接通过ModelAndView的带参数构造器将视图名称放入ModelAndView中。
### 2.2.5 视图解析器ViewResolver
* 视图解析器ViewResolver接口负责处理结果生成View视图。常用的实现类有四种：
#### （1）InternalResourceViewResolver视图解析器
* 该视图解析器用于完成对当前Web应用内部资源的封装与跳转，而对于内部资源的查找规则是，将ModelAndView中指定的视图名称与为视图解析器配置的前缀与后缀相结合的方式，拼接成一个Web应用内部资源路径。拼接规则是：前缀 + 视图名称 + 后缀
#### （2）BeanNameViewResolver视图解析器
* InternalResourceViewResolver解析器存在两个问题，使其使用很不灵活。
  * 只可以完成将内部资源封装后的跳转。但无法转向外部资源，如外部网页：淘宝等
  * 对于内部资源的定义，也只能定义一种格式的资源：存放于同一目录的同一文件类型的资源文件。
* BeanNameViewResolver，将资源封装为“Spring容器中注册的Bean实例”，ModelAndView通过设置视图名称为该Bean的id属性值来完成对该资源的访问。所以在springmvc.xml中，可以定义多个View视图Bean，让处理器中ModelAndView通过对这些Bean的id的引用来完成向View中封装资源的跳转。
```
	<!-- 定义内部资源视图   -->
        <bean id="internalview" class="org.springframework.web.servlet.view.JstlView">
       		<property name="url" value="/WEB-INF/jsp/welcome.jsp"/>
        </bean>	
		<!-- 定义外部资源视图   -->
        <bean id="taobao" class="org.springframework.web.servlet.view.RedirectView">
       		<property name="url" value="http://taobao.com"/>
        </bean>	   
		<!--  定义视图解析器      -->
        <bean class="org.springframework.web.servlet.view.BeanNameViewResolver"/>	   
        <bean id="/my.do" class="com.zm.handler.MyController"/>
```
#### （3）XMLViewResolver视图解析器
* 如果配置文件中视图配置很多时，会使配置文件变得很庞大，所以可以将视图的定义单独放在一个xml文件中，然后在配置文件中引入即可。
```
  <bean class="org.springframework.web.servlet.view.XmlViewResolver">
	<property name="location" value="classpath:myViews.xml"/>
  </bean>	
```
#### （4）ResourceBundleViewResolver视图解析器
* 视图除了可以放在xml文件外，还可以放过在properties文件中，如下：这个时候setViewName中添加的就是taobao或jd或internalview
properties文件：
```
taobao.(class)=org.springframework.web.servlet.view.RedirectView
taobao.url=http://www.taobao.com

jd.(class)=org.springframework.web.servlet.view.RedirectView
jd.url=http://www.jd.com

internalview.(class)=org.springframework.web.servlet.view.JstlView
internalview.url=/WEB-INF/jsp/welcome.jsp
```
配置文件：
```
 <bean class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
	<property name="basename" value="myViews"/>
	<!--如果xml中存在多个视图解析器，这个属性可以设置视图解析器的优先级，value必须是一个大于0的整数，值越小优先级越高-->
	<property name="order" value="3"/>
 </bean>	
```
#### （5）视图解析器的优先级
* 视图解析器有一个order属性，专门用于设置对个视图解析器的优先级。**数字越小，优先级越高。数字相同，先注册的优先级高**。一般不为InternalResourceViewResolver解析器指定优先级，即让其优先级是最低的。如上述的例子。
## 3、注解式开发（重点）
* 所谓SpringMVC的注解式开发是指，处理器是基于注解的类的开发。对于每一个定义的处理器，无需在配置文件中逐个注册，只需在代码中通过对类与方法的注解，便可完成注册。即注册替换的是配置文件中对于处理器的注册部分。
### 3.1、处理器的请求映射规则的定义
#### 3.1.1、 对请求URL的命名空间的定义（!）
* 如果处理器中很多地方的请求路径都包含/test，则可以在处理器的类前加上注解@RequestMapping("/test")，可以减少冗余
#### 3.1.2、请求URL中通配符的应用(了解)
#### 3.1.3、对请求提交方式的定义（了解）
* 对于@RequestMapping()，其有一个睡醒method，用于对被注解方法所处理请求的提交方式进行限制，即只有满足该method属性指定的提交方式的请求，才会执行该被注解方法。
* method属性取值为RequestMethod枚举常量，常用的为RequestMethod.GET与RequestMethod.POST，分别表示提交方式的匹配规则WieGET与POST提交。
* 几种默认请求方式的提交方式：
  * 表单请求       默认为GET，可以指定POST
  * AJAX请求       默认为GET，可以指定为POST
  * 地址栏请求     GET请求
  * 超链接请求     GET请求
  * src资源路径请求  GET请求
#### 3.1.4、对请求中携带参数的定义（了解）
### 3.2、处理器方法的参数
* 处理器方法中常用的参数有五类，这些参数会在系统调用时有系统自动赋值，即程序员可在方法内直接使用。
  * HttpServletRequest
  * HttpServletResponse
  * HttpSession
  * 用于承载数据的Model
  * 请求中所携带的请求参数
#### 3.2.1、逐个参数接收
* 只要保证请求参数名与该请求处理方法的参数名相同即可。
#### 3.2.2、请求参数中文乱码问题
* 在web.xml中添加字符过滤器
#### 3.2.3、校正请求参数名@RequestParam
* 解决表单提交的参数名与处理器方法中参数名不一致问题，做映射
```java
	public ModelAndView doFirst(@RequestParam("pname") String name, int age){}
```
#### 3.2.4、整体参数接收
* 将所有的参数定义成一个实体，然后保持表单的参数名与实体中定义的属性名相同，之后就可以在处理器的方法参数中，可以接受整体参数。
#### 3.2.5、域属性参数的接收
* 所谓域属性，即对象属性。当请求参数中的数据为某类对象域属性的属性值时，要求请求参数名为“域属性.属性”。例如：
```
  <!-- 域属性的接收 -->
  学校：<input type="text" name="school.sname">
  校址：<input type="text" name="school.address">
```
#### 3.2.6、路径变量@PathVariable
* 对于处理器方法中所接收的请求参数，可以来自于请求中所携带的参数，也可以来自于请求的URL中所携带的变量，即路径变量。不过此时，需要借助@PathVariable注解。
* @PathVariable在不指定参数的情况下，默认其参数名，即路径变量名与用于接收其值得属性名相同。若路径变量名与用于接收其值得属性名不同，则@PathVariable可通过参数指出路径变量名称。
### 3.3、处理器方法的返回值
#### 3.3.1、返回ModelAndView
* 若处理器方法处理完后，需要跳转到其他资源，且又要在跳转的资源间传递数据。此时处理器方法返回ModelAndView比较好。若要返回ModelAndView，则处理器方法中需要定义ModelAndView对象。
* 在使用时，若该处理器方法只是进行跳转而不传递数据，或只是传递数据而并不向任何资源跳转（如对页面的AJAX异步响应），此时若返回ModelAndView，则将总是有一部分多余：要么Model多余，要么View多余。即此时返回ModelAndView不合适。
#### 3.3.2、返回String
* 处理器方法返回的字符串可以指定逻辑视图名，通过视图解析器可以将其转换为物理视图地址。
##### （1）返回内部资源逻辑视图名
* 若要跳转的资源为内部资源，则视图解析器可以使用InternalResourceViewResolver内部资源视图解析器。此时处理器方法返回的字符串就是要跳转页面的文件名去掉文件扩展名后的部分。这个字符串与视图解析器中的prefix、suffix相结合，即可形成要访问的URL。
##### （2）返回View对象名
```
	<!-- 定义外部资源视图   -->
        <bean id="taobao" class="org.springframework.web.servlet.view.RedirectView">
       		<property name="url" value="http://taobao.com"/>
        </bean>	
		<!-- 定义外部资源视图   -->
        <bean id="jd" class="org.springframework.web.servlet.view.RedirectView">
       		<property name="url" value="http://jd.com"/>
        </bean>	
        
		<!--  定义视图解析器      -->
        <bean class="org.springframework.web.servlet.view.BeanNameViewResolver"/>	
```
#### 3.3.3、返回void
##### (1)通过ServletAPI传递数据并完成跳转
* 可在方法参数中放入HTTPServletRequest或HTTPSession，使方法中可以直接将数据放入到request、session的域中，也可以通过request.getServletContext()获取到ServletContext，从而将数据放入到application的域中。
* 可在方法参数中放入HTTPServletRequest与HTTPServletSession，使方法可以完成请求转发与重定向。
##### (2)AJAX响应
#### 3.3.4、返回Object
* 处理器方法也可以返回Object对象。但返回的这个Object对象不是作为逻辑视图出现的，而是作为直接在页面显示的数据出现的。
* 返回Object对象，需要使用@ResponseBody注解，将转换后的JSON数据放入到响应体中。
* 将Object数据转化为JSON数据，需要由HTTPMessageConverter完成。而转换器的开启，需要由`<mvc:annontation-driven/>`完成
##### （1）返回数值型对象
```
@RequestMapping(value="/myAjax.do")
	@ResponseBody   //将返回的数据放入响应体中
	public Object doAjax() {
		return "1234";
	}
```
##### （2）返回字符串对象
* 如果有中文，会出现乱码问题只需要在@RequestMapping加上`produces="text/html;charset=utf-8"`即可
```
@RequestMapping(value="/myAjax.do", produces="text/html;charset=utf-8")
```
##### （3）返回自定义类型对象
* 返回自定义类型对象时，不能以对象的形式直接返回给客户端浏览器，而是将对象转换为JSON格式的数据发送给浏览器的。
##### （4）返回Map集合
##### （5）返回List集合
## 4、SpringMVC核心技术
### 4.1、请求转发与重定向
* 当处理器对请求处理完毕后，向其它资源进行跳转时，有两种跳转方式：请求转发与重定向。而根据所要跳转的资源类型，又可分为两类：跳转到页面与跳转到其它处理器。
#### 4.1.1、返回ModelAndView时的请求转发
* 默认情况下，当处理器方法返回ModelAndView时，跳转到指定的View，使用的是请求转发，但也可显示的进行指出。此时，需在setViewName()指定的视图前添加forward，且此时的视图不会再与视图解析器中的前缀与后缀进行拼接。即 **必须写出相对于项目根的路径** 。故此时的视图解析器不再需要前缀与后`mv.setViewName("forward:......")`
### 4.2、异常处理
* 常用的SpringMVC异常处理方式主要有三种：
  * 使用系统定义好的异常处理器SimpleMappingExceptionResolver
  * 使用自定义异常处理器
  * 使用异常处理注解
#### 4.2.1、SimpleMappingExceptionResolver异常处理器
* 该方式只需要在SpringMVC配置文件中注册该异常处理器Bean即可。该Bean比较特殊，没有id属性，无需显示调用或被注入给其它<bean/>，当异常发生时会自动执行该类。
#### 4.2.2、自定义异常处理器HandlerExceptionResolver
* 使用SpringMVC定义好的SimpleMappingExceptionResolver异常处理器，可以实现发生指定异常后的跳转。但若要实现在捕获到异常时，执行一些操作的目的，它是完成不了的。此时，就需要自定义异常处理器。
* 自定义异常处理器，需要实现HandlerExceptionResolver接口，并且该类需要在SpringMVC配置文件中进行注册。
#### 4.2.3、异常处理注解@ExceptionHandler
* 使用注解@ExceptionHandler可以将一个方法指定为异常处理方法。该注解只有一个可选属性value，为一个Class<?>数组，用于指定该注解的方法所要处理的异常类，即所要匹配的异常。
### 4.3、类型转换器
### 4.4、初始化参数绑定
### 4.5、数据验证
### 4.6、文件上传
### 4.7、拦截器





  
  
  
