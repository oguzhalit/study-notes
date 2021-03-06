# 1 JavaSE
## 1.1权限
### 1.1.1 Java中的四种访问权限

public、protected、default、private，其中public具有最大权限，private则是最小权限。成员变量使用`private` （隐藏细节）、构造方法使用` public` （方便创建对象）、成员方法使用`public` （方便调用方法）。

## 1.2 内部类
### 1.2.1 匿名内部类

匿名内部类是内部类的简化写法。它的本质是一个带具体实现的父类或者父接口的匿名的子类对象。

```java
public interface FlyAble {
    // 抽象方法 :
    void fly();
}
```

```java
public class Test {
    public static void main(String[] args) {
// 写法一 : 多态语法
FlyAble flyAble = new FlyAble() {
    @Override
    public void fly() {
System.out.println("鸟儿在天上飞 ...");
    }
};
showFly(flyAble);
    }

    public static void showFly(FlyAble flyAble) {
// 调用 fly 方法
flyAble.fly();
    }
}
```
## 1.3 引用
### 1.3.1 引用类型方法总结

类作为成员变量，用于丰富类的定义；接口是对方法的封装，用于扩展类的功能。

## 1.4 常用 API
### 1.4.1 DateFormat

构造方法：`public SimpleDateFormat(String pattern)`，参数pattern是一个字符串，代表日期时间的自定义格式。常用的格式规则为：

| 标识字母（区分大小写） | 含义 |
| ---------------------- | ---- |
| y      | 年   |
| M      | 月   |
| d      | 日   |
| H      | 时   |
| m      | 分   |
| s      | 秒   |

format方法：`public String format(Date date)，`将Date对象格式化为字符串。

```java
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;
/*
 把Date对象转换成String
*/
public class Demo03DateFormatMethod {
    public static void main(String[] args) {
Date date = new Date();
// 创建日期格式化对象,在获取格式化对象时可以指定风格
DateFormat df = new SimpleDateFormat("yyyy年MM月dd日");
String str = df.format(date);
System.out.println(str); // 2008年1月23日
    }
}
```

parse方法：`public Date parse(String source)，`将字符串解析为Date对象。

```java
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
/*
 把String转换成Date对象
*/
public class Demo04DateFormatMethod {
    public static void main(String[] args) throws ParseException {
DateFormat df = new SimpleDateFormat("yyyy年MM月dd日");
String str = "2018年12月11日";
Date date = df.parse(str);
System.out.println(date); // Tue Dec 11 00:00:00 CST 2018
    }
}
```

### 1.4.2 Math
#### 1.4.2.1 random

```java
//    产生一个从 a 到 b 的随机数的方法 : 先用 random.nextDouble() 产生一个从 0 到 1 的小数，然后乘以 (b - a) , 然后再加上 a , 然后再将得到的结果转化为整数.
private static int getRandom(int a, int b) {
    return (int)(Math.random() * (b - a) + a);
}
```
### 1.4.3 ResourceBundle

```java
//    读取 classpath 路径下的 my.properties 文件
ResourceBundle bundle = ResourceBundle.getBundle("my");
//    先读取 classpath 路径下的 my_zh_CN.properties 文件，如果该文件不存在，则读取 my_zh.properties 文件，如果该文件不存在，则读取 my.properties 文件
ResourceBundle bundle = ResourceBundle.getBundle("my", new Locale("zh", "CN"));
```
### 1.4.4 StringUtils (org.apache.commons.lang3.StringUtils)
#### 1.4.4.1 isNotBlank

```java
if (StringUtils.isNotBlank(key)) {
    criteria.orLike("title", "%" + key + "%");
}
```
#### 1.4.4.2 join

```java
List<String> names = categories.stream().map(Category::getName).collect(Collectors.toList());
String name = StringUtils.join(names, " ");
```
#### 1.4.4.3 substringBefore

```java
StringUtils.substringBefore(sku.getImages(), ",")
```

### 1.4.5 CollectionUtils (org.springframework.util.CollectionUtils)
#### 1.4.5.1 isEmpty

```java
if (CollectionUtils.isEmpty(specGroups)) {
    throw new LyException(ExceptionEnum.SPEC_GROUP_NOT_FOUND);
}
```

## 1.5 集合
### 1.5.1 什么是集合，集合和数组的区别是什么

集合是java中提供的一种容器，可以用来存储多个数据。

集合和数组的区别：

数组的长度是固定的。集合的长度是可变的。

数组中存储的是同一类型的元素，可以存储基本数据类型值。集合存储的都是对象。而且对象的类型可以不一致。在开发中一般当对象多的时候，使用集合进行存储。

### 1.5.2 简述集合框架

集合是单列集合类的根接口，用于存储一系列符合某种规则的元素，它有两个重要的子接口，分别是`java.util.List`和`java.util.Set`。其中，`List`的特点是元素有序、元素可重复。`Set`的特点是元素无序，而且不可重复。`List`接口的主要实现类有`java.util.ArrayList`和`java.util.LinkedList`，`Set`接口的主要实现类有`java.util.HashSet`和`java.util.TreeSet`。

![](../img/1.png)

### 1.5.3 什么是迭代

迭代是指Collection集合元素的通用获取方式。在取元素之前先要判断集合中有没有元素，如果有，就把这个元素取出来，继续在判断，如果还有就再取出出来。一直把集合中的所有元素全部取出。这种取出方式专业术语称为迭代。

### 1.5.4 增强for循环的原理是什么

增强for循环的内部原理其实是个Iterator迭代器，所以在遍历的过程中，不能对集合中的元素进行增删操作。

### 1.5.5 List接口的特点

它是一个元素存取有序的集合。

它是一个带有索引的集合，通过索引就可以精确的操作集合中的元素（与数组的索引是一个道理）。

集合中可以有重复的元素，通过元素的equals方法，来比较是否为重复的元素。

### 1.5.6 HashSet去除重复的原理

通过hashCode与equals方法，保证了元素的唯一性。

### 1.5.7 Comparable和Comparator两个接口的区别

**Comparable**：强行对实现它的每个类的对象进行整体排序。这种排序被称为类的自然排序，类的compareTo方法被称为它的自然比较方法。只能在类中实现compareTo()一次，不能经常修改类的代码实现自己想要的排序。实现此接口的对象列表（和数组）可以通过Collections.sort（和Arrays.sort）进行自动排序，对象可以用作有序映射中的键或有序集合中的元素，无需指定比较器。

**Comparator**强行对某个对象进行整体排序。可以将Comparator 传递给sort方法（如Collections.sort或 Arrays.sort），从而允许在排序顺序上实现精确控制。还可以使用Comparator来控制某些数据结构（如有序set或有序映射）的顺序，或者为那些没有自然顺序的对象collection提供排序。

### 1.5.8 List和Set接口的区别

List接口：有序、可重复

Set接口：无序、不可重复

### 1.5.9 为什么ArrayList类的父类和爷爷类都是Abstract类

父类：AbstractList implements List List接口定义了很多方法. 而Abstarct类没有对应List接口的方法进行所有实现, 因为该类包含了抽象方法, 所以该类必须被定义为 '抽象类'. 无法实例化对象.

爷爷类：AbstractCollection implements Collection Collection接口中同样定义了很多方法. AbstractCollection类并没有对该接口进行全部方法的实现, 因此该类中也包含抽象方法, 所以该类必须被定义为抽象类, 抽象类无法实例化对象.

### 1.5.10 AbstractList & AbstractCollection两个类都无法被实例化, 那么其存在的目的是什么

存在的目的就是为了给下一代类提供功能. 下一代类就可以直接继承父类中已经完成的功能, 无需再次实现了.

### 1.5.11 为什么ArrayList就可以实例化对象了呢

因为ArrayList类将AbstractList & AbstractCollection类中没有实现的方法, 进行了全部实现, 因此该类就不再为抽象类了.

### 1.5.12 Collection集合体系

![1530273400549](../img/2.png)

### 1.5.13 Map 的特点

1. Map中的集合为双列集合，元素是成对存在的，每个元素由键与值两部分组成，通过键可以找对所对应的值.
2. Map 中的集合不能包含重复的键，值可以重复，每个键只能对应一个值.

### 1.5.14 HashSet 和 HashMap 之间有什么关系

1. HashSet 是单列集合.
2. HashMap 是双列集合.
3. HashSet 实现 Set 接口，由哈希表 ( 实际上是一个 HashMap 实例 ) 支持，只要创建一个 HashSet 集合，底层创建的就是 HashMap 集合.
4. HashSet 底层就是 HashMap 支持，但是，HashSet 仅仅使用了 HashMap 的 key，而没有使用 HashMap 的 value.

### 1.5.15 HashMap 中重写 hashCode 和 equals 方法的步骤

```java
	@Override
    public int hashCode() {
return Objects.hash(name, age);
    }

	@Override
    public boolean equals(Object obj) {
if(this == obj) return true;
if(obj == null || this.getClass() != obj.getClass()) return false;
Student student = (Student) obj;
return this.age == student.getAge() && Objects.equals(this.name, student.getName());
    }
```

## 1.6 泛型
### 1.6.1 什么是泛型

泛型是指可以在类或方法中预支地使用未知的类型。一般在创建对象时，将未知的类型确定具体的类型。当没有指定泛型时，默认类型为Object类型。

### 1.6.2 使用泛型的好处有哪些

将运行时期的ClassCastException，转移到了编译时期变成了编译失败。

避免了类型强转的麻烦。

## 1.7 多态
### 1.7.1 多态的前提

继承或者实现

方法的重写

父类引用指向子类对象（格式体现）

```
父类类型 变量名 = new 子类对象；
变量名.方法名();
```

```java
Fu f = new Zi();
f.method();
```

当使用多态方式调用方法时，首先检查父类中是否有该方法，如果没有，则编译错误；如果有，执行的是子类重写后方法。

### 1.7.2 多态的好处

实际开发的过程中，父类类型作为方法形式参数，传递子类对象给方法，进行方法的调用，更能体现出多态的扩展性与便利。

### 1.7.3 向上转型与向下转型

**向上转型：**父类引用指向子类对象

使用格式：

```java
父类类型  变量名 = new 子类类型();
如：Animal a = new Cat();
```

**向下转型：**父类引用指向子类引用

使用格式：

```java
子类类型 变量名 = (子类类型) 父类变量名;
如:Cat c =(Cat) a;  
```

## 1.8 异常
### 1.8.1 异常的分类

**编译时期异常：**checked 异常，在编译时期，就会检查，如果没有处理异常，则编译失败（如日期格式化异常）.

**运行时期异常：**runtime 异常，在运行时期，检查异常，运行异常不会被编译器检测（不报错）（如数学异常）.

### 1.8.2 异常的处理方式

**抛出异常 throw ：**throw 用在方法内，用来抛出一个异常对象，将这个异常对象传递到调用者处，并结束当前方法的执行.

使用格式：

```java
throw new 异常类名（参数）;
```

**捕获异常 try ... catch ... finally ：**Java 中对异常有针对性的语句进行捕获，可以对出现的异常进行指定方式的处理.

使用格式：

```java
try {
    编写可能会出现异常的代码
} catch (异常类型 e) {
    处理异常的代码
    //	记录日志
    //	打印异常信息
    //	继续抛出异常
} finally {
    关闭资源（硬件文件资源，数据库资源，网络资源 ...）
}
```

**try : ** 该代码块中编写可能产生异常的代码.

**catch :** 用来进行某种异常的捕获，实现对捕获到的异常进行处理.

**finally :** finally 中的代码块一定会被执行，就是为了 '关闭资源' 而设计，主要用于关闭硬件文件资源，数据库资源，网络资源 ... ，finally 块的设计，就是为了解决 catch 块中 return 语句的问题，在 return 之前会先执行 finally 代码块.

try ... catch ... finally 参考代码如下：

```java
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class ExceptionTest1 {
    public static void main(String[] args) {

String result = readFile("a.txt");
System.out.println("result = " + result);

    }

    // 需求 : IO (BufferedReader & BufferedWriter)
    // 说明 : 定义一个方法, 读取指定文件中的数据, 并返回是否读取成功字符串.
    public static String readFile(String fileName) {

// 1. 读取文件
// 说明 : 创建 FileReader 时, 会发生一个 `FileNotFoundException` 文件找不到异常.
// 选择捕获异常 : try - catch
FileReader reader = null;
try {
    reader = new FileReader(fileName);
} catch (FileNotFoundException e) {
    System.out.println("发生了文件找不到异常, 请检查文件路径或文件名称.");
    // 如果程序进入 catch 语句, 说明程序发生了异常. 文件读取失败了.
    return "文件读取失败";// return 之前会先执行 finally 代码块.
} finally {
    // finally 代码块就是为 `关闭资源` 而设计. (硬件文件资源, 数据库资源, 网络资源 ...)
    // 说明 : finally 块的设计就是为了解决 catch 块中 return 语句的问题.
    System.out.println("finally 代码块被执行 ...");
    // 说明 : 如果一个对象的默认值为null, 调用时, 必须要做 null 判断.
    if (reader != null) {
try {
    reader.close();
} catch (IOException exception) {
    // 忽略 ...
}
    }

    // finally 块中, 在开发时, 绝对不会出现 return 语句, 因为 finally 块就是用来关闭资源的, 而不是返回结果的.
    // 如果要返回结果, 请在 catch 语句中返回.
    // return "哈哈哈哈";
}

return "文件读取成功";
    }
}

输出结果 :
发生了文件找不到异常, 请检查文件路径或文件名称.
finally 代码块被执行 ...
result = 文件读取失败
```

### 1.8.3 try ... catch ... finally 执行顺序的思考

当有多个 catch 时，前面一个 catch 捕获异常后，后面的 catch 不会再执行.

参考代码如下：

```java
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class ExceptionTest1 {
    public static void main(String[] args) {

String result = readFile("a.txt");
System.out.println("result = " + result);

    }

    // 需求 : IO (BufferedReader & BufferedWriter)
    // 说明 : 定义一个方法, 读取指定文件中的数据, 并返回是否读取成功字符串.
    public static String readFile(String fileName) {

// 1. 读取文件
// 说明 : 创建 FileReader 时, 会发生一个 `FileNotFoundException` 文件找不到异常.
// 选择捕获异常 : try - catch
FileReader reader = null;
try {
    reader = new FileReader(fileName);
} catch (FileNotFoundException e) {
    System.out.println("发生了文件找不到异常, 请检查文件路径或文件名称.");
    // 如果程序进入 catch 语句, 说明程序发生了异常. 文件读取失败了.
    return "文件读取失败";// return 之前会先执行 finally 代码块.
} finally {
    // finally 代码块就是为 `关闭资源` 而设计. (硬件文件资源, 数据库资源, 网络资源 ...)
    // 说明 : finally 块的设计就是为了解决 catch 块中 return 语句的问题.
    System.out.println("finally 代码块被执行 ...");
    // 说明 : 如果一个对象的默认值为null, 调用时, 必须要做 null 判断.
    if (reader != null) {
try {
    reader.close();
} catch (IOException exception) {
    // 忽略 ...
}
    }

    // finally 块中, 在开发时, 绝对不会出现 return 语句, 因为 finally 块就是用来关闭资源的, 而不是返回结果的.
    // 如果要返回结果, 请在 catch 语句中返回.
    // return "哈哈哈哈";
}

return "文件读取成功";
    }
}

输出结果 :
发生了文件找不到异常, 请检查文件路径或文件名称.
finally 代码块被执行 ...
result = 文件读取失败
```

### 1.8.4 异常处理过程

如果有异常发生，并没有进行处理，此时该异常继续向上抛，该方法由谁调用，该异常就抛给谁. 由于 Java 虚拟机调用了 main 方法，所以该异常就抛给了 'Java虚拟机'，当虚拟机接收到异常时，就输出异常的相关信息，然后提前终止程序，退出Java虚拟机. 当对异常进行处理时，就根据处理方案对异常进行处理.

## 1.9 多线程
### 1.9.1 并行与并发

**并行：**指两个或多个事件在同一时刻发生（同时发生）.

**并发：**指两个或多个事件在同一个时间段内发生.

### 1.9.2 多线程执行过程

![](..\img/3.png)

1. 多线程执行时，在栈内存中，每一个执行线程都有一片自己所属的栈内存空间，进行方法的压栈和弹栈.
2. 每条线程都有自己独立的栈区执行空间，堆区只有一个.
3. 多线程中，每条线程的栈空间独立，堆空间 '共享'.
4. 堆空间中存储的是 '对象'，而对象中存储的是该对象的 '数据'.
5. 对象的属性存放在堆区中，局部变量存放在栈区中，多线程操作对象的属性会存在安全问题，多线程操作局部变量不会存在安全问题.

### 1.9.3 实现 Runnable 接口比继承 Thread 类所具有的优势

1. 适合多个相同的程序代码的线程去共享同一个资源.
2. 可以避免 Java 中的单继承的局限性.
3. 增加程序的健壮性，实现解耦操作，代码可以被多个线程共享，代码和线程独立.
4. 线程池只能放入实现 Runnable 或 Callable 类线程，不能直接放入继承 Thread 的类.

## 1.10 字节流与字符流
### 1.10.1 字节流与字符流的区别

![](..\img/4.png)

**字节流：**所有类型的数据在硬盘中都是以字节的形式实现存储的，字节流可以完成所有类型的 '读写' 操作.

**字符流：**字符流仅能操作字符数据，不能操作其他类型数据，字符流 = 字节流 + 文字编码表. 字符流写入数据三部曲 `写入、换行、刷新`

程序以内存作为参照物，区分读和写.

1. 数据从硬盘中，流入到内存，被称为读.
2. 数据从内存中，流入到硬盘，被称为写.

**字节流使用方法**

~~~java
/**
     * 复制文件（BufferedInputStream & BufferedOutputStream）
     * @throws IOException
     */

    public static void copyPicture2() throws IOException {

//  创建 File 对象
File origin = new File("file/柳岩.jpg");
File destination = new File("file/柳岩3.jpg");
//  创建文件输入、输出流对象
FileInputStream in = new FileInputStream(origin);
FileOutputStream out = new FileOutputStream(destination);
//  创建高效字节缓冲输入、输出流对象
BufferedInputStream bis = new BufferedInputStream(in);
BufferedOutputStream bos = new BufferedOutputStream(out);
try (in; out) {

    //  循环读取原文件数据并将其复制到目标文件
    int len = -1;
    while ((len = bis.read()) != -1) {
bos.write(len);
    }

} catch (IOException e) {
    e.printStackTrace();
}

System.out.println("文件复制完成");

    }
~~~

**字符流使用方法**

~~~java
/**
     * FileReader 的构造方法
     * @throws FileNotFoundException
     */

    public static void testFileReader() throws IOException {

//  读取字符数据

//  创建一个字符输入流
File file2 = new File("file/a.txt");
FileReader reader3 = new FileReader(file2);

//  循环读取数据
char[] c = new char[2];
int read = -1;
while ((read = reader3.read(c)) != -1) {
    String str = new String(c, 0, read);
    System.out.println(str);
}

//  关闭资源
reader3.close();

    }
~~~

~~~java
/**
     * FileWriter 的构造方法
     * @throws IOException
     */

    public static void testFileWriter() throws IOException {

//  字符缓冲流写入数据，写入数据三部曲：写入、换行、刷新

//  创建一个字符缓冲输出流对象
File file2 = new File("file/e.txt");
FileWriter fileWriter = new FileWriter(file2, true);
BufferedWriter writer2 = new BufferedWriter(fileWriter);

//  写入数据
writer2.write("我爱上海明珠塔.");
writer2.newLine();
writer2.flush();

//  关闭资源
writer2.close();

    }
~~~

## 1.11 函数式编程
### 1.11.1 方法中不定数组传参的方法

方法定义：

~~~java
public static int getNum(int... nums) {
    System.out.println(Arrays.toString(nums));
}
~~~

方法调用：

~~~java
private static void main(String[] args) {
    getNum(10, 20);
}
~~~

### 1.11.2 Lambda 表达式

**省略原则：**能推导, 就可以省略.

1. () 参数列表小括号中, 参数类型可以以省略, 如果参数仅有一个, 小括号也可以省略.
2. {} 如果方法体中有多条语句, 大括号必须书写. 如果方法体中仅有一条语句, 大括号就可以省略. 不管该方法有没有返回值, return 关键字和最后的分号都可以省略.

**使用前提条件：**

1. 必须拥有 `函数式接口`. (Java语言已经提供了很多函数式接口)
2. 调用的方法必须拥有函数式接口作为方法的参数. (Java语言已经提供了很多方法, 这些方法的参数都是函数式接口)

### 1.11.3 常用函数式接口

1. Supplier<T> : 无中生有，抽象方法为 T get() .
2. Consumer<T> : 有去无回，抽象方法为 void accept(T t)，默认方法为 andThen .
3. Predicate<T> : 元芳，你怎么看，抽象方法为 boolean test(T t)，默认方法为 and、or、negate(or) .
4. Function<T, R> : 有去有会，抽象方法为 R apply(T t)，默认方法为 andThen .

## 1.12 方法引用
### 1.12.1 方法引用的类型

1. 通过对象名引用成员方法.
2. 通过类名称引用静态方法.
3. 通过 super 引用成员方法.
4. 通过 this 引用成员方法.
5. 类的构造器引用.
6. 数组的构造器引用.

代码示例：

~~~java
//	通过对象名引用成员方法.

public class Assistant {
    public void dealFile(String file) {
System.out.println("帮忙处理 <" + file + "> 文件.");
    }
}

@FunctionalInterface
public interface WorkHelper {
    void help(String file);
}

public class Test2 {
    public static void main(String[] args) {

// 调用方法 : Lambda 表达式
work(s -> System.out.println("帮忙处理 <" + s + "> 文件."));

// 调用方法 : 对象引用对象方法
Assistant assistant = new Assistant();
work(assistant::dealFile);
    }

    public static void work(WorkHelper helper) {
helper.help("机密");
    }
}

输出结果 :
帮忙处理 <机密文件> 文件.
帮忙处理 <机密文件> 文件.
~~~

~~~java
//	通过类名称引用静态方法.

public class StringUtils {

    // 方法 : 判断是否为空
    public static boolean isBlank(String str) {
// 条件1 : str 参数不能为 null
// 条件2 : str 参数取出前后空格不能拿为空字符串
return str == null || "".equals(str.trim());
    }
}

@FunctionalInterface
public interface StringChecker {
    // 抽象方法
    boolean checkString(String str);
}

public class Test4 {
    public static void main(String[] args) {

// 调用方法 : Lambda 表达式
stringCheck("   ", s -> s == null || "".equals(s.trim()));

// 调用方法 : 静态方法引用
stringCheck("   ", StringUtils::isBlank);
    }

    public static void stringCheck(String str, StringChecker stringChecker) {
boolean result = stringChecker.checkString(str);
System.out.println("传入的字符串是否为空 : " + result);
    }
}

输出结果 :
传入的字符串是否为空 : true
传入的字符串是否为空 : true
~~~

~~~java
//	通过 super 引用成员方法.

@FunctionalInterface
public interface Greetable {
  	void greet();
}

public class Human {
    public void sayHello() {
      	System.out.println("Hello!");
    }
}

public class Man extends Human {

    public void sayHi() {

// 调用方法 : Lambda 表达式
method(() -> System.out.println("Hello!"));

// 调用方法 : 执行父类中的 sayHello 方法.
method(() -> super.sayHello());

// 调用方法 : super引用, 使用父类中的 sayHello 方法.
method(super::sayHello);
    }

    public void method(Greeting greeting) {
greeting.greet();

System.out.println("I am a Man.");
    }
}
~~~

~~~java
//	通过 this 引用成员方法.

@FunctionalInterface
public interface Richable {
    void buy();
}

public class Husband {

    // 行为 : 变得快乐
    public void beHappy() {
// 结婚吧
merry(() -> System.out.println("买套房子."));

merry(() -> this.buyCar());

merry(this::changeWife);
    }

    // 行为 : 结婚 (需要变得有钱, 必须要买东西)
    private void merry(Richable richable) {
richable.buy();
    }

    // 行为 : 买套方法
    private void buyHouse() {
System.out.println("买套房子.");
    }

    // 行为 : 买辆车子
    private void buyCar() {
System.out.println("买辆车子.");
    }

    // 行为 : 换个老婆
    private void changeWife() {
System.out.println("换个老婆. 开个玩笑.");
    }
}

public class Test {
    public static void main(String[] args) {

Husband husband = new Husband();
husband.beHappy();

    }
}

输出结果 :
买套房子.
买辆车子.
换个老婆. 开个玩笑.
~~~

~~~java
//	类的构造器引用.

public class Person {
    
    private String name;

    public Person(String name) {
this.name = name;
    }

    public Person() {
    }

    public String getName() {
return name;
    }

    public void setName(String name) {
this.name = name;
    }
}

@FunctionalInterface
public interface PersonBuilder {

    // 抽象方法
    Person builderPerson(String name);
}

public class Test1 {
    public static void main(String[] args) {

printPerson("张三丰", name -> new Person(name));

printPerson("张三丰", Person::new);
    }

    // 定义方法 :
    public static void printPerson(String name, PersonBuilder personBuilder) {
Person person = personBuilder.builderPerson(name);
System.out.println("person = " + person);
System.out.println("person.getName() = " + person.getName());
    }
}

输出结果 :
person = cn.itcast.test5.Person@7f63425a
person.getName() = 张三丰
person = cn.itcast.test5.Person@340f438e
person.getName() = 张三丰
~~~

~~~java
//	数组的构造器引用.

@FunctionalInterface
public interface ArrayBuilder {
    // 抽象方法
    int[] buildArray(int length);
}

public class Test2 {
    public static void main(String[] args) {

int[] arr1 = initIntArray(10, len -> new int[len]);
System.out.println("arr1.length = " + arr1.length);

int[] arr2 = initIntArray(10, int[]::new);
System.out.println("arr2.length = " + arr2.length);
    }

    // 初始化一个 int[] 数组
    public static int[] initIntArray(int length, ArrayBuilder builder) {
       return builder.buildArray(length);
    }
}

输出结果 :
arr1.length = 10
arr2.length = 10
~~~

## 1.13 Stream 流
### 1.13.1 Stream 常用方法

![](..\img/5.png)

1. count : 统计个数

~~~java
Stream<String> stream = Stream.of("张无忌", "张学友", "刘德华", "张三丰");

// long count();
long count = stream.filter(name -> name.startsWith("张")).count();
System.out.println("count = " + count);
~~~

2. forEach : 逐一处理

~~~java
Stream<String> stream = Stream.of("张无忌", "张学友", "刘德华", "张三丰");

// Consumer 接口 : void accept(T t)
stream.forEach(System.out::println);
~~~

3. filter : 过滤

~~~java
Stream<String> stream = Stream.of("张无忌", "张学友", "刘德华", "张三丰");

// Predicate 接口 : boolean test(T t);
stream.filter(name -> name.startsWith("张")).forEach(System.out::println);
~~~

4. limit : 取用前几个

~~~java
Stream<String> stream = Stream.of("张无忌", "张学友", "刘德华", "张三丰");

// Stream<T> limit(long maxSize);
stream.filter(name -> name.startsWith("张"))
.limit(2).forEach(System.out::println);
~~~

5. skip : 跳过前几个

~~~java
Stream<String> stream = Stream.of("张无忌", "张学友", "刘德华", "张三丰");

// Stream<T> skip(long n);
stream.filter(name -> name.startsWith("张"))
.skip(1).forEach(System.out::println);
~~~

6. map : 映射

~~~java
Stream<String> stream = Stream.of("10", "20", "30", "40", "50");

// Function 接口 : R apply(T t);
stream.map(Integer::parseInt).forEach(System.out::println);
~~~

7. concat : 组合

~~~java
Stream<String> stream1 = Stream.of("刘德华", "张学友", "黎明", "郭富城");
Stream<String> stream2 = Stream.of("西施", "杨玉环", "貂蝉", "王昭君");

Stream<String> stream = Stream.concat(stream1, stream2);
~~~

## 1.14 反射
### 1.14.1 反射常用方法

1. 获取 class 对象的 constructor 信息 : 

~~~java
//	根据参数类型获取构造方法对象，包括private修饰的构造方法。
Constructor getDeclaredConstructor(Class... parameterTypes) 

//	根据指定参数创建对象。
T newInstance(Object... initargs) 
   
//	暴力反射，设置为可以直接访问私有类型的构造方法。
void setAccessible(true)
~~~

~~~java
//	示例

// 1. 获取 Student 类表示的 Class 对象
Class<?> cls = Class.forName("cn.itcast.test2.Student");
// 2. 调用 getDeclaredConstructor 方法
Constructor<?> constructor = cls.getDeclaredConstructor(String.class, int.class);
// 3. 暴力反射 (设置可访问权限)
constructor.setAccessible(true);
// 4. 调用 Constructor 对象的 newInstance 方法, 创建对象
Object obj = constructor.newInstance("柳岩", 18);
~~~

2. 获取 class 对象的 method 信息

~~~java
//	根据方法名和参数类型获得一个方法对象，包括private修饰的
Method getDeclaredMethod("方法名", 方法的参数类型... 类型)

//	根据参数args调用对象obj的该成员方法，如果obj=null，则表示该方法是静态方法
Object invoke(Object obj, Object... args) 

//	暴力反射，设置为可以直接调用私有修饰的成员方法
void setAccessible(boolean flag) 
~~~

~~~java
//	示例

// 1. 获取 Student 类表示的 Class 对象
Class<?> cls = Class.forName("cn.itcast.test2.Student");
// 2. 调用 declaredMethod 方法
Method sleep = cls.getDeclaredMethod("fallInLove");
// 3. 暴力反射 (设置可访问权限)
sleep.setAccessible(true);
// 4. 调用 invoke 执行
Object obj = cls.getDeclaredConstructor().newInstance();
sleep.invoke(obj);
~~~

3. 获取 class 对象的 field 信息

~~~java
//	根据属性名获得属性对象，包括private修饰的
Field getDeclaredField(String name) 

//	设置属性值
void set(Object obj, Object value) 

//	获取属性值
Object get(Object obj)  

//	暴力反射，设置为可以直接访问私有类型的属性。
void setAccessible(true);

//	 获取属性的类型，返回Class对象。
Class getType();
~~~

~~~java
//	示例

// 1. 获取 Student 类表示的 Class 对象
 Class<?> cls = Class.forName("cn.itcast.test2.Student");
 // 2. 调用 getField 方法
 Field description = cls.getField("description");
 // 3. 设置属性
 Object obj = cls.getDeclaredConstructor().newInstance();
 description.set(obj, "这就是那个神奇的学生.");
 // 4. 获取属性
 Object desc = description.get(obj); 
~~~

## 1.15 动态代理
### 1.15.1 动态代理流程

![](..\img/6.png)

1. **ClassLoader :** 类加载器
2. **Class[] interfaces :** 接口数组
3. **InvacationHandler :** 调用处理器对象

**原理：**如果使用代理对象调用任何方法，最终都会执行 `调用处理器对象`的 invoke 方法.

**创建过程：**

~~~java
//	创建一个被代理对象
SuperStar star = new SuperStar();

//	创建一个代理对象
ClassLoader loader = star.getClass().getClassLoader();
Class<?> [] interfaces = star.getClass().getInterfaces();
MyInvocationHandler handler = new MyInvocationHandler(star);
SuperStar proxy = (SuperStar)Proxy.newProxyInstance(loader, interfaces, handler);

//	使用代理调用接口中的行为规范
proxy.sing(10);
proxy.liveShow(100);
proxy.sleep();

//	自定义 '调用处理器对象类'
public class MyInvocationHandler implements InvocationHandler {

    private SuperStar star;

    public MyInvocationHandler(SuperStar star) {
this.star = star;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

String methodName = method.getName();

//  如果方法是 'sing'，当 money 小于 10000 时，给出提示，不进行操作，否则，执行 'sing' 方法
if (methodName.equals("sing")) {

    int money = (int) args[0];  //  收入
    if (money < 10000) {
System.out.println("滚，你这个屌丝，赶快回家撸代码去 ...");
return null;
    } else {
//  代理抽取回扣
int rebate = (int) (money * 0.3);
money = money - rebate;
System.out.println("代理抽取回扣 : " + rebate);
return method.invoke(star, money);
    }

} else if (methodName.equals("liveShow")) {

    int money = (int) args[0];  //  收入
    if (money < 100000) {
System.out.println("滚，how old are you，回家继续撸代码去 ...");
return null;
    } else {
//  代理抽取回扣
int rebate = (int) (money * 0.3);
money = money - rebate;
System.out.println("代理抽取回扣 : " + rebate);
return method.invoke(star, money);
    }

}
return method.invoke(star, args);

    }
}
~~~

## 1.16 正则表达式
### 1.16.1 常用正则表达式

**符号 :** 

1. []	取值的范围. 0-9 数值0到9都成立.
   说明 : [0-9] 可以使用 `\\d` 表示

   . {}	表示前一个条件中 `值` 可以出现的次数.

   说明 : {4,11} 至少4次, 最多11次.
   ​	  {0,1}	  至少0次,最多一次.	可以使用 `？` 表示.
   ​	  {1,}	  至少1次,最多无限次	可以使用 `+` 表示.
   ​	  {0,}	  至少0次,最多无限次.	 可以使用 `*` 表示.

   . ()	表示分组. 在replaceAll方法的第二个参数上可以使用 $ 符号来引用之前的分组,分组编号自动从1开始.

**示例：**

~~~java
//	手机号码规则 :
//	1. 长度必须是11位
//	2. 第一位只能是数字1
//	3. 第二位可以是3, 4, 5, 7, 8

String phone = "13366285946";
boolean result = phone.matches("1[34578]\\d{9}");
~~~

~~~java
//	切割字符串

String str = "boxing#####basketball####football##ILOVEYOU##爱我中华";
String[] names = str.split("#+");

String str = "boxing123basketball4567football888ILOVEYOU9876爱我中华";
String[] names = str.split("\\d+");

String str = "boxingaaaaabasketballYYYYYfootball888ILOVEYOU9999爱我中华";
String[] names = str.split("(.)\\1{2,}");
~~~

~~~java
//	隐藏部分手机号码
//	13366285946  ->  133****5946

String phone = "13366285946";

/*
    源数据 : 13366285946
    第一部分 : 133      规则一 : 1[34578]\\d
    第二部分 : 6628     规则二 : \\d{4}
    第三部分 : 5946     规则三 : \\d{4}
 */

String result = phone.replaceAll("(1[34578]\\d)(\\d{4})(\\d{4})", "$1****$3");
System.out.println("result = " + result);
~~~