* Hibernate框架是提供了全面的数据库封装机制的“全自动”ORM（Object Relationship Mapping），即实现了POJO和数据库表之间的映射，以及SQL的自动生成和执行。<br>
* 相对于此，MyBatis只能算作是“半自动”ORM。其着力点，是在POJO类与SQL语句之间的映射关系。也就是说，MyBatis并不会为程序员自动生成SQL语句。具体的SQL需要程序员自己编写，然后通过SQL语句映射文件，将SQL所需的参数，以及返回的结果字段映射到指定POJO。因此，MyBatis成为了“全自动”ORM的一种有益补充。<br>
* **与Hibernate相比，MyBatis 具有以下几个特点：**
  * （1）在XML文件中配置SQL语句，实现了SQL语句与代码的分离，给程序的维护带来了很大便利。<br>
  * （2）因为需要程序员自己去编写SQL语句，程序员可以结合数据库自身的特点灵活控制SQL语句，因此能够实现比Hibernate等全自动ORM框架更高的查询效率。能够完成复杂查询。<br>
  * （3）简单，易于学习，易于使用，上手快。
