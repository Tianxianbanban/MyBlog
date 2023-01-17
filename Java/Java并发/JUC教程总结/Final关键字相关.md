## Final关键字相关



**final关键字和不变性**

+ 什么是不变性（Immutable）

  + 如果对象**在被创建之后，状态就不能被修改**，那么它就是不可变的。
  + 具有不变性的对象**一定是线程安全**的，我们不需要对它采取任何的额外的安全措施，也能保证线程安全。

+ final的作用

  + 早期：早期的Java实现版本中，会将final方法转换为内嵌调用。（相当于在一个方法内就完成工组，而没有调用这个过程，方法调用也是有性能损耗的，那么也就能有一定的**效率**提高）
  + 现在：**类防止被继承、方法防止被重写、变量防止被修改**；而且天生是**线程安全**的，而不需要额外的同步开销。（而目前由于JVM性能的提高，已经不需要考虑效率问题，更多的还是final带来的清晰语义）

+ 3中用法：修饰变量、方法、类

  + （变量）被final修饰的变量，意味着**值不能被修改**。如果变量是对象，那么对象的**引用不能变**，但是对象自身的内容还是可以变化的。

    **属性**被final声明以后，该变量就**只能被赋值一次**，且一旦被被赋值，就**不能再被改变**。final修饰的三种变量：（1）类中的final属性：直接赋值；构造函数中赋值；初始化块中。（不能不赋值）（2）类中的static final属性：直接赋值；静态初始化块中。（3）方法中的final变量：不要求赋值时机，但是使用前必须赋值。（和方法中普通变量是一样的）

    为什么要规定赋值时机呢？如果初始化不赋值，而后续赋值，那就是从null变成了我们自己的赋值，这本身就**违反了final不变**的原则了。

  + （方法）不允许修饰构造方法。修饰普通方法意味着不能被重写，即使子类有同样名字的方法也不是重写（此时也不允许子类有重名方法），和static方法是一样的道理。引申：静态方法无法被重写，但是可以重名。

  + （类）不可被继承，比如String类。

+ 注意点

  + final修饰对象的时候，只是对象的**引用不可变**，而对象本身的**属性是可以变化的**。
  + 良好编程习惯：如果已经明确知道某个变量不会改变，最好加上final，保证数据不可变，也是开发过程中语义的体现。

+ 不变性和final的关系

  不变性**不意味**着简单用final修饰就是不可变。

  对于**基本数据类型**，的确是被final修饰以后就不可变了。但是对于**对象类型**，需要该对象保证自身被创建以后，状态永远不会变才行。

  那么如果利用final实现对象不可变？把所有属性都声明为final（不完全正确）；一个属性是对象类型的不可变对象的正确示例：对象创建之后状态就不能被修改、所有属性都是final修饰的、对象创建过程中没有发生溢出。

  ```java
  /**
   * 描述：     一个属性是对象，但是整体不可变，其他类无法修改set里面的数据
   */
  public class ImmutableDemo {
  
      private final Set<String> students = new HashSet<>();
  
      public ImmutableDemo() {
          students.add("李小美");
          students.add("王壮");
          students.add("徐福记");
      }
  
      public boolean isStudent(String name) {
          return students.contains(name);
      }
  }
  ```

  **把变量写在线程内部——栈封闭**：

  在方法中新建的局部变量，实际上是存储在每个线程私有的栈空间，而每个栈的栈空间是不能被其他线程访问到的，所以不会有线程安全问题。这就是“栈封闭”技术，是“线程封闭”技术的一种情况。

  ```java
  
  /**
   * 描述：     演示栈封闭的两种情况，基本变量和对象 先演示线程争抢带来错误结果，然后把变量放到方法内，情况就变了
   */
  public class StackConfinement implements Runnable {
  
      int index = 0;
  
      public void inThread() {
          int neverGoOut = 0;
          //这里的synchronized其实也没有意义，会被优化掉。
          synchronized (this) {
              for (int i = 0; i < 10000; i++) {
                  neverGoOut++;
              }
          }
  
          System.out.println("栈内保护的数字是线程安全的：" + neverGoOut);
      }
  
      @Override
      public void run() {
          for (int i = 0; i < 10000; i++) {
              index++;
          }
          inThread();
      }
  
      public static void main(String[] args) throws InterruptedException {
          StackConfinement r1 = new StackConfinement();
          Thread thread1 = new Thread(r1);
          Thread thread2 = new Thread(r1);
          thread1.start();
          thread2.start();
          thread1.join();
          thread2.join();
          System.out.println(r1.index);
      }
  }
  
  /*
  输出样例：
  栈内保护的数字是线程安全的：10000
  栈内保护的数字是线程安全的：10000
  18668
  */
  ```

  

**相关思考**

+ ```java
  public class FinalStringDemo1 {
  
      public static void main(String[] args) {
          String a = "wukong2";
          final String b = "wukong";//相当于C语言的宏替换
          String d = "wukong";//最开始是指向常量池中的wukong
          String c = b + 2;//由于b，可以直接读出wukong2，同时因为a已经是wukong2，没有必要去创建一个新的对象，直接指向a一样的地址
          String e = d + 2;//d没有被final修饰，所以编译时也不会知道d的值，而是运行时才能确定，所以这个e对应的值会在堆上面生成，
          System.out.println((a == c));
          System.out.println((a == e));
      }
  }
  /*
  true
  false
  */
  ```

+ ```java
  public class FinalStringDemo2 {
  
      public static void main(String[] args) {
          String a = "wukong2";
          final String b = getDashixiong();
          String c = b + 2;
          System.out.println(a == c);
  
      }
  
      private static String getDashixiong() {
          return "wukong";
      }
  }
  /*
  false
  */
  ```

  

