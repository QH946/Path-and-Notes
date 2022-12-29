# Java9、10、11、12、13、14、15、16、17、18个版本新特性



# Java9新特性

## 1 模块化系统

模块(module)的概念，其实就是package外再裹一层，也就是说，用模块来管理各个package，通过声明某个package暴露，不声明默认就是隐藏。因此，模块化使得代码组织上更安全，因为它可以指定哪些部分可以暴露，哪些部分隐藏。

### 导出模块

被引用模块需要导出指定的文件夹，并且在[根目录](https://so.csdn.net/so/search?q=根目录&spm=1001.2101.3001.7020)下定义 **module-info.java** 文件，编写需要导出的文件包全路径名。

```java
module modulea {
    exports com.lz.java9.bean2;
    exports com.lz.java9.bean;
}
1234
```

### 引用模块

在引用项目的根目录下新建一个 **module-info.java** 文件，如果不新建module-info文件，模块化不生效，可以调用包中的所有类，编写需要引入的模块名称。

```java
module moduleb {
    requires modulea;
}
123
```

## 2 jshell

JShell的目标是提供一个交互工具，通过它来运行和计算java中的表达式。开发者可以轻松地与JShell交互，其中包括：编辑历史，tab键代码补全，自动添加分号，可配置的imports和definitions。其他的很多主流编程语言如python都已经提供了console，便于编写一些简单的代码用于测试。

### demo

```java
jshell> for(int i=0; i<10; i++){
   ...> System.out.println(i);
   ...> }
0
1
2
3
4
5
6
7
8
9
12345678910111213
```

## 3 多版本兼容jar包

多版本兼容 JAR 功能能让你创建仅在特定版本的 Java 环境中运行库程序时选择使用的 class 版本。通过 --release 参数指定编译版本。具体的变化就是 META-INF 目录下 MANIFEST.MF 文件新增了一个属性：

```properties
Multi-Release: true
1
```

然后 META-INF 目录下还新增了一个 versions 目录，如果是要支持 java9，则在 versions 目录下有 9 的目录。

```
multirelease.jar
├── META-INF
│   └── versions
│       └── 9
│           └── multirelease
│               └── Helper.class
├── multirelease
    ├── Helper.class
    └── Main.class
123456789
```

### 实例

使用多版本兼容 JAR 功能将 Tester.java 文件生成了两个版本的 jar 包, 一个是 jdk 7，另一个是 jdk 9，然后我们再不同环境下执行。

#### java8

```java
public class Tester {
   public static void main(String[] args) {
      System.out.println("Inside java 8");
   }
}
12345
```

#### java9

```java
public class Tester {
   public static void main(String[] args) {
      System.out.println("Inside java 9");
   }
}
12345
```

#### 编译两个文件

```shell
javac --release 9 Tester.java
javac --release 8 Tester.java
12
```

#### 创建多版本兼容 jar 包

```shell
jar -c -f test.jar -C java7 . --release 9 -C java9.
1
```

在不同的jdk环境下执行打印不同的结果

## 4 接口中声明私有方法

## 5 钻石操作符使用升级

## 6 异常处理try-with-resources改进

### 9之前

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.Reader;
import java.io.StringReader;

public class Tester {
   public static void main(String[] args) throws IOException {
      System.out.println(readData("test"));
   }
   static String readData(String message) throws IOException {
      Reader inputString = new StringReader(message);
      BufferedReader br = new BufferedReader(inputString);
      try (BufferedReader br1 = br) {
         return br1.readLine();
      }
   }
}
1234567891011121314151617
```

### 9之后

在 Java 9 中，我们不需要声明资源 br1 就可以使用它，并得到相同的结果。

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.Reader;
import java.io.StringReader;

public class Tester {
   public static void main(String[] args) throws IOException {
      System.out.println(readData("test"));
   }
   static String readData(String message) throws IOException {
      Reader inputString = new StringReader(message);
      BufferedReader br = new BufferedReader(inputString);
      try (br) {
         return br.readLine();
      }
   }
}
1234567891011121314151617
```

## 7 _ 下划线命名标识符不能单独使用

## 8 String 底层存储结构发生变化使用字节[数组](https://so.csdn.net/so/search?q=数组&spm=1001.2101.3001.7020)

## 创建不可变集合

```java
    List<String> list = List.of("1", "2", "q");
    Set<String> set = Set.of("aa", "bb", "cc", "dd");
    Map<String, Integer> map = Map.of("aa", 1, "bb", 2);
123
```

## 9 增强Stream API

### takeWhile 从Stream中依次获取满足条件的元素，直匹配到一个不满足条件为止结束获取，不同与filter

```java
    List<Integer> list = List.of(11,33,44,102,232,454,67,556,46,78);
    list.stream().takeWhile(x -> x < 100).forEach(System.out::println);
// 输出结果如下
11
33
44
123456
```

### dropWhile 从Stream中依次删除满足条件的元素，直到匹配到一个不满足条件为止结束删除

```java
    List<Integer> list = List.of(11,33,44,102,232,454,67,556,46,78);
    list.stream().dropWhile(x -> x < 100).forEach(System.out::println);
// 输出结果如下
102
232
454
67
556
46
78
12345678910
```

### Stream.ofNullable(null) 允许单一的null元素

### Stream.iterate重载方法

```java
    Stream.iterate(0, k -> k + 1).limit(12).forEach(System.out::println);
    // 9 新增的重载方法
    Stream.iterate(0, k -> k < 12, k -> k + 1).forEach(System.out::println);
123
```

## 10 Optional新增元素转化为stream()方法等

```java
Optional.of(List.of(1, 23, 4, 5, 6)).stream().forEach(System.out::println);
1
```

## 11 多分辨率图像API

## 12 全新的HTTP客户端API（Java11才正式可用进行了修改和包名改变）

```java
    HttpClient httpClient = HttpClient.newHttpClient();
    HttpRequest httpRequest = HttpRequest
                    .newBuilder(new URI("https://www.baidu.com"))
                    .GET()
                    .build();
    HttpResponse<String> response = httpClient.send(httpRequest, HttpResponse.BodyHandlers.ofString());
    System.out.println(response.statusCode());
    System.out.println(response.body());
12345678
```

## 12 InputStream 增加transferTo方法，将数据直接传输到输出流中

## 13 统一的JVM日志

# Java10新特性

## 1、局部变量的类型推断 var关键字

```java
    var list = new ArrayList<String>();
    list.add("hello，world！");
    System.out.println(list);
123
```

## 2、GC改进和内存管理 并行全垃圾回收器 G1

## 3、新增API：ByteArrayOutputStream

## 4、新增API：List、Map、Set

## 5、新增API：java.util.Properties

## 6、新增API： Collectors收集器

## 7、Optional 新增orElseThrow方法

## 8、基于Java的实验性JIT编译器

Java 10 开启了 Java JIT编译器 Graal，用作Linux / x64平台上的实验性JIT编译器。

其它特性

# Java11新特性

## 1、字符串新增方法

```java
    String str = " qq eeee  ";
    System.out.println(str.isBlank());// 是否为空
    System.out.println(str.strip());// 去除开头结尾空格
    System.out.println(str.stripLeading());// 去除头部空格
    System.out.println(str.stripTrailing());// 去除尾部空格
    System.out.println(str.repeat(3));// 复制三次
    System.out.println(str.lines().count());// 行数操作统计
1234567
```

## 2、Optional isEmpty方法

## 3、局部变量类型推断升级

```java
   Consumer<String> con = (@Deprecated var c) -> System.out.println(c.toLowerCase());
1
```

## 4、确认HttpClient使用，标准化并修改优化

## 5、java命令直接运行源文件，不需要事先javac编译

## 6、引入实验性的 ZGC

# Java12新特性

## 1、更简洁的 switch 语法（预览功能）

在之前的 JAVA 版本中，`switch` 语法还是比较啰嗦的，如果多个值走一个逻辑需要写多个 `case`：

```java
DayOfWeek dayOfWeek = LocalDate.now().getDayOfWeek();
String typeOfDay = "";
switch (dayOfWeek) {
    case MONDAY:
    case TUESDAY:
    case WEDNESDAY:
    case THURSDAY:
    case FRIDAY:
        typeOfDay = "Working Day";
        break;
    case SATURDAY:
    case SUNDAY:
        typeOfDay = "Day Off";
}
1234567891011121314
```

Java12

```java
typeOfDay = switch (dayOfWeek) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Working Day";
    case SATURDAY, SUNDAY -> "Day Off";
};

//特定场景的计算逻辑
int day = 1;
int result = switch (day) {
    case 1, 2, 3, 4, 5 -> 1;
    case 6, 7 -> 2;
    default -> 0;
};
System.out.println(result);
12345678910111213
```

## 2、核心库java.lang中支持Unicode11

1、684个新角色 1.1、66个表情符号字符 1.2、Copyleft符号 1.3、评级系统的半星 1.4、额外的占星符号 1.5、象棋中国象棋符号 2、11个新区块 2.1、格鲁吉亚扩展 2.2、玛雅数字 2.3、印度Siyaq数字 2.4、国际象棋符号 3、7个新脚本 3.1、Hanifi Rohingya 3.2、Old Sogdian 3.3、Sogdian 3.4、Dogra 3.5、Gunjala Gondi 3.6、Makasar 3.7、Medefaidrin

## 3、核心库java.text支持压缩数字格式

```java
System.out.println(NumberFormat.getCompactNumberInstance().format(10000000));
// 输出结果：1000万
12
```

## 4、Collectors.teeing 新增方法

Collectors.teeing： 将 downstream1 和 downstream2 的流入合并，然后从 merger 流出

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class SwitchDemo {
    public static void main(String[] args) {
        CountSum countsum = Stream.of(2, 11, 1, 5, 7, 8, 12)
                .collect(Collectors.teeing(
                        Collectors.counting(),
                        Collectors.summingInt(e -> e),
                        CountSum::new));

        System.out.println(countsum.toString());
    }
}
class CountSum {
    private final Long count;
    private final Integer sum;
    public CountSum(Long count, Integer sum) {
        this.count = count;
        this.sum = sum;
    }

    @Override
    public String toString() {
        return "CountSum{" +
                "count=" + count +
                ", sum=" + sum +
                '}';
    }
}
123456789101112131415161718192021222324252627282930
```

## 5、安全库javax.net.ssl

ChaCha20和Poly1305 TLS密码

JSSE中添加了使用ChaCha20-Poly1305算法的新TLS密码套件。默认情况下启用这些密码套件。TLS_CHACHA20_POLY1305_SHA256密码套件适用于TLS 1.3。

## 6、移除项

核心库/ java.util.jar中，删除java.util.ZipFile / Inflator / Deflator中的finalize方法

核心库/ java.util.jar中，删除java.util.ZipFile / Inflator / Deflator中的finalize方法

工具/ javac的删除javac支持6 / 1.6源，目标和发布值

## 7、JVM常量API

引入API来模拟关键类文件和运行时工件的名义描述，特别是可从常量池加载的常量。

在新的 java.lang.invoke.constant 包中定义了一系列基于值的符号引用（JVMS 5.1）类型，它们能够描述每种可加载常量。

符号引用以纯 nominal 形式描述可加载常量，与类加载或可访问性上下文区分开。有些类可以作为自己的符号引用（例如 String），而对于可链接常量，定义了一系列符号引用类型（ClassDesc、MethodTypeDesc、MethodHandleDesc 和 DynamicConstantDesc），它们包含描述这些常量的 nominal 信息。

## 8、只保留一个 AArch64 实现(源码)

在保留 32 位 ARM 实现和 64 位 aarch64 实现的同时，删除与 arm64 实现相关的所有源码。

JDK 中存在两套 64 位 ARM 实现，主要存在于 src/hotspot/cpu/arm 和 open/src/hotspot/cpu/aarch64 目录。两者都实现了 aarch64，现在将只保留后者，删除由 Oracle 提供的 arm64。这将使贡献者将他们的精力集中在单个 64 位 ARM 实现上，并消除维护两套实现所需的重复工作。

## 9、默认CDS档案

针对 64 位平台，使用默认类列表增强 JDK 构建过程，以生成类数据共享（class data-sharing，CDS）归档。

## 10、G1的可流动混合收集

如果G1混合集合可能超过暂停目标，则使其可以中止。

## 11、G1 及时返回未使用的已分配内存

增强 G1 GC，以便在空闲时自动将 Java 堆内存返回给操作系统。

为了实现向操作系统返回最大内存量的目标，G1 将在应用程序不活动期间定期执行或触发并发周期以确定整体 Java 堆使用情况。这将导致它自动将 Java 堆的未使用部分返回给操作系统。而在用户控制下，可以可选地执行完整的 GC，以使返回的内存量最大化。

## 12、JDK12之Shenandoah低暂停时间垃圾收集器（实验性）

 添加一个名为Shenandoah的新垃圾收集（GC）算法，通过与正在运行的Java线程同时进行疏散工作来减少GC暂停时间。使用Shenandoah的暂停时间与堆大小无关， 这意味着无论堆是200MB还是200GB，您都将具有相同的一致暂停时Shenandoah是适用于评估响应性和可预测的短暂停顿 的应用程序的算法。目标不是解决所有JVM暂停问题。由于GC之外的其他原因（例如安全时间点（TTSP）发布或监控通胀）而暂停时间超出了此JEP的范围。

## 13、JDK12之Microbenchmark Suite

在JDK源代码中添加一套基本的微基准测试，使开发人员可以轻松运行现有的微基准测试并创建新的基准测试。

# Java13新特性

## 1、switch 语法再增强

JAVA 12 中虽然增强了 `swtich` 语法，但并不能在 `->` 之后写复杂的逻辑，JAVA 12 带来了 `swtich`更完美的体验，就像 `lambda`一样，可以写逻辑，然后再返回

```java
typeOfDay = switch (dayOfWeek) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> {
        // do sth...
        yield "Working Day";
    }
    case SATURDAY, SUNDAY -> "Day Off";
};
1234567
```

## 2、多行文本块（预览）

13之前

```java
 String html = "<html>\n" +
                "    <body>\n" +
                "        <p>Hello, world</p>\n" +
                "    </body>\n" +
                "</html>\n";
12345
```

13

```java
String html = """
              <html>
                  <body>
                      <p>Hello, world</p>
                  </body>
              </html>
              """;
1234567
```

## 3、重新实现传统套接字API

使用更简单，更现代的实现替换java.net.Socket和java.net.ServerSocketAPI使用的底层实现，
易于维护和调试。

> 新的实现旨在易于适应使用用户模式线程（也称为光纤），这些线程正在Project Loom中进行探索。上述传统API可以追溯到JDK 1.0，并且包含传统C和Java代码的混合，这些代码被描述为调试和维护的痛苦。遗留实现还存在其他问题：支持异步关闭，导致可靠性和移植问题的本机数据结构，以及需要彻底检查的并发问题。

## 4、API增加新方法

### FileSystems.newFileSystem新方法

核心库/ java.nio中添加了FileSystems.newFileSystem（Path，Map <String，？>）方法，添加了三种新方法`java.nio.file.FileSystems`，以便更轻松地使用将文件内容视为文件系统的文件系统提供程序。

### nio新方法

核心库/ java.nio中新的java.nio.ByteBuffer批量获取/放置方法转移字节而不考虑缓冲区位置。

### 核心库/ java.util中：I18N

支持Unicode 12.1，此版本将Unicode支持升级到12.1，其中包括以下内容：

> java.lang.Character支持12.1级的Unicode字符数据库，其中12.0从11.0开始增加554个字符，
> 总共137,928个字符。这些新增内容包括4个新脚本，总共150个脚本，以及61个新的表情符号字符。
> U+32FF SQUARE ERA NAME REIWA从12.0开始，12.1只添加一个字符。java.text.Bidi和
> java.text.Normalizer类分别支持12.0级的Unicode标准附件，＃9和＃15。
> java.util.regexpackage支持基于12.0级Unicode标准附件＃29的扩展字形集群。

## 5、增强ZGC

增强ZGC以将未使用的堆内存返回给操作系统。

> ZGC目前没有取消提交并将内存返回给操作系统，即使该内存长时间未使用。对于所有类型的应用
> 程序和环境，此行为并非最佳，尤其是那些需要关注内存占用的应用程序和环境 例如：通过使用支付资
> 源的容器环境。应用程序可能长时间处于空闲状态并与许多其他应用程序共享或竞争资源的环境。应用
> 程序在执行期间可能具有非常不同的堆空间要求。
> 例如，启动期间所需的堆可能大于稳态执行期间稍后所需的堆。HotSpot中的其他垃圾收集器，如
> G1和Shenandoah，提供了这种功能，某些类别的用户发现它非常有用。将此功能添加到ZGC将受到同一
> 组用户的欢迎。

# Java14新特性

## 1、Switch（最终版）

和之前的jdk12、13功能一样，只不过确定下来为最终版

## 2、Record（预览功能）

使用它可以替代构造器、equal方法、toString方法，hashCode方法

```java
public record Point(int x, int y) {
}

 public static void main(String[] args) {
        System.out.println(new Point(1,2));
 }
123456
```

由于用record声明的类，已经继承了java.lang.Record 故，record 不能在显示的继承其它类。

> 为了对record 的支持，java.lang.Class引入了*isRecord( )和getRecordComponents( )方法。其中isRecord( )*为判断一个类是否用record进行声明的，*getRecordComponents( )*获取record 中的属性数组RecordComponent[ ]。该数组中包含属性的类型，和属性的值。

## 3、instanceof的模式匹配（预览版）

```java
Object o = new String("test");
if (o instanceof String str) {
    System.out.println(str);
} else{
    // System.out.println(str); 报错
}
123456
```

## 4、NullPointerExceptions 错误栈

Java14之前，NEP报错信息不会指出为Null的实例具体是那一个。例如：a.b.c.d 出现NEP时，开发者无法确定究竟是*a、b、c、d*中的那个变量报了空指针。而在这个新特点的加入之后，NEP错误栈则会明确表明，触发NEP的对象是哪个。

```java
    public static void main(String[] args) {
        A a = new A();
        a.b.toString();
    }

    static class A {
        B b;
    }

    static class B {

    }

// 运行结果明确指出 a.b 是null
Exception in thread "main" java.lang.NullPointerException: Cannot invoke "Object.toString()" because "a.b" is null
        at com.lz.java12.Test.main(Test.java:6)

// 改动再次运行
    A a = null;
    a.b.toString();
// 明确指出 a 是 null
Exception in thread "main" java.lang.NullPointerException: Cannot read field "b" because "a" is null
        at com.lz.java12.Test.main(Test.java:6)
    
123456789101112131415161718192021222324
```

## 5、打包工具 (Incubator)

jpackage打包工具可以将Java应用程序打包为针对特定平台的安装包，这个安装包包含所有必需的依赖项。该应用程序可以以普通JAR文件集合或模块集合的方式提供。软件包格式可以分为：

- Linux：deb和rpm
- macOS：pkg和dmg
- Windows：msi和exe

## 6、垃圾回收器（更新优化）

- 移除 CMS（Concurrent Mark Sweep）垃圾收集器
- ZGC优化window和mac
- 弃用 ParallelScavenge + SerialOld GC 组合

## 7、外部存储器访问 API（孵化）

目的是引入一个 API，以允许 Java 程序安全、有效地访问 Java 堆之外的外部存储器。如本机、持久和托管堆。

# Java15新特性

## 1、Edwards-Curve 数字签名算法

Java15引入了Edwards-Curve(EdDSA)数据签名算法,EdDAS具有更好的性能和更高的安全性。

```java
// example: generate a key pair and sign
KeyPairGenerator kpg = KeyPairGenerator.getInstance("Ed25519");
KeyPair kp = kpg.generateKeyPair();
// algorithm is pure Ed25519
Signature sig = Signature.getInstance("Ed25519");
sig.initSign(kp.getPrivate());
sig.update(msg);
byte[] s = sig.sign();

// example: use KeyFactory to contruct a public key
KeyFactory kf = KeyFactory.getInstance("EdDSA");
boolean xOdd = ...
BigInteger y = ...
NamedParameterSpec paramSpec = new NamedParameterSpec("Ed25519");
EdECPublicKeySpec pubSpec = new EdECPublicKeySpec(paramSpec, new EdPoint(xOdd, y));
PublicKey pubKey = kf.generatePublic(pubSpec);
12345678910111213141516
```

## 2、封闭类（预览）

封闭类和接口主要限制能被哪些类扩展或实现，增加开发人员对定义的类或接口进行管控，实现该类需要得到定义者允许。

密闭类修饰符 `sealed`

指定可扩展修饰符 `permits`

```java
public abstract sealed class Shape
    permits Circle, Rectangle, Square {...}
12
```

密闭类 Shape，定义了三个允许的子类。

预览版作为了解，更多特性见：[JEP 360: Sealed Classes (Preview) (java.net)](https://openjdk.java.net/jeps/360)

## 3、隐藏类

标准 API 来定义**无法发现且具有有限生命周期**的隐藏类，从而提高 JVM 上所有语言的效率。JDK内部和外部的框架将能够动态生成类，而这些类可以定义隐藏类。通常来说基于JVM的很多语言都有动态生成类的机制，这样可以提高语言的灵活性和效率。

- 隐藏类天生为框架设计的，在运行时生成内部的class。
- 隐藏类只能通过反射访问，不能直接被其他类的字节码访问。
- 隐藏类可以独立于其他类加载、卸载，这可以减少框架的内存占用。

不能直接被其他class的二进制代码使用的class。隐藏类主要被一些框架用来生成运行时类，但是这些类不是被用来直接使用的，而是通过反射机制来调用。

比如在JDK8中引入的lambda表达式，JVM并不会在编译的时候将lambda表达式转换成为专门的类，而是在运行时将相应的字节码动态生成相应的类对象。

- java.lang.reflect.Proxy可以定义隐藏类作为实现代理接口的代理类。
- java.lang.invoke.StringConcatFactory可以生成隐藏类来保存常量连接方法；
- java.lang.invoke.LambdaMetaFactory可以生成隐藏的nestmate类，以容纳访问封闭变量的lambda主体；

## 3、禁用、弃用偏向锁

## 4、文本块(正式版)

自Java13引入了文本块 解决多行的问题，Java14 优化 增加\和\s的符号区分 在Java15中 对文本框改进如下

1. stripIndent()用于从文本块去除空白字符
2. translateEscapes() 用于翻译转义字符
3. formatted()用于格式化

## 5、ZGC正式

- ZGC是Java 11引入的新的垃圾收集器（JDK9以后默认的垃圾回收器是G1），经过了多个实验阶段，自此终于成为正式特性。
- 自 2018 年以来，ZGC 已增加了许多改进，从并发类卸载、取消使用未使用的内存、对类数据共享的支持到改进的 NUMA 感知。此外，最大堆大小从 4 TB 增加到 16 TB。支持的平台包括 Linux、Windows 和 MacOS。
- ZGC是一个重新设计的并发的垃圾回收器，通过减少 GC 停顿时间来提高性能。
- 默认的GC仍然还是G1；之前需要通过-XX:+UnlockExperimentalVMOptions -XX:+UseZGC来启用ZGC，现在只需要-XX:+UseZGC就可以。

## 6、Shenandoah GC正式

一种低停顿的垃圾回收器，-XX:+UseShenandoahGC 开启 不需要添加参数 -XX:+UnlockExperimentalVMOptions

## 7、外部存储器访问 API（二次孵化）

Foreign-Memory Access API在JDK14被作为incubating API引入，在JDK15处于Second Incubator，提供了改进。

## 8、重新实现 DatagramSocket API

JEP 373:Reimplement the Legacy DatagramSocket API(重新实现 DatagramSocket API)

新的计划是JEP 353的后续，该方案重新实现了遗留的套接字API。

更改java.net.DatagramSocket 和 java.net.MulticastSocket 为更加简单、现代化的底层实现。提高了 JDK 的可维护性和稳定性。

通过将java.net.datagram.Socket和java.net.MulticastSocket API的底层实现替换为更简单、更现代的实现来重新实现遗留的DatagramSocket API。

## 9、移除废弃

### 1、移除Solaris 和 SPARC 端口

删除对Solaris/SPARC、Solaris/x64和Linux/SPARC端口的源代码和构建支持，在JDK 14中被标记为废弃，在JDK15版本正式移除。

### 2、移除the Nashorn JS引擎

Nashorn是在JDK提出的脚本执行引擎，该功能是 2014 年 3 月发布的 JDK 8 的新特性。在JDK11就已经把它标记为废弃了，JDK15完全移除。

在JDK11中取以代之的是GraalVM。GraalVM是一个运行时平台，它支持Java和其他基于Java字节码的语言，但也支持其他语言，如JavaScript，Ruby，Python或LLVM。性能是Nashorn的2倍以上。

JDK15移除了Nashorn JavaScript Engine及jjs 命令行工具。具体就是jdk.scripting.nashorn及jdk.scripting.nashorn.shell这两个模块被移除了。

# Java16新特性

## 1、JEP 338: Vector API (孵化阶段)

提供了jdk.incubator.vector来用于矢量计算，实例如下

```java
static final VectorSpecies<Float> SPECIES = FloatVector.SPECIES_256;

void vectorComputation(float[] a, float[] b, float[] c) {

    for (int i = 0; i < a.length; i += SPECIES.length()) {
        var m = SPECIES.indexInRange(i, a.length);
        // FloatVector va, vb, vc;
        var va = FloatVector.fromArray(SPECIES, a, i, m);
        var vb = FloatVector.fromArray(SPECIES, b, i, m);
        var vc = va.mul(va).
                    add(vb.mul(vb)).
                    neg();
        vc.intoArray(c, i, m);
    }
}
123456789101112131415
```

## 2、JEP 389: Foreign Linker API (孵化阶段)

提供jdk.incubator.foreign来简化native code的调用。

易用: 用一个纯Java开发模型替换JNI.
支持C语言:
通用性:
高性能:

## 3、instanceof模式匹配、Record、jpackage打包工具开始正式使用

1. instanceof的模式匹配在JDK14作为preview，在JDK15作为第二轮的preview，在JDK16转正
2. Record类型在JDK14作为preview，在JDK15处于第二轮preview，在JDK16转正
3. jpackage在JDK14引入，JDK15作为incubating工具，在JDK16转正，从`jdk.incubator.jpackage`转为`jdk.jpackage`。它支持Linux: deb and rpm、macOS: pkg and dmg、Windows: msi and exe

## 4、ZGC优化

实现了并发thread-stack处理来降低GC safepoints的负担

## 5、Elastic Metaspace

及时地将未使用的 HotSpot 类元数据（即元空间）内存返回给操作系统，减少元空间占用，并简化元空间代码以降低维护成本。

# Java17新特性

## 1、Sealed 密封类转正

sealed class 密封类允许描述哪个类或接口可以扩展或实现这个类或接口。简而言之，我们可以限制谁可以使用这个类或接口。

## 2、提供更好的伪随机数生成

为伪随机数生成器 (PRNG) 提供新的接口类型和实现，包括可跳转 PRNG 和另一类可拆分 PRNG 算法 (LXM)。

引入了一个名为`RandomGenerator`的新接口。该接口的目标是为所有现有和新的PRNG提供统一的API。

`RandomGenerator`提供名为`ints、longs、doubles、nextBoolean、nextInt、nextLong、nextDouble和nextFloat`的方法。

## 3、Switch模式匹配（预览）

通过对 switch 表达式和语句的模式匹配以及对模式语言的扩展来增强 Java 编程语言。将模式匹配扩展到 switch 允许针对多个模式测试表达式，每个模式都有特定的操作，因此可以简洁安全地表达复杂的面向数据的查询。这是 JDK 17 中的预览语言功能。

```java
static String formatterPatternSwitch(Object o) {
    return switch (o) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> o.toString();
    };
}
123456789
```

## 4、用于特定于上下文的反序列化过滤器

## 5、浮点运算更加严格

简化数字敏感库开发，包括java.lang.Math和java.lang.StrictMath

## 6、删除一些功能

### 1、删除实验性 AOT 和 JIT 编译器

### 2、删除Applet API

### 3、弃用安全管理器Security Manager

弃用安全管理器以便在将来的版本中删除。安全管理器可追溯到 Java 1.0。多年来，它一直不是保护客户端 Java 代码的主要方法，也很少用于保护服务器端代码。为了推动 Java 向前发展，我们打算弃用安全管理器，以便与旧版 Applet API (JEP 398) 一起删除。

### 4、移除RMI激活机制

删除远程方法调用 (RMI) 激活机制，同时保留 RMI 的其余部分。

[![img](https://profile.csdnimg.cn/9/E/8/3_small_to_large)



[![CSDN首页](https://img-home.csdnimg.cn/images/20201124032511.png)](https://www.csdn.net/)

- [博客](https://blog.csdn.net/)
- [下载](https://download.csdn.net/)
- [学习](https://edu.csdn.net/)
- [社区](https://bbs.csdn.net/)
- [GitCode](https://gitcode.net/?utm_source=csdn_toolbar)
- [云服务](https://dev-portal.csdn.net/welcome?utm_source=toolbar)
- [猿如意 ![img](https://img-home.csdnimg.cn/images/20220830073411.png)](https://devbit.csdn.net/?source=csdn_toolbar)



 搜索

[![img](https://profile.csdnimg.cn/5/7/1/2_weixin_51651455)](https://blog.csdn.net/weixin_51651455)

[会员中心 ![img](https://img-home.csdnimg.cn/images/20210918025138.gif)](https://mall.csdn.net/vip)

[足迹](https://i.csdn.net/#/user-center/collection-list?type=1)

[动态](https://blink.csdn.net/)

[消息](https://i.csdn.net/#/msg/index)

[创作中心 ![img](https://img-home.csdnimg.cn/images/20220627041202.png)](https://mp.csdn.net/)

[发布](https://mp.csdn.net/edit)

# Java 18 新特性



### 文章目录

- [Java 18 新特性](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#Java_18__1)
- - [1、默认 UTF-8 字符编码](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#1_UTF8__27)
  - [2、简单的 Web 服务器](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#2_Web__63)
  - [3、Javadoc 中支持代码片段](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#3Javadoc__125)
  - - [3.1、高亮代码片段](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#31_149)
    - [3.2、正则高亮代码片段](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#32_180)
    - [3.3、替换代码片段](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#33_206)
    - [3.4、附：Javadoc 生成方式](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#34Javadoc__237)
  - [4、使用方法句柄重新实现反射核心功能](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#4_251)
  - [5、Vector API（三次孵化）](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#5Vector_API_301)
  - [6、互联网地址解析 SPI](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#6_SPI_311)
  - [7、Foreign Function & Memory API (第二次孵化）](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#7Foreign_Function__Memory_API__324)
  - [8、switch 表达式（二次孵化）](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#8switch__340)
  - [9、弃用删除相关](https://blog.csdn.net/wang_luwei/article/details/125320523?ops_request_misc=%7B%22request%5Fid%22%3A%22166286898316782248548337%22%2C%22scm%22%3A%2220140713.130102334.pc%5Fall.%22%7D&request_id=166286898316782248548337&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~first_rank_ecpm_v1~times_rank-1-125320523-null-null.142^v47^body_digest,201^v3^control_1&utm_term=java18新特性&spm=1018.2226.3001.4187#9_424)



# Java 18 新特性

Java 18 在 2022 年 3 月 22 日正式发布，Java 18 不是一个长期支持版本，这次更新共带来 9 个新功能。

Java 18 全部的新特性，请看官网：[JDK 18 发行说明](https://www.oracle.com/java/technologies/javase/18-relnote-issues.html#NewFeature)

Java各个版本的文档入口：[Java平台，标准版文档](https://docs.oracle.com/en/java/javase/index.html)

Java各个版本下载：https://jdk.java.net/archive/

| JEP     | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| JEP 400 | [默认为 UTF-8(opens new window)](https://openjdk.java.net/jeps/400) |
| JEP 408 | [简单的网络服务器(opens new window)](https://openjdk.java.net/jeps/408) |
| JEP 413 | [Java API 文档中的代码片段(opens new window)](https://openjdk.java.net/jeps/413) |
| JEP 416 | [使用方法句柄重新实现核心反射(opens new window)](https://openjdk.java.net/jeps/416) |
| JEP 417 | [Vector API（三次孵化）(opens new window)](https://openjdk.java.net/jeps/417) |
| JEP 418 | [互联网地址解析 SPI(opens new window)](https://openjdk.java.net/jeps/418) |
| JEP 419 | [Foreign Function & Memory API (二次孵化)(opens new window)](https://openjdk.java.net/jeps/419) |
| JEP 420 | [switch 模式匹配（二次预览）(opens new window)](https://openjdk.java.net/jeps/420) |
| JEP 421 | [弃用完成删除(opens new window)](https://openjdk.java.net/jeps/421) |

## 1、默认 UTF-8 字符编码

JDK 一直都是支持 UTF-8 字符编码，这次是把 UTF-8 设置为了默认编码，也就是在不加任何指定的情况下，默认所有需要用到编码的 JDK [API](https://so.csdn.net/so/search?q=API&spm=1001.2101.3001.7020) 都使用 UTF-8 编码，这样就可以避免因为不同系统，不同地区，不同环境之间产生的编码问题。

> Mac OS 默认使用 UTF-8 作为默认编码，但是其他操作系统上，编码可能取决于系统的配置或者所在区域的设置。如中国大陆的 windows 使用 GBK 作为默认编码。很多同学初学 Java 时可能都遇到过一个正常编写 Java 类，在 windows 系统的命令控制台中运行却出现乱码的情况。

使用下面的命令可以输出 JDK 的当前编码。

```shell
# Mac 系统，默认 UTF-8
➜  ~ java -XshowSettings:properties -version 2>&1 | grep file.encoding
    file.encoding = UTF-8
    file.encoding.pkg = sun.io
➜  ~
12345
```

下面编写一个简单的 Java 程序，输出默认字符编码，然后输出中文汉字 ” 你好 “，看看 Java 18 和 Java 17 运行区别。

系统环境：Windows 11

```java
import java.nio.charset.Charset;

public class Hello{
    public static void main(String[] args) {
        System.out.println(Charset.defaultCharset());
        System.out.println("你好");
    }
}
12345678
```

从下面的运行结果中可以看到，使用 JDK 17 运行输出的默认字符编码是 GBK，输出的中文 ” 你好 “已经乱码了；乱码是因为 VsCode 默认的文本编辑器编码是 UTF-8，而中国地区的 Windows 11 默认字符编码是 GBK，也是 JDK 17 默认获取到的编码，所以会在控制台输出时乱码；而使用 JDK 18 输出的默认编码就是 UTF-8，所以可以正常的输出中文” 你好 “。

![在这里插入图片描述](https://img-blog.csdnimg.cn/80e8bc5bc9ba4fd49dcbd6f63368902b.png#pic_center)

## 2、简单的 Web 服务器

在 Java 18 中，提供了一个新命令 `jwebserver`，运行这个命令可以启动一个**简单的 、最小化的**静态 Web 服务器，它不支持 CGI 和 Servlet，所以最好的使用场景是用来测试、教育、演示等需求。

其实在如 Python、Ruby、PHP、Erlang 等许多平台都提供了开箱即用的 Web 服务器，可见一个简单的 Web 服务器是一个常见的需求，Java 一直没有这方面的支持，现在可以了。

在 Java 18 中，使用 `jwebserver` 启动一个 Web 服务器，默认发布的是当前目录。

在当前目录创建一个网页文件 index.html

```html
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
</head>
<body>
<h1>标题</h1>
</body>
</html>
12345678
```

启动 `jwebserver`.

```shell
➜  bin ./jwebserver
Binding to loopback by default. For all interfaces use "-b 0.0.0.0" or "-b ::".
Serving /Users/darcy/develop/jdk-18.jdk/Contents/Home/bin and subdirectories on 127.0.0.1 port 8000
URL http://127.0.0.1:8000/
1234
```

浏览器访问：

![在这里插入图片描述](https://img-blog.csdnimg.cn/8e75b6a6cf694d45a069d1e9e356c124.png#pic_center)

有请求时会在控制台输出请求信息：

```shell
127.0.0.1 - - [26/3月/2022:16:53:30 +0800] "GET /favicon.ico HTTP/1.1" 404 -
127.0.0.1 - - [26/3月/2022:16:55:13 +0800] "GET / HTTP/1.1" 200 -
12
```

通过 `help` 参数可以查看 `jwebserver` 支持的参数。

```shell
➜  bin ./jwebserver --help
Usage: jwebserver [-b bind address] [-p port] [-d directory]
                  [-o none|info|verbose] [-h to show options]
                  [-version to show version information]
Options:
-b, --bind-address    - 绑定地址. Default: 127.0.0.1 (loopback).
                        For all interfaces use "-b 0.0.0.0" or "-b ::".
-d, --directory       - 指定目录. Default: current directory.
-o, --output          - Output format. none|info|verbose. Default: info.
-p, --port            - 绑定端口. Default: 8000.
-h, -?, --help        - Prints this help message and exits.
-version, --version   - Prints version information and exits.
To stop the server, press Ctrl + C.
12345678910111213
```

## 3、Javadoc 中支持代码片段

在 Java 18 之前，已经支持在 Javadoc 中引入代码片段，这样可以在某些场景下更好的展示描述信息，但是之前的支持功能有限，比如我想高亮代码片段中的某一段代码是无能为力的。现在 Java 18 优化了这个问题，增加了 `@snippet` 来引入更高级的代码片段。

在 Java 18 之前，使用 `<pre>{@code ...}</pre>` 来引入代码片段。

```java
 /**
  * 时间工具类
  * Java 18 之前引入代码片段：
  * <pre>{@code
  *     public static String timeStamp() {
  *        long time = System.currentTimeMillis();
  *         return String.valueOf(time / 1000);
  *     }
  * }</pre>
  *
  */
1234567891011
```

生成 Javadoc 之后，效果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/aff1678e8e2f4a5fb2616333fbc9e945.png#pic_center)

### 3.1、高亮代码片段

从 Java 18 开始，可以使用 `@snippet` 来生成注释，且可以高亮某个代码片段。

```java
/**
 * 在 Java 18 之后可以使用新的方式
 * 下面的代码演示如何使用 {@code Optional.isPresent}:
 * {@snippet :
 * if (v.isPresent()) {
 *     System.out.println("v: " + v.get());
 * }
 * }
 *
 * 高亮显示 println
 *
 * {@snippet :
 * class HelloWorld {
 *     public static void main(String... args) {
 *         System.out.println("Hello World!");      // @highlight substring="println"
 *     }
 * }
 * }
 *
 */
1234567891011121314151617181920
```

效果如下，更直观，效果更好。
![在这里插入图片描述](https://img-blog.csdnimg.cn/94efb35d5b994a6aade36b2c78d9b655.png#pic_center)

### 3.2、正则高亮代码片段

甚至可以使用正则来高亮某一段中的某些关键词：

```java
/** 
  * 正则高亮：![在这里插入图片描述](https://img-blog.csdnimg.cn/ae58ca6d07a542e98479b660d284b991.png#pic_center)

  * {@snippet :
  *   public static void main(String... args) {
  *       for (var arg : args) {                 // @highlight region regex = "\barg\b"
  *           if (!arg.isBlank()) {
  *               System.out.println(arg);
  *           }
  *       }                                      // @end
  *   }
  *   }
  */
12345678910111213
```

生成的 Javadoc 效果如下:
![在这里插入图片描述](https://img-blog.csdnimg.cn/aaf70676efcf4c0bb0f64268340af814.png#pic_center)

### 3.3、替换代码片段

可以使用正则表达式来替换某一段代码。

```java
 /** 
   * 正则替换：
   * {@snippet :
   * class HelloWorld {
   *     public static void main(String... args) {
   *         System.out.println("Hello World!");  // @replace regex='".*"' replacement="..."
   *     }
   * }
   * }
   */
12345678910
```

这段注释会生成如下 Javadoc 效果。

```java
class HelloWorld {
    public static void main(String... args) {
        System.out.println(...);
    }
}
12345
```

### 3.4、附：Javadoc 生成方式

```shell
# 使用 javadoc 命令生成 Javadoc 文档
➜  bin ./javadoc -public -sourcepath ./src -subpackages com -encoding utf-8 -charset utf-8 -d ./javadocout
# 使用 Java 18 的 jwebserver 把生成的 Javadoc 发布测试
➜  bin ./jwebserver -d /Users/darcy/develop/javadocout
1234
```

访问测试：

![在这里插入图片描述](https://img-blog.csdnimg.cn/81437cbc2a18445dac1c8e99c251a45e.png#pic_center)

## 4、使用方法句柄重新实现反射核心功能

Java 18 改进了 `java.lang.reflect.Method`、`Constructor` 的实现逻辑，使之性能更好，速度更快。这项改动不会改动相关 API ，这意味着开发中不需要改动反射相关代码，就可以体验到性能更好反射。

OpenJDK 官方给出了新老实现的反射性能基准测试结果。

Java 18 之前：

```shell
Benchmark                                     Mode  Cnt   Score  Error  Units
ReflectionSpeedBenchmark.constructorConst     avgt   10  68.049 ± 0.872  ns/op
ReflectionSpeedBenchmark.constructorPoly      avgt   10  94.132 ± 1.805  ns/op
ReflectionSpeedBenchmark.constructorVar       avgt   10  64.543 ± 0.799  ns/op
ReflectionSpeedBenchmark.instanceFieldConst   avgt   10  35.361 ± 0.492  ns/op
ReflectionSpeedBenchmark.instanceFieldPoly    avgt   10  67.089 ± 3.288  ns/op
ReflectionSpeedBenchmark.instanceFieldVar     avgt   10  35.745 ± 0.554  ns/op
ReflectionSpeedBenchmark.instanceMethodConst  avgt   10  77.925 ± 2.026  ns/op
ReflectionSpeedBenchmark.instanceMethodPoly   avgt   10  96.094 ± 2.269  ns/op
ReflectionSpeedBenchmark.instanceMethodVar    avgt   10  80.002 ± 4.267  ns/op
ReflectionSpeedBenchmark.staticFieldConst     avgt   10  33.442 ± 2.659  ns/op
ReflectionSpeedBenchmark.staticFieldPoly      avgt   10  51.918 ± 1.522  ns/op
ReflectionSpeedBenchmark.staticFieldVar       avgt   10  33.967 ± 0.451  ns/op
ReflectionSpeedBenchmark.staticMethodConst    avgt   10  75.380 ± 1.660  ns/op
ReflectionSpeedBenchmark.staticMethodPoly     avgt   10  93.553 ± 1.037  ns/op
ReflectionSpeedBenchmark.staticMethodVar      avgt   10  76.728 ± 1.614  ns/op
12345678910111213141516
```

Java 18 的新实现：

```shell
Benchmark                                     Mode  Cnt    Score   Error  Units
ReflectionSpeedBenchmark.constructorConst     avgt   10   32.392 ± 0.473  ns/op
ReflectionSpeedBenchmark.constructorPoly      avgt   10  113.947 ± 1.205  ns/op
ReflectionSpeedBenchmark.constructorVar       avgt   10   76.885 ± 1.128  ns/op
ReflectionSpeedBenchmark.instanceFieldConst   avgt   10   18.569 ± 0.161  ns/op
ReflectionSpeedBenchmark.instanceFieldPoly    avgt   10   98.671 ± 2.015  ns/op
ReflectionSpeedBenchmark.instanceFieldVar     avgt   10   54.193 ± 3.510  ns/op
ReflectionSpeedBenchmark.instanceMethodConst  avgt   10   33.421 ± 0.406  ns/op
ReflectionSpeedBenchmark.instanceMethodPoly   avgt   10  109.129 ± 1.959  ns/op
ReflectionSpeedBenchmark.instanceMethodVar    avgt   10   90.420 ± 2.187  ns/op
ReflectionSpeedBenchmark.staticFieldConst     avgt   10   19.080 ± 0.179  ns/op
ReflectionSpeedBenchmark.staticFieldPoly      avgt   10   92.130 ± 2.729  ns/op
ReflectionSpeedBenchmark.staticFieldVar       avgt   10   53.899 ± 1.051  ns/op
ReflectionSpeedBenchmark.staticMethodConst    avgt   10   35.907 ± 0.456  ns/op
ReflectionSpeedBenchmark.staticMethodPoly     avgt   10  102.895 ± 1.604  ns/op
ReflectionSpeedBenchmark.staticMethodVar      avgt   10   82.123 ± 0.629  ns/op
12345678910111213141516
```

可以看到在某些场景下性能稍微好些。

## 5、Vector API（三次孵化）

在 Java 16 中引入一个新的 API 来进行向量计算，它可以在运行时可靠的编译为支持的 CPU 架构，从而实现更优的计算能力。

在 Java 17 中改进了 Vector API 性能，增强了例如对字符的操作、字节向量与布尔数组之间的相互转换等功能。

现在在 JDK 18 中将继续优化其性能。

## 6、互联网地址解析 SPI

对于互联网地址解析 SPI，为主机地址和域名地址解析定义一个 SPI，以便 `java.net.InetAddress` 可以使用平台内置解析器以外的解析器。

```java
InetAddress inetAddress = InetAddress.getByName("www.wdbyte.com");
System.out.println(inetAddress.getHostAddress());
// 输出
// 106.14.229.49
1234
```

## 7、Foreign Function & Memory API (第二次孵化）

新的 API 允许 Java 开发者与 JVM 之外的代码和数据进行交互，通过调用外部函数，可以在不使用 JNI 的情况下调用本地库。

这是一个孵化功能；需要添加 `--add-modules jdk.incubator.foreign` 来编译和运行 Java 代码，Java 18 改进了相关 API ，使之更加简单易用。

*历史*

- Java 14 [JEP 370 (opens new window) (opens new window)](https://openjdk.java.net/jeps/370)引入了外部内存访问 API（孵化器）。
- Java 15 [JEP 383 (opens new window) (opens new window)](https://openjdk.java.net/jeps/383)引入了外部内存访问 API（第二孵化器）。
- Java 16 [JEP 389 (opens new window) (opens new window)](https://openjdk.java.net/jeps/389)引入了外部链接器 API（孵化器）。
- Java 16 [JEP 393 (opens new window) (opens new window)](https://openjdk.java.net/jeps/393)引入了外部内存访问 API（第三孵化器）。
- Java 17 [JEP 412 (opens new window) (opens new window)](https://openjdk.java.net/jeps/412)引入了外部函数和内存 API（孵化器）。

## 8、switch 表达式（二次孵化）

从 Java 17 开始，对于 Switch 的改进就已经在进行了，Java 17 的 JEP 406 已经对 Switch 表达式进行了增强，使之可以减少代码量。

下面是几个例子：

```java
// JDK 17 以前
static String formatter(Object o) {
    String formatted = "unknown";
    if (o instanceof Integer i) {
        formatted = String.format("int %d", i);
    } else if (o instanceof Long l) {
        formatted = String.format("long %d", l);
    } else if (o instanceof Double d) {
        formatted = String.format("double %f", d);
    } else if (o instanceof String s) {
        formatted = String.format("String %s", s);
    }
    return formatted;
}
1234567891011121314
```

而在 Java 17 之后，可以通过下面的写法进行改进：

```java
// JDK 17 之后
static String formatterPatternSwitch(Object o) {
    return switch (o) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        default        -> o.toString();
    };
}
12345678910
```

switch 可以和 `null` 进行结合判断：

```java
static void testFooBar(String s) {
    switch (s) {
        case null         -> System.out.println("Oops");
        case "Foo", "Bar" -> System.out.println("Great");
        default           -> System.out.println("Ok");
    }
}
1234567
```

case 时可以加入复杂表达式：

```java
static void testTriangle(Shape s) {
    switch (s) {
        case Triangle t && (t.calculateArea() > 100) ->
            System.out.println("Large triangle");
        default ->
            System.out.println("A shape, possibly a small triangle");
    }
}
12345678
```

case 时可以进行类型判断：

```java
sealed interface S permits A, B, C {}
final class A implements S {}
final class B implements S {}
record C(int i) implements S {}  // Implicitly final

static int testSealedExhaustive(S s) {
    return switch (s) {
        case A a -> 1;
        case B b -> 2;
        case C c -> 3;
    };
}
123456789101112
```

> 扩展：[JEP 406：Switch 的类型匹配](https://bugs.openjdk.org/browse/JDK-8273326)

## 9、弃用删除相关

在未来将删除 Finalization，目前 Finalization 仍默认保持启用状态，但是已经可以手动禁用；在未来的版本中，将会默认禁用；在以后的版本中，它将被删除。需要进行资源管理可以尝试 `try-with-resources` 或者 `java.lang.ref.Cleaner`。

