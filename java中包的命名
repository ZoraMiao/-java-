## 一般分为4-5层
之所以要这样命名项目中的包，是为了保证项目中所用到的类具有全球唯一性<br>
比如：com.bjpowernode.oa.xxx.ooo.jjj.service.impl<br>
第一层：甲方公司域名的倒序  com.bjpowernode<br>
第二层：项目名称     oa<br>
第三层：模块功能     xxx.ooo.jjj<br>
第四层：功能顶层包   service、dao、beans、vo<br>
第五层：实现类层     impl<br>
       有些功能顶层包是没有实现类层的<br>
      
>beans:<br>
entity:<br>
po:Persistance Object,持久化对象<br>
上面这些包中所存放的类，一般具有id属性。因为它们在DB中都有相应的表<br>

>vo:Value Object值对象。一般用于在类与页面传值<br>
dto:Data Transter Object,数据传输对象。一般用于在类间进行传值<br>
下面这些包中所存放的类，一般没有id属性。因为它们不需要持久化到DB，仅仅是用于在代码中进行数据传输。
