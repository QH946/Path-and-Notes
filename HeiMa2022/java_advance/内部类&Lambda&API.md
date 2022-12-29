## 1.内部类 

### 1.1 内部类的基本使用（理解）

- 内部类概念

  - 在一个类中定义一个类。举例：在一个类A的内部定义一个类B，类B就被称为内部类

- 内部类定义格式

  - 格式&举例：

    ```java
    /*
    	格式：
        class 外部类名{
        	修饰符 class 内部类名{
        	
        	}
        }
    */
  
    class Outer {
        public class Inner {
            
        }
    }
    ```

- 内部类的访问特点 

  - 内部类可以直接访问外部类的成员，包括私有
  - 外部类要访问内部类的成员，必须创建对象

- 示例代码：

  ```java
  /*
      内部类访问特点：
          内部类可以直接访问外部类的成员，包括私有
          外部类要访问内部类的成员，必须创建对象
   */
  public class Outer {
      private int num = 10;
      public class Inner {
          public void show() {
              System.out.println(num);
          }
      }
      public void method() {
          Inner i = new Inner();
          i.show();
      }
  }
  ```

### 1.2 成员内部类（理解）

- 成员内部类的定义位置

  - 在类中方法，跟成员变量是一个位置

- 外界创建成员内部类格式

  - 格式：外部类名.内部类名 对象名 = 外部类对象.内部类对象;
  - 举例：Outer.Inner oi = new Outer().new Inner();

- 私有成员内部类

  - 将一个类，设计为内部类的目的，大多数都是不想让外界去访问，所以内部类的定义应该私有化，私有化之后，再提供一个可以让外界调用的方法，方法内部创建内部类对象并调用。

  - 示例代码：

    ```java
    class Outer {
        private int num = 10;
        private class Inner {
            public void show() {
                System.out.println(num);
            }
        }
        public void method() {
            Inner i = new Inner();
            i.show();
        }
    }
    public class InnerDemo {
        public static void main(String[] args) {
    		//Outer.Inner oi = new Outer().new Inner();
    		//oi.show();
            Outer o = new Outer();
            o.method();
        }
    }
    ```

- 静态成员内部类

  + 静态成员内部类访问格式：外部类名.内部类名 对象名 = new 外部类名.内部类名();

  + 静态成员内部类中的静态方法：外部类名.内部类名.方法名();

  + 示例代码

    ```java
    class Outer {
        static class Inner {
            public void show(){
                System.out.println("inner..show");
            }
    
            public static void method(){
                System.out.println("inner..method");
            }
        }
    }
    
    public class Test3Innerclass {
        /*
            静态成员内部类演示
         */
        public static void main(String[] args) {
            // 外部类名.内部类名 对象名 = new 外部类名.内部类名();
            Outer.Inner oi = new Outer.Inner();
            oi.show();
    
            Outer.Inner.method();
        }
    }
    ```

### 1.3 局部内部类（理解）

- 局部内部类定义位置

  - 局部内部类是在方法中定义的类

- 局部内部类方式方式

  - 局部内部类，外界是无法直接使用，需要在方法内部创建对象并使用
  - 该类可以直接访问外部类的成员，也可以访问方法内的局部变量

- 示例代码

  ```java
  class Outer {
      private int num = 10;
      public void method() {
          int num2 = 20;
          class Inner {
              public void show() {
                  System.out.println(num);
                  System.out.println(num2);
              }
          }
          Inner i = new Inner();
          i.show();
      }
  }
  public class OuterDemo {
      public static void main(String[] args) {
          Outer o = new Outer();
          o.method();
      }
  }
  
  ```

### 1.4 匿名内部类（应用）

- 匿名内部类的前提

  - 存在一个类或者接口，这里的类可以是具体类也可以是抽象类

- 匿名内部类的格式

  - 格式：new 类名 ( ) {  重写方法 }    new  接口名 ( ) { 重写方法 }

  - 举例： 

    ```java
    new Inter(){
        @Override
        public void method(){}
    } 
    ```

- 匿名内部类的本质

  - 本质：是一个继承了该类或者实现了该接口的子类匿名对象

- 匿名内部类的细节

  - 匿名内部类可以通过多态的形式接受

    ```java
    Inter i = new Inter(){
      @Override
        public void method(){
            
        }
    }
    ```

- 匿名内部类直接调用方法

  ```java
  interface Inter{
      void method();
  }
  
  class Test{
      public static void main(String[] args){
          new Inter(){
              @Override
              public void method(){
                  System.out.println("我是匿名内部类");
              }
          }.method();	// 直接调用方法
      }
  }
  ```

### 1.5 匿名内部类在开发中的使用（应用）

- 匿名内部类在开发中的使用

  - 当发现某个方法需要，接口或抽象类的子类对象，我们就可以传递一个匿名内部类过去，来简化传统的代码

- 示例代码：

  ```java
  /*
      游泳接口
   */
  interface Swimming {
      void swim();
  }
  
  public class TestSwimming {
      public static void main(String[] args) {
          goSwimming(new Swimming() {
              @Override
              public void swim() {
                  System.out.println("铁汁, 我们去游泳吧");
              }
          });
      }
  
      /**
       * 使用接口的方法
       */
      public static void goSwimming(Swimming swimming){
          /*
              Swimming swim = new Swimming() {
                  @Override
                  public void swim() {
                      System.out.println("铁汁, 我们去游泳吧");
                  }
              }
           */
          swimming.swim();
      }
  }
  ```

## 2.Lambda表达式

### 2.1体验Lambda表达式【理解】

- 代码演示

  ```java
  /*
      游泳接口
   */
  interface Swimming {
      void swim();
  }

  public class TestSwimming {
      public static void main(String[] args) {
          // 通过匿名内部类实现
          goSwimming(new Swimming() {
              @Override
              public void swim() {
                  System.out.println("铁汁, 我们去游泳吧");
              }
          });

          /*  通过Lambda表达式实现
              理解: 对于Lambda表达式, 对匿名内部类进行了优化
           */
          goSwimming(() -> System.out.println("铁汁, 我们去游泳吧"));
      }

      /**
       * 使用接口的方法
       */
      public static void goSwimming(Swimming swimming) {
          swimming.swim();
      }
  }
  ```

- 函数式编程思想概述

  在数学中，函数就是有输入量、输出量的一套计算方案，也就是“拿数据做操作”

  面向对象思想强调“必须通过对象的形式来做事情”

  函数式思想则尽量忽略面向对象的复杂语法：“强调做什么，而不是以什么形式去做”

  而我们要学习的Lambda表达式就是函数式思想的体现

### 2.2Lambda表达式的标准格式【理解】

- 格式：

  ​	(形式参数) -> {代码块}

  - 形式参数：如果有多个参数，参数之间用逗号隔开；如果没有参数，留空即可
  - ->：由英文中画线和大于符号组成，固定写法。代表指向动作
  - 代码块：是我们具体要做的事情，也就是以前我们写的方法体内容

- 组成Lambda表达式的三要素：

  - 形式参数，箭头，代码块

### 2.3Lambda表达式练习1【应用】

- Lambda表达式的使用前提

  - 有一个接口
  - 接口中有且仅有一个抽象方法

- 练习描述

  ​	无参无返回值抽象方法的练习

- 操作步骤

  - 定义一个接口(Eatable)，里面定义一个抽象方法：void eat();
  - 定义一个测试类(EatableDemo)，在测试类中提供两个方法
    - 一个方法是：useEatable(Eatable e)
    - 一个方法是主方法，在主方法中调用useEatable方法

- 示例代码

  ```java
  //接口
  public interface Eatable {
      void eat();
  }
  //实现类
  public class EatableImpl implements Eatable {
      @Override
      public void eat() {
          System.out.println("一天一苹果，医生远离我");
      }
  }
  //测试类
  public class EatableDemo {
      public static void main(String[] args) {
          //在主方法中调用useEatable方法
          Eatable e = new EatableImpl();
          useEatable(e);
  
          //匿名内部类
          useEatable(new Eatable() {
              @Override
              public void eat() {
                  System.out.println("一天一苹果，医生远离我");
              }
          });
  
          //Lambda表达式
          useEatable(() -> {
              System.out.println("一天一苹果，医生远离我");
          });
      }
  
      private static void useEatable(Eatable e) {
          e.eat();
      }
  }
  ```

### 2.4Lambda表达式练习2【应用】

- 练习描述

  有参无返回值抽象方法的练习

- 操作步骤

  - 定义一个接口(Flyable)，里面定义一个抽象方法：void fly(String s);
  - 定义一个测试类(FlyableDemo)，在测试类中提供两个方法
    - 一个方法是：useFlyable(Flyable f)
    - 一个方法是主方法，在主方法中调用useFlyable方法

- 示例代码

  ```java
  public interface Flyable {
      void fly(String s);
  }
  
  public class FlyableDemo {
      public static void main(String[] args) {
          //在主方法中调用useFlyable方法
          //匿名内部类
          useFlyable(new Flyable() {
              @Override
              public void fly(String s) {
                  System.out.println(s);
                  System.out.println("飞机自驾游");
              }
          });
          System.out.println("--------");
  
          //Lambda
          useFlyable((String s) -> {
              System.out.println(s);
              System.out.println("飞机自驾游");
          });
  
      }
  
      private static void useFlyable(Flyable f) {
          f.fly("风和日丽，晴空万里");
      }
  }
  ```

### 2.5Lambda表达式练习3【应用】

- 练习描述

  有参有返回值抽象方法的练习

- 操作步骤

  - 定义一个接口(Addable)，里面定义一个抽象方法：int add(int x,int y);
  - 定义一个测试类(AddableDemo)，在测试类中提供两个方法
    - 一个方法是：useAddable(Addable a)
    - 一个方法是主方法，在主方法中调用useAddable方法

- 示例代码

  ```java
  public interface Addable {
      int add(int x,int y);
  }
  
  public class AddableDemo {
      public static void main(String[] args) {
          //在主方法中调用useAddable方法
          useAddable((int x,int y) -> {
              return x + y;
          });
  
      }
  
      private static void useAddable(Addable a) {
          int sum = a.add(10, 20);
          System.out.println(sum);
      }
  }
  ```

### 2.6Lambda表达式的省略模式【应用】

- 省略的规则

  - 参数类型可以省略。但是有多个参数的情况下，不能只省略一个
  - 如果参数有且仅有一个，那么小括号可以省略
  - 如果代码块的语句只有一条，可以省略大括号和分号，和return关键字

- 代码演示

  ```java
  public interface Addable {
      int add(int x, int y);
  }
  
  public interface Flyable {
      void fly(String s);
  }
  
  public class LambdaDemo {
      public static void main(String[] args) {
  //        useAddable((int x,int y) -> {
  //            return x + y;
  //        });
          //参数的类型可以省略
          useAddable((x, y) -> {
              return x + y;
          });
  
  //        useFlyable((String s) -> {
  //            System.out.println(s);
  //        });
          //如果参数有且仅有一个，那么小括号可以省略
  //        useFlyable(s -> {
  //            System.out.println(s);
  //        });
  
          //如果代码块的语句只有一条，可以省略大括号和分号
          useFlyable(s -> System.out.println(s));
  
          //如果代码块的语句只有一条，可以省略大括号和分号，如果有return，return也要省略掉
          useAddable((x, y) -> x + y);
      }
  
      private static void useFlyable(Flyable f) {
          f.fly("风和日丽，晴空万里");
      }
  
      private static void useAddable(Addable a) {
          int sum = a.add(10, 20);
          System.out.println(sum);
      }
  }
  ```

### 2.7Lambda表达式的使用前提【理解】

- 使用Lambda必须要有接口
- 并且要求接口中有且仅有一个抽象方法

### 2.8Lambda表达式和匿名内部类的区别【理解】

- 所需类型不同
  - 匿名内部类：可以是接口，也可以是抽象类，还可以是具体类
  - Lambda表达式：只能是接口
- 使用限制不同
  - 如果接口中有且仅有一个抽象方法，可以使用Lambda表达式，也可以使用匿名内部类
  - 如果接口中多于一个抽象方法，只能使用匿名内部类，而不能使用Lambda表达式
- 实现原理不同
  - 匿名内部类：编译之后，产生一个单独的.class字节码文件
  - Lambda表达式：编译之后，没有一个单独的.class字节码文件。对应的字节码会在运行的时候动态生成

## 3.API

### 3.1 API概述【理解】

- 什么是API

  ​	API (Application Programming Interface) ：应用程序编程接口

- java中的API

  ​	指的就是 JDK 中提供的各种功能的 Java类，这些类将底层的实现封装了起来，我们不需要关心这些类是如何实现的，只需要学习这些类如何使用即可，我们可以通过帮助文档来学习这些API如何使用。

### 3.2 如何使用API帮助文档【应用】

- 打开帮助文档

![](D:\HOLY\Path-and-Notes\HeiMa2022\java_advance\img\01.png)

- 找到索引选项卡中的输入框

![](D:\HOLY\Path-and-Notes\HeiMa2022\java_advance\img\02.png)

- 在输入框中输入Random

![](D:\HOLY\Path-and-Notes\HeiMa2022\java_advance\img\03.png)

- 看类在哪个包下

![](D:\HOLY\Path-and-Notes\HeiMa2022\java_advance\img\04.png)

- 看类的描述

![](D:\HOLY\Path-and-Notes\HeiMa2022\java_advance\img\05.png)

- 看构造方法

![](D:\HOLY\Path-and-Notes\HeiMa2022\java_advance\img\06.png)

- 看成员方法

![](D:\HOLY\Path-and-Notes\HeiMa2022\java_advance\img\07.png)

## 4.常用API

### 4.1 Math（应用）

- 1、Math类概述

  - Math 包含执行基本数字运算的方法

- 2、Math中方法的调用方式

  - Math类中无构造方法，但内部的方法都是静态的，则可以通过   **类名.进行调用**

- 3、Math类的常用方法

  | 方法名    方法名                               | 说明                                           |
  | ---------------------------------------------- | ---------------------------------------------- |
  | public static int   abs(int a)                 | 返回参数的绝对值                               |
  | public static double ceil(double a)            | 返回大于或等于参数的最小double值，等于一个整数 |
  | public static double floor(double a)           | 返回小于或等于参数的最大double值，等于一个整数 |
  | public   static int round(float a)             | 按照四舍五入返回最接近参数的int                |
  | public static int   max(int a,int b)           | 返回两个int值中的较大值                        |
  | public   static int min(int a,int b)           | 返回两个int值中的较小值                        |
  | public   static double pow (double a,double b) | 返回a的b次幂的值                               |
  | public   static double random()                | 返回值为double的正值，[0.0,1.0)                |

### 4.2 System（应用）

- System类的常用方法 

  | 方法名                                   | 说明                                             |
  | ---------------------------------------- | ------------------------------------------------ |
  | public   static void exit(int status)    | 终止当前运行的   Java   虚拟机，非零表示异常终止 |
  | public   static long currentTimeMillis() | 返回当前时间(以毫秒为单位)                       |

- 示例代码

  - 需求：在控制台输出1-10000，计算这段代码执行了多少毫秒 

  ```java
  public class SystemDemo {
      public static void main(String[] args) {
          // 获取开始的时间节点
          long start = System.currentTimeMillis();
          for (int i = 1; i <= 10000; i++) {
              System.out.println(i);
          }
          // 获取代码运行结束后的时间节点
          long end = System.currentTimeMillis();
          System.out.println("共耗时：" + (end - start) + "毫秒");
      }
  }
  ```

### 4.3 Object类的toString方法（应用）

- Object类概述

  - Object 是类层次结构的根，每个类都可以将 Object 作为超类。所有类都直接或者间接的继承自该类，换句话说，该类所具备的方法，所有类都会有一份

- 查看方法源码的方式

  - 选中方法，按下Ctrl + B

- 重写toString方法的方式

  - 1. Alt + Insert 选择toString
  - 1. 在类的空白区域，右键 -> Generate -> 选择toString

- toString方法的作用：

  - 以良好的格式，更方便的展示对象中的属性值

- 示例代码：

  ```java
  class Student extends Object {
      private String name;
      private int age;
  
      public Student() {
      }
  
      public Student(String name, int age) {
          this.name = name;
          this.age = age;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public int getAge() {
          return age;
      }
  
      public void setAge(int age) {
          this.age = age;
      }
  
      @Override
      public String toString() {
          return "Student{" +
                  "name='" + name + '\'' +
                  ", age=" + age +
                  '}';
      }
  }
  public class ObjectDemo {
      public static void main(String[] args) {
          Student s = new Student();
          s.setName("林青霞");
          s.setAge(30);
          System.out.println(s); 
          System.out.println(s.toString()); 
      }
  }
  ```

- 运行结果：

  ```java
  Student{name='林青霞', age=30}
  Student{name='林青霞', age=30}
  ```

### 4.4 Object类的equals方法（应用）

- equals方法的作用

  - 用于对象之间的比较，返回true和false的结果
  - 举例：s1.equals(s2);    s1和s2是两个对象

- 重写equals方法的场景

  - 不希望比较对象的地址值，想要结合对象属性进行比较的时候。

- 重写equals方法的方式

  - 1. alt + insert  选择equals() and hashCode()，IntelliJ Default，一路next，finish即可
  - 1. 在类的空白区域，右键 -> Generate -> 选择equals() and hashCode()，后面的同上。

- 示例代码：

  ```java
  class Student {
      private String name;
      private int age;
  
      public Student() {
      }
  
      public Student(String name, int age) {
          this.name = name;
          this.age = age;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public int getAge() {
          return age;
      }
  
      public void setAge(int age) {
          this.age = age;
      }
  
      @Override
      public boolean equals(Object o) {
          //this -- s1
          //o -- s2
          if (this == o) return true;
          if (o == null || getClass() != o.getClass()) return false;
  
          Student student = (Student) o; //student -- s2
  
          if (age != student.age) return false;
          return name != null ? name.equals(student.name) : student.name == null;
      }
  }
  public class ObjectDemo {
      public static void main(String[] args) {
          Student s1 = new Student();
          s1.setName("林青霞");
          s1.setAge(30);
  
          Student s2 = new Student();
          s2.setName("林青霞");
          s2.setAge(30);
  
          //需求：比较两个对象的内容是否相同
          System.out.println(s1.equals(s2));
      }
  }
  
  ```

- 面试题

  ```java
  // 看程序,分析结果
  String s = “abc”;
  StringBuilder sb = new StringBuilder(“abc”);
  s.equals(sb); 
  sb.equals(s); 
  
  public class InterviewTest {
      public static void main(String[] args) {
          String s1 = "abc";
          StringBuilder sb = new StringBuilder("abc");
          //1.此时调用的是String类中的equals方法.
          //保证参数也是字符串,否则不会比较属性值而直接返回false
          //System.out.println(s1.equals(sb)); // false
  
          //StringBuilder类中是没有重写equals方法,用的就是Object类中的.
          System.out.println(sb.equals(s1)); // false
      }
  }
  ```

### 4.5 Objects (应用)

- 常用方法

  | 方法名                                          | 说明                             |
  | ----------------------------------------------- | -------------------------------- |
  | public static String toString(对象)             | 返回参数中对象的字符串表示形式。 |
  | public static String toString(对象, 默认字符串) | 返回对象的字符串表示形式。       |
  | public static Boolean isNull(对象)              | 判断对象是否为空                 |
  | public static Boolean nonNull(对象)             | 判断对象是否不为空               |

- 示例代码

  学生类

  ```java
  class Student {
        private String name;
        private int age;
  
        public Student() {
        }
  
        public Student(String name, int age) {
            this.name = name;
            this.age = age;
        }
  
        public String getName() {
            return name;
        }
  
        public void setName(String name) {
            this.name = name;
        }
  
        public int getAge() {
            return age;
        }
  
        public void setAge(int age) {
            this.age = age;
        }
  
        @Override
        public String toString() {
            return "Student{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
  ```

  测试类

  ```java
  public class MyObjectsDemo {
            public static void main(String[] args) {
        //        public static String toString(对象): 返回参数中对象的字符串表示形式。
        //        Student s = new Student("小罗同学",50);
        //        String result = Objects.toString(s);
        //        System.out.println(result);
        //        System.out.println(s);
  
        //        public static String toString(对象, 默认字符串): 返回对象的字符串表示形式。如果对象为空,那么返回第二个参数.
                //Student s = new Student("小花同学",23);
        //        Student s = null;
        //        String result = Objects.toString(s, "随便写一个");
        //        System.out.println(result);
        
        //        public static Boolean isNull(对象): 判断对象是否为空
                //Student s = null;
        //        Student s = new Student();
        //        boolean result = Objects.isNull(s);
        //        System.out.println(result);
  
        //        public static Boolean nonNull(对象): 判断对象是否不为空
                //Student s = new Student();
                Student s = null;
                boolean result = Objects.nonNull(s);
                System.out.println(result);
            }
    }
  ```

### 4.6 BigDecimal (应用)

- 作用

  可以用来进行精确计算

- 构造方法

  | 方法名                 | 说明         |
  | ---------------------- | ------------ |
  | BigDecimal(double val) | 参数为double |
  | BigDecimal(String val) | 参数为String |

- 常用方法

  | 方法名                                                       | 说明 |
  | ------------------------------------------------------------ | ---- |
  | public BigDecimal add(另一个BigDecimal对象)                  | 加法 |
  | public BigDecimal subtract (另一个BigDecimal对象)            | 减法 |
  | public BigDecimal multiply (另一个BigDecimal对象)            | 乘法 |
  | public BigDecimal divide (另一个BigDecimal对象)              | 除法 |
  | public BigDecimal divide (另一个BigDecimal对象，精确几位，舍入模式) | 除法 |

- 总结

  1. BigDecimal是用来进行精确计算的
  2. 创建BigDecimal的对象，构造方法使用参数类型为字符串的。
  3. 四则运算中的除法，如果除不尽请使用divide的三个参数的方法。

  代码示例：

  ```java
  BigDecimal divide = bd1.divide(参与运算的对象,小数点后精确到多少位,舍入模式);
  参数1 ，表示参与运算的BigDecimal 对象。
  参数2 ，表示小数点后面精确到多少位
  参数3 ，舍入模式  
    BigDecimal.ROUND_UP  进一法
    BigDecimal.ROUND_FLOOR 去尾法
    BigDecimal.ROUND_HALF_UP 四舍五入
  ```



