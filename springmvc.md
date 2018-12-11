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

#### （4）ResourceBundleViewResolver视图解析器
  
  
  
  
  
