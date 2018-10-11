#  4、Spring 与DAO
* **对于JDBC模板的使用，是IOC的应用，是将JDBC模板对象注入给了Dao层的实现类**
* **对于Spring的事物管理，是AOP的应用，将事物作为切面织入到了Service层的业务方法中**
## 4.1、Spring与JDBC模板
* 为了避免直接使用JDBC而带来的复杂且冗长的代码，Spring提供了一个强有力的模板类---JdbcTemplate来简化JDBC操作。并且并且，数据源DataSource对象与模板JdbcTemplate对象均可通过Bean的形式定义在配置文件中，充分发挥了依赖注入的威力。
### 4.1.1、数据源的配置（3种）
* （1）Spring默认的数据源DriverManagerDataSource
* （2）DBCP数据源BasticDataSource
* （3）C3P0数据源ComboPooledDataSource
### 4.1.9、注意JDBC模板对象是多例的
* 即系统会为每一个使用模板对象的线程（方法）创建一个JdbcTemplate实例，并且在该线程（方法）结束时，自动释放JdbcTemplate实例。所以在每次使用JdbcTemplate对象时，都需要通过getJdbcTemplate()方法获取。
## 4.2、Spring的事务管理
* 事务原本是数据库中的概念，在Dao层。但一般情况下，需要将事务提升到业务层，即Service层。这样做是为了能够使用事务的特性来管理具体的业务。
* 在Spring中通常可以通过以下三种方式来实现对事务的管理：
  * （1）使用Spring的事务代理工厂管理事务
  * （2）使用Spring的事务注解管理工厂
  * （3）使用AspectJ的AOP配置管理事务
### 4.2.1、Spring事务管理API
* Spring的事务管理，主要用到两个事务相关的接口。
  * **（1）事务管理器接口**
  * 事务管理器PlatformTransactionManager接口对象。其主要用于完成事务的提交、回滚，及获取事务的状态信息。
  * PlatformTransactionManager接口有两个常用的实现类：
    * DataSourceTransactionManager：使用JDBC或iBatis进行持久化数据时使用。
    * HibernateTransactionManager：使用Hibernate进行持久化数据时使用。
  * **Spring的回滚方式**
    * Spring事务的默认回滚方式是：**发生运行时异常时回滚，发生受查异常时提交**。不过，对于受查异常，程序员也可以手工设置其回滚方式。
  * 错误与异常
    * Throwable：Error 和 Exception
    * Exception：RuntimeException 和 IOException 和 SQLException 和 ... 和 自定义Exception
    * Throwable类是Java语言中所有错误或异常的超类。只有当对象是此类(或其子类之一)的实例时，才能通过Java虚拟机或者Java的throw语句抛出。
    * Error是程序在运行过程中出现的无法处理的错误，比如OutOfMemoryError、ThreadDeath. NoSuchMethodError 等。当这些错误发生时，程序是无法处理(捕获或抛出)的，JVM一般会终止线程。
    * 程序在编译和运行时出现的另一类错误称之为异常，它是JMM通知程序员的一种方式。通过这种方式，让程序员知道已经或可能出现错误，要求程序员对其进行处理。
    * **异常分为运行时异常与受查异常。**
    * 运行时异常，是RuntimeException类或其子类，即只有在运行时才出现的异常。如，NullPointerException、ArrayIndexQutOfBoundsException、 llegalArgumentException 等均属于运行时异常。这些异常由JVM抛出，在编译时不要求必须处理(捕获或抛出)。但，只要代码编写足够仔细，程序足够健壮，运行时异常是可以避免的。
    * **注意，Hibernate 异席HibernateException就属于运行时异常。**
    * 受查异常，也叫编译时异常，即在代码编写时要求必须捕获或抛出的异常，若不处理，则无法通过编译，如SQLException ClassNotFoundException, lOException等都属于受查异常。
    * RuntimeException及其子类以外的异常，均属于受查异常。当然，用户自定义的Exception的子类，即用户自定义的异常也属受查异常。程序员在定义异常时，只要未明确声明定义的为RuntimeException的子类，那么定义的就是受查异常。
* **(2)事务定义接口**
  *  事务定义接口TransactionDefinition中定义了事务描述相关的三类常量:事务隔离级别、事务传播行为、事务默认超时时限，及对它们的操作。
  * **定义了七个事务传播行为常量**
  * 所谓事务传播行为是指，处于不同事务中的方法在相互调用时，执行期间事务的维护情况。如，A事务中的方法doSome(调用B事务中的方法doOther(),在调用执行期间事务的维护情况，就称为事务传播行为。事务传播行为是加在方法上的。
    * 事务传造行为常量都是以PROPAGATION_开头， 形如PROPAGATION x0x.
  * a、REQUIRED:
  * 指定的方法必须在事务内执行。若当前存在事务，就加入到当前事务中:若当前没有事务，则创建一个新事务。这种传播行为是最常见的选择，也是Spring默认的事务传播行为。
  * b、SUPPORTS
  * 指定的方法支持当前事务，但若当前没有事务，也可以以非事务方式执行。
  * c、MANDATORY
  * 指定的方法必须在当前事务内执行，若当前没有事务，则直接抛出异常。
  * d、REQUIRES_NEW
  * 总是新建一个事务，若当前存在事务，就将当前事务挂起，直到新事务执行完毕。
  * e、NOT_SUPPORTED
  * 指定的方法不能在事务环境中执行，若当前存在事务，就将当前事务挂起。
  * f、NEVER
  * 指定的方法不能在事务环境下执行，若当前存在事务，就直接抛出异常。
  * g、NESTED
  * 指定的方法必须在事务内进行，若当前存在事务，则在嵌套事务内执行；若当前没有事务，则创建一个新事务。
* c、 定义了默认事务超时时限
* 常量TMEOUT DEFAULT定义了事务底层默认的超时时限，及不支持事务超时时限设置的none值。
* 注意，事务的超时时限起作用的条件比较多，且超时的时间计算点较复杂。所以，该值-般就使用默认值即可。
# 5、Spring与Mybatis
* 将Spring与Mybatis进行整合，主要解决的问题就是将SqlSessionFactory对象交给Spring来管理。所以，该整合，只需要将SqlSessionFactory的对象生成器SqlSessionFactoryBean注册在Spring容器中，再将其注入给Dao的实现类即可完成整合。两种方式进行整合：
  * （1）Mapper动态代理
  * （2）支持扫描的Mapper动态代理
# 6、Spring与Web
* 创建一个servlet来获取Spring容器的对象，进而获取Service对象，就可以对数据库进行操作了。




    
    
