## 1.ArrayList

**集合和数组的区别 :** 

​	共同点：都是存储数据的容器

​	不同点：数组的容量是固定的，集合的容量是可变的

### 1.1 -ArrayList的构造方法和添加方法

| public ArrayList()                   | 创建一个空的集合对象               |
| ------------------------------------ | ---------------------------------- |
| public boolean add(E e)              | 将指定的元素追加到此集合的末尾     |
| public void add(int index,E element) | 在此集合中的指定位置插入指定的元素 |

**ArrayList<E> ：** 

​	可调整大小的数组实现 

​	<E> : 是一种特殊的数据类型，泛型。

**怎么用呢 ?**	

​	在出现E的地方我们使用引用数据类型替换即可	

​	举例：ArrayList<String>, ArrayList<Student>

### 1.2ArrayList类常用方法【应用】

**成员方法 : **

| public boolean remove(Object o)   | 删除指定的元素，返回删除是否成功       |
| --------------------------------- | -------------------------------------- |
| public E remove(int index)        | 删除指定索引处的元素，返回被删除的元素 |
| public E set(int index,E element) | 修改指定索引处的元素，返回被修改的元素 |
| public E get(int index)           | 返回指定索引处的元素                   |
| public int size()                 | 返回集合中的元素的个数                 |

**示例代码 :** 

```java
public class ArrayListDemo02 {
    public static void main(String[] args) {
        //创建集合
        ArrayList<String> array = new ArrayList<String>();

        //添加元素
        array.add("hello");
        array.add("world");
        array.add("java");

        //public boolean remove(Object o)：删除指定的元素，返回删除是否成功
//        System.out.println(array.remove("world"));
//        System.out.println(array.remove("javaee"));

        //public E remove(int index)：删除指定索引处的元素，返回被删除的元素
//        System.out.println(array.remove(1));

        //IndexOutOfBoundsException
//        System.out.println(array.remove(3));

        //public E set(int index,E element)：修改指定索引处的元素，返回被修改的元素
//        System.out.println(array.set(1,"javaee"));

        //IndexOutOfBoundsException
//        System.out.println(array.set(3,"javaee"));

        //public E get(int index)：返回指定索引处的元素
//        System.out.println(array.get(0));
//        System.out.println(array.get(1));
//        System.out.println(array.get(2));
        //System.out.println(array.get(3)); //？？？？？？ 自己测试

        //public int size()：返回集合中的元素的个数
        System.out.println(array.size());

        //输出集合
        System.out.println("array:" + array);
    }
}
```

### 1.3 ArrayList存储字符串并遍历

**案例需求 :** 

​	创建一个存储字符串的集合，存储3个字符串元素，使用程序实现在控制台遍历该集合

**实现步骤 :** 

 	1:创建集合对象
        2:往集合中添加字符串对象
        3:遍历集合，首先要能够获取到集合中的每一个元素，这个通过get(int index)方法实现
        4:遍历集合，其次要能够获取到集合的长度，这个通过size()方法实现
        5:遍历集合的通用格式

**代码实现 :** 

```java
/*
    思路：
        1:创建集合对象
        2:往集合中添加字符串对象
        3:遍历集合，首先要能够获取到集合中的每一个元素，这个通过get(int index)方法实现
        4:遍历集合，其次要能够获取到集合的长度，这个通过size()方法实现
        5:遍历集合的通用格式
 */
public class ArrayListTest01 {
    public static void main(String[] args) {
        //创建集合对象
        ArrayList<String> array = new ArrayList<String>();

        //往集合中添加字符串对象
        array.add("刘正风");
        array.add("左冷禅");
        array.add("风清扬");

        //遍历集合，其次要能够获取到集合的长度，这个通过size()方法实现
//        System.out.println(array.size());

        //遍历集合的通用格式
        for(int i=0; i<array.size(); i++) {
            String s = array.get(i);
            System.out.println(s);
        }
    }
}
```

### 1.4 ArrayList存储学生对象并遍历

**案例需求 :** 

​	创建一个存储学生对象的集合，存储3个学生对象，使用程序实现在控制台遍历该集合

**实现步骤 : ** 

​	1:定义学生类    

​	2:创建集合对象    

​	3:创建学生对象    

​	4:添加学生对象到集合中    

​	5:遍历集合，采用通用遍历格式实现

**代码实现 :**

```java
/*
    思路：
        1:定义学生类
        2:创建集合对象
        3:创建学生对象
        4:添加学生对象到集合中
        5:遍历集合，采用通用遍历格式实现
 */
public class ArrayListTest02 {
    public static void main(String[] args) {
        //创建集合对象
        ArrayList<Student> array = new ArrayList<>();

        //创建学生对象
        Student s1 = new Student("林青霞", 30);
        Student s2 = new Student("风清扬", 33);
        Student s3 = new Student("张曼玉", 18);

        //添加学生对象到集合中
        array.add(s1);
        array.add(s2);
        array.add(s3);

        //遍历集合，采用通用遍历格式实现
        for (int i = 0; i < array.size(); i++) {
            Student s = array.get(i);
            System.out.println(s.getName() + "," + s.getAge());
        }
    }
}
```

### 1.5 键盘录入学生信息到集合

**案例需求 :** 

​	创建一个存储学生对象的集合，存储3个学生对象，使用程序实现在控制台遍历该集合

​        学生的姓名和年龄来自于键盘录入

**实现步骤 :**

​	1:定义学生类，为了键盘录入数据方便，把学生类中的成员变量都定义为String类型    

​	2:创建集合对象    

​	3:键盘录入学生对象所需要的数据    

​	4:创建学生对象，把键盘录入的数据赋值给学生对象的成员变量    

​	5:往集合中添加学生对象    

​	6:遍历集合，采用通用遍历格式实现

**代码实现 :** 

```java
/*
    思路：
        1:定义学生类，为了键盘录入数据方便，把学生类中的成员变量都定义为String类型
        2:创建集合对象
        3:键盘录入学生对象所需要的数据
        4:创建学生对象，把键盘录入的数据赋值给学生对象的成员变量
        5:往集合中添加学生对象
        6:遍历集合，采用通用遍历格式实现
 */
public class ArrayListTest {
    public static void main(String[] args) {
        //创建集合对象
        ArrayList<Student> array = new ArrayList<Student>();

        //为了提高代码的复用性，我们用方法来改进程序
        addStudent(array);
        addStudent(array);
        addStudent(array);

        //遍历集合，采用通用遍历格式实现
        for (int i = 0; i < array.size(); i++) {
            Student s = array.get(i);
            System.out.println(s.getName() + "," + s.getAge());
        }
    }

    /*
        两个明确：
            返回值类型：void
            参数：ArrayList<Student> array
     */
    public static void addStudent(ArrayList<Student> array) {
        //键盘录入学生对象所需要的数据
        Scanner sc = new Scanner(System.in);

        System.out.println("请输入学生姓名:");
        String name = sc.nextLine();

        System.out.println("请输入学生年龄:");
        String age = sc.nextLine();

        //创建学生对象，把键盘录入的数据赋值给学生对象的成员变量
        Student s = new Student();
        s.setName(name);
        s.setAge(age);

        //往集合中添加学生对象
        array.add(s);
    }
}
```

## 2. 学生管理系统

### 2.1 学生管理系统实现步骤

- **案例需求**

  ​	针对目前我们的所学内容，完成一个综合案例：学生管理系统！该系统主要功能如下：

  ​	添加学生：通过键盘录入学生信息，添加到集合中

  ​	删除学生：通过键盘录入要删除学生的学号，将该学生对象从集合中删除

  ​	修改学生：通过键盘录入要修改学生的学号，将该学生对象其他信息进行修改

  ​	查看学生：将集合中的学生对象信息进行展示

  ​	退出系统：结束程序

- **实现步骤**

  1. 定义学生类，包含以下成员变量

     学生类：	Student成员变量：	

     ​	学号：sid	

     ​	姓名：name	

     ​	年龄：age	

     ​	生日：birthday

     ​	构造方法：	

     ​		无参构造	

     ​		带四个参数的构造成员方法：	

     ​	每个成员变量对应给出get/set方法

  2. 学生管理系统主界面的搭建步骤

     2.1 用输出语句完成主界面的编写

     2.2 用Scanner实现键盘录入数据

     2.3 用switch语句完成操作的选择

     2.4 用循环完成再次回到主界面

  3. 学生管理系统的添加学生功能实现步骤

     3.1 用键盘录入选择添加学生

     3.2 定义一个方法，用于添加学生

     ​	显示提示信息，提示要输入何种信息

     ​	键盘录入学生对象所需要的数据

     ​	创建学生对象，把键盘录入的数据赋值给学生对象的成员变量

     ​	将学生对象添加到集合中（保存）

     ​	给出添加成功提示

     3.3 调用方法

  4. 学生管理系统的查看学生功能实现步骤

     4.1 用键盘录入选择查看所有学生信息

     4.2 定义一个方法，用于查看学生信息

     ​	显示表头信息

     ​	将集合中数据取出按照对应格式显示学生信息，年龄显示补充“岁”

     4.3 调用方法

  5. 学生管理系统的删除学生功能实现步骤

     5.1 用键盘录入选择删除学生信息

     5.2 定义一个方法，用于删除学生信息

     ​	显示提示信息

     ​	键盘录入要删除的学生学号

     ​	调用getIndex方法，查找该学号在集合的索引

     ​	如果索引为-1，提示信息不存在

     ​	如果索引不是-1，调用remove方法删除并提示删除成功

     5.3 调用方法

  6. 学生管理系统的修改学生功能实现步骤

     6.1 用键盘录入选择修改学生信息

     6.2 定义一个方法，用于修改学生信息

     ​	显示提示信息

     ​	键盘录入要修改的学生学号

     ​	调用getIndex方法，查找该学号在集合的索引

     ​	如果索引为-1，提示信息不存在

     ​	如果索引不是-1，键盘录入要修改的学生信息

     ​	集合修改对应的学生信息

     ​	给出修改成功提示

     6.3 调用方法

  7. 退出系统

     使用System.exit(0);退出JVM

### 2.2 学生类的定义

```java
package com.itheima.domain;

public class Student {
    private String sid; // 学号
    private String name; // 姓名
    private int age; // 年龄
    private String birthday; // 生日

    public Student() {
    }

    public Student(String sid, String name, int age, String birthday) {
        this.sid = sid;
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }

    public String getSid() {
        return sid;
    }

    public void setSid(String sid) {
        this.sid = sid;
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

    public String getBirthday() {
        return birthday;
    }

    public void setBirthday(String birthday) {
        this.birthday = birthday;
    }
}
```

### 2.3 测试类的定义

```java
package com.itheima.test;

import com.itheima.domain.Student;

import java.util.ArrayList;
import java.util.Scanner;

public class StudentManager {
    public static void main(String[] args) {

        Scanner sc = new Scanner(System.in);

        // 创建集合容器对象
        ArrayList<Student> list = new ArrayList<>();

        lo:
        while (true) {
            // 1. 搭建主界面菜单
            System.out.println("--------欢迎来到学生管理系统--------");
            System.out.println("1 添加学生");
            System.out.println("2 删除学生");
            System.out.println("3 修改学生");
            System.out.println("4 查看学生");
            System.out.println("5 退出");
            System.out.println("请输入您的选择:");

            String choice = sc.next();

            switch (choice) {
                case "1":
                    //System.out.println("添加学生");
                    addStudent(list);
                    break;
                case "2":
                    //System.out.println("删除学生");
                    deleteStudent(list);
                    break;
                case "3":
                    //System.out.println("修改学生");
                    updateStudent(list);
                    break;
                case "4":
                    // System.out.println("查看学生");
                    queryStudents(list);
                    break;
                case "5":
                    System.out.println("感谢您的使用");
                    break lo;
                default:
                    System.out.println("您的输入有误");
                    break;
            }
        }


    }

    // 修改学生的方法
    public static void updateStudent(ArrayList<Student> list) {
        System.out.println("请输入您要修改的学生学号:");
        Scanner sc = new Scanner(System.in);
        String updateSid = sc.next();
        // 3. 调用getIndex方法, 查找该学号在集合中出现的索引位置
        int index = getIndex(list,updateSid);
        // 4. 根据索引判断, 学号在集合中是否存在
        if(index == -1){
            // 不存在: 给出提示
            System.out.println("查无信息, 请重新输入");
        }else{
            // 存在: 接收新的学生信息
            System.out.println("请输入新的学生姓名:");
            String name = sc.next();
            System.out.println("请输入新的学生年龄:");
            int age = sc.nextInt();
            System.out.println("请输入新的学生生日:");
            String birthday = sc.next();
            // 封装为新的学生对象
            Student stu = new Student(updateSid, name, age, birthday);
            // 调用集合的set方法, 完成修改
            list.set(index, stu);
            System.out.println("修改成功!");
        }
    }

    // 删除学生的方法
    public static void deleteStudent(ArrayList<Student> list) {
        // 1. 给出提示信息 (请输入您要删除的学号)
        System.out.println("请输入您要删除的学生学号:");
        // 2. 键盘接收要删除的学号
        Scanner sc = new Scanner(System.in);
        String deleteSid = sc.next();
        // 3. 调用getIndex方法, 查找该学号在集合中出现的索引位置
        int index = getIndex(list,deleteSid);
        // 4. 根据索引判断, 学号在集合中是否存在
        if(index == -1){
            // 不存在: 给出提示
            System.out.println("查无信息, 请重新输入");
        }else{
            // 存在:删除
            list.remove(index);
            System.out.println("删除成功!");
        }
    }

    // 查看学生的方法
    public static void queryStudents(ArrayList<Student> list) {
        // 1. 判断集合中是否存在数据, 如果不存在直接给出提示
        if(list.size() == 0){
            System.out.println("无信息, 请添加后重新查询");
            return;
        }
        // 2. 存在: 展示表头数据
        System.out.println("学号\t\t姓名\t年龄\t生日");
        // 3. 遍历集合, 获取每一个学生对象的信息, 打印在控制台
        for (int i = 0; i < list.size(); i++) {
            Student stu = list.get(i);
            System.out.println(stu.getSid() + "\t" + stu.getName() + "\t" + stu.getAge() + "\t\t" + stu.getBirthday());
        }
    }

    // 添加学生的方法
    public static void addStudent(ArrayList<Student> list) {
        Scanner sc = new Scanner(System.in);
        // 1. 给出录入的提示信息

        String sid;

        while(true){
            System.out.println("请输入学号:");
            sid = sc.next();

            int index = getIndex(list, sid);

            if(index == -1){
                // sid不存在, 学号可以使用
                break;
            }
        }

        System.out.println("请输入姓名:");
        String name = sc.next();
        System.out.println("请输入年龄:");
        int age = sc.nextInt();
        System.out.println("请输入生日:");
        String birthday = sc.next();
        // 2. 将键盘录入的信息封装为学生对象
        Student stu = new Student(sid,name,age,birthday);
        // 3. 将封装好的学生对象, 添加到集合容器当中
        list.add(stu);
        // 4. 给出添加成功的提示信息
        System.out.println("添加成功!");
    }

    /*
        getIndex : 接收一个集合对象, 接收一个学生学号

        查找这个学号, 在集合中出现的索引位置
     */
    public static int getIndex(ArrayList<Student> list, String sid){
        // 1. 假设传入的学号, 在集合中不存在
        int index = -1;
        // 2. 遍历集合, 获取每一个学生对象, 准备进行查找
        for (int i = 0; i < list.size(); i++) {
            Student stu = list.get(i);
            // 3. 获取每一个学生对象的学号
            String id = stu.getSid();
            // 4. 使用获取出的学生学号, 和传入的学号(查找的学号)进行比对
            if(id.equals(sid)){
                // 存在: 让index变量记录正确的索引位置
                index = i;
            }
        }

        return index;
    }
}
```
