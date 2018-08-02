# 垃圾收集(GC)
## 判断对象是否存活
* 引用计数算法  <br>
  给对象添加一个引用计数器，每当有一个地方引用他时，计数器值就加1；当引用失效时，计数器值就减1；任何时刻计数器为0的对象就是不可能再被使用，将对其回收。<br>
  但是主流的java虚拟机里面没有选用这种方法来管理内存，主要因为它很难解决对象之间相互循环引用的问题。例如：<br>
  ```java
  public class MyObject {
    public Object ref = null;
    public static void main(String[] args) {
        MyObject myObject1 = new MyObject();
        MyObject myObject2 = new MyObject();
        myObject1.ref = myObject2;
        myObject2.ref = myObject1;
        myObject1 = null;
        myObject2 = null;
    }
  }
  ```
  这两个对象没有任何其他的引用，但是他们之间相互引用着对方，导致它们的计数器都不为0，于是引用计数法无法通知GC收集器回收它们。<br>
* 可达性分析算法<br>
  通过一系列的称为"GC Roots"的对象作为起点，从这些节点开始向下搜索，搜索所走过的路径称为引用链，当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用；是可回收对象。<br>
  在java语言中，可作为GC Roots的对象包括下面几种：<br>
  * 虚拟机栈（栈帧中的本地变量表）中引用的对象
  * 方法区中类静态属性引用的对象
  * 方法区中常量引用的对象
  * 本地方法栈中JNI（即一般说的Native方法）引用的对象
