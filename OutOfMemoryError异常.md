![](https://images2015.cnblogs.com/blog/665375/201601/665375-20160126212928129-1855187537.png)<br>
**java堆溢出:** java 堆用来存储对象实例，只要不断创建对象，并且保证GC Roots到对象之间有可达路径来避免垃圾回收机制清除这些对象，那么在对象数量到达最大堆的容量限制后就会产生内存溢出异常。<br>
**虚拟机栈和本地方法栈溢出:** 如果线程请求的栈深度大于虚拟机所允许的醉倒深度，将抛出StackOverflowError异常；如果虚拟机在扩展栈时无法申请到足够的内存空间，则抛出OutOfMemoryError异常。<br>
#### 方法区和运行时常量池溢出:
由于运行时常量池是方法区的一部分，因此这两个区域的溢出测试就放在一起进行。如果要向运行时常量池中添加内容，最简单的做法就是使用String.intern()这个Native方法。该方法的作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回代表池中这个字符串的String对象；否则，将此String对象包含的字符串添加到常量池中，并且返回此String对象的引用。在JDK 1.6及之前的版本中，由于常量池分配在永久代区内，我们可以通过-XX:PermSize和-XX:MaxPermSize限制方法区的大小，从而间接限制其中常量池的容量，如代码清单2-6所示。
```java
/**
 * VM Args：-XX:PermSize=10M -XX:MaxPermSize=10M
 * @author zm
 */
public class RuntimeConstantPoolOOM {
  public static void main(String[] args) {
		// 使用List保持着常量池引用，避免Full GC回收常量池行为
		List<String> list = new ArrayList<String>();
		// 10MB的PermSize在integer范围内足够产生OOM了
		int i = 0;
		while (true) {
			list.add(String.valueOf(i++).intern());
		}
	}
}
```
运行结果：
```java
Exception in thread "main" java.lang.OutOfMemoryError: PermGen space 
  at java.lang.String.intern(Native Method)
  at org.fenixsoft.oom.RuntimeConstantPoolOOM.main(RuntimeConstantPoolOOM.java:18)
```
从运行结果中可以看到，运行时常量池溢出，在OutOfMemoryError后面跟随的提示信息是“PermGen space”，说明运行时常量池属于方法区（HotSpot虚拟机中的永久代）的一部分。而使用JDK 1.7 运行这段程序就不会得到相同的结果，While循环将一直进行下去。
