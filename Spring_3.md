#  Spring 与DAO
* **对于JDBC模板的使用，是IOC的应用，是将JDBC模板对象注入给了Dao层的实现类**
* **对于Spring的事物管理，是AOP的应用，将事物作为切面织入到了Service层的业务方法中**
## Spring与JDBC模板
* 为了避免直接使用JDBC而带来的复杂且冗长的代码，Spring提供了一个强有力的模板类---JdbcTemplate来简化JDBC操作。并且并且，数据源DataSource对象与模板JdbcTemplate对象均可通过Bean的形式定义在配置文件中，充分发挥了依赖注入的威力。
