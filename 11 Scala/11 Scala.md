# 11 Scala
## 11.1 使用 Scala 逆序遍历数组

```scala
val arr = Array(1, 2, 3, 4)

//  正常顺序
for (i <- 0 until arr.length) {
  println(arr(i))
}

//  逆序
for (i <- (0 until arr.length).reverse) {
  println(arr(i))
}
```
## 11.2 Map 集合的用法

```scala
//    创建空的 Map 集合，这里要用实现类 HashMap，因为 Map 是抽象类，使用时要声明具体实现类
var map = new HashMap[String, Int]();

//    创建 Map 集合
var map = Map("金箍棒" -> 10.0, "装甲" -> 20.0, "机枪" -> 30.0, "飞机" -> 40.0)

//    向 Map 集合中添加元素
map.put("航母", 50)

//    从 Map 集合中删除元素
map.remove("装甲")

//    遍历 Map 集合
map.foreach {
    case (key, value) => println(key, value)
    case _ =>
}

//    判断 Map 集合中是否包含某元素
println(map.contains("航母"))

//    判断 Map 集合是否为空
println(map.isEmpty)

//    只打印 key 或 value
map.keys.foreach(println)
map.values.foreach(println)

//    对 Map 集合进行排序
map.toSeq.sortBy(_._1)    //    排序 key
map.toSeq.sortBy(_._2)    //    排序 value
map.toSeq.sortBy(_._1 > _._1)    //    降序排序 key
map.toSeq.sortBy(_._2 > _._2)    //    降序排序 value
```
## 11.3 Scala 中将 Map 转换为 Java 中的 Map

```scala
var map = Map("金箍棒" -> 10.0, "装甲" -> 20.0, "机枪" -> 30.0, "飞机" -> 40.0)
var javaMap = new java.util.TreeMap[String, Double];
map.foreach{
    case (key, value) => {
        javaMap.put(key, value)
    }
    case =>
}
```
## 11.4 List 集合的用法

```scala
//    创建一个 List 集合
var list = List(1, 2, 3, 4)

//    向集合中添加一个元素
var list2 = 0 :: list

//    将两个 List 合并
var list3 = list ::: list2

//    返回第一个元素
var head = list.head

//    返回除第一个元素的 List 集合
var tailList = list.tail

//    判断 List 是否为空
var isEmpty = list.isEmpty

//    取出 List 中的偶数
var evenList = list.filter(_ % 2 ==0)

//    过滤字符串中的数字
var str = "123 hello scala 168"
var str2 = str.filter(Character.isDigit(_))

//    取出一个字符之前的所有字符
var str3 = str.toList.takeWhile(_ != 's')

//    将每个元素进行转换映射
var str4 = str.map(_.toUpper)

//    取出集合中的偶数，并为每个元素加上 100
var list4 = list.filter(_ % 2 ==0).map(_ + 100)

//    两层 List 集合
var list5 = List(list, List(4, 5, 6))

//    取出 list5 中的所有偶数，第一步 : map 获取每个 list，第二步 : filter 过滤每个 list 中的偶数元素
var list6 = list5.map(_.filter(_ % 2 == 0))

//    将 List 中的结果打平
var list7 = list5.flatMap(_.filter(_ % 2 == 0))

//    reduceLeft 和 reduceRight
var list8 = List(1, 7, 2, 9)
var result = list8.reduceLeft(_ - _)    //    ((1 - 7) -2) - 9 = -17
var result2 = list8.reduceRight(_ - _)    //    1 - (7 - (2 - 9)) = -13

//    foldLeft 和 foldRight 注 : foldLeft 和 foldRight 的简写操作中 : 均位于集合一侧，集合的位置与相应方法的名字相反
var result3 = list8.foldLeft((0)(_ - _))    //    (((0 - 1) - 7) -2) - 9 = -19
var result4 = (0 /: list8)(_ - _)    //    foldLeft 简写操作
var result5 = list8.folderRight((0)(_ - _))    //    1 - (7 - (2 - (9 - 0))) = -13
var result6 = (list8 /: 0)(_ - _)    //    foldRight 简写操作

//    foldLeft 可以用于集合中的元素反转，foldRight 不可以
var list9 = List(1, 2, 4, 3, 5)
var list10 = (List[Int]() /: list9)((x, y) => y :: x)
var list11 = (list9 :\ List[Int]())((x, y) => x :: y)

//    返回列表中的元素以指定的分隔符分割的字符串
var str5 = list.mkString(",")

//    返回列表中移除指定元素后的列表
var list9 = list.remove(_ > 1)

//    返回逆序组成的新列表
var list10 = list.reverse

//    对 List 集合进行排序 (逆序排列)
var list11 = list10.sort((num1, num2) => {
    num1 > num2
})

//    对 List 集合中的元素分组，其中每一组仍然是一个 List 集合
val list12 = List(1, 2, 4, 3, 5, 6, 7, 8)
var list13 = arr.grouped(4).toList    //    4 是指每一组中有 4 个元素，最后一组可能少于 4 个元素
```
## 11.5 Scala 中 :: 、+: 、++ 、::: 的区别

**::**向队列的头部添加数据，创造新的列表，用法为 x :: list，其中 x 为加入到头部的元素
**+:**在头部追加元素，**:+**在尾部追加元素，冒号永远靠近集合类型
**++**连接两个集合，用法为 list1 ++ list2
**:::**只能用于连接两个 List 类型的集合

```scala
var list1 = List("a", "b", "c", "d")
var list2 = List("1", "2", "3", "4")

list1 :: list2 => List(List("a", "b", "c", "d"), "1", "2", "3", "4")
"e" +: list1 => List("e", "a", "b", "c", "d")
list1 :+ "e" => List("a", "b", "c", "d", "e")
list1 ++ list2 => List("a", "b", "c", "d", "1", "2", "3", "4")
list1 ::: list2 => List("a", "b", "c", "d", "1", "2", "3", "4")
```
## 11.6 Scala 中 _ 和 _* 的用法

**_ 的用法 :**
**作为函数的参数 :**当函数中的参数在 => 右侧只出现一次时，可以用 _ 代替这个参数

```scala
def compute(f : (Double) => Double) = {
    f(3)
}

//    传递一个匿名函数作为 compute 的参数
compute((x : Double) => 2 * x)

//    如果参数 x 在 => 右侧只出现一次，可以用 _ 代替这个参数
compute(2 * _)

//    其它常见的使用方式
(1 to 9).filter(_ % 2 == 0)
(1 to 3).map(_ * 3)
```
**作为通配符 :**

```scala
//    在 match 中的 case 语句中使用
def matchTest(num : Int) => {
    num match{
        case 1 => "One"
        case 2 => "Two"
        case _ => "Many"
    }
}

//    可以通过模式匹配获取元组中的元素，当不需要某个值的时候可以使用 _ 代替
var tup = (1, 11.14, "Fred")
var (first, second, _) = tup
```
**_* :**
**变长参数 :**

```scala
//    求和
def sum(nums : Int*) : Int = {
    var sum = 0
    for (num <- nums) {
        sum += num
    }
    sum
}

var sum1 = sum(1, 2, 3, 4, 5)
var sum2 = sum(1 to 5 : _*)    //    _* 将 1 to 5 转化为参数序列
```
**变量声明中的模式 :**

```scala
var arr = Array(1, 2, 3, 4, 5)

var Array(1, 2, _*) = arr
var Array(first, second, _*) = arr
```
## 11.7 Map 和 FlatMap 的区别

Map 是将一个数据按照一个函数进行相应的变换，返回的结果中的元素的类型可能不一致，FlatMap 是将 Map 后的数据进行压平，结果中数据为一个集合，集合中元素的类型都一样.

代码 :

```scala
object collection_t1 {

  def flatMap1(): Unit = {
    val li = List(1,2,3)
    val res = li.flatMap(x => x match {
      case 3 => List('a','b')
      case _ => List(x*2)
    })
    println(res)
  }

  def map1(): Unit = {
    val li = List(1,2,3)
    val res = li.map(x => x match {
      case 3 => List('a','b')
      case _ => x*2
    })
    println(res)
  }

  def main(args: Array[String]): Unit = {
    flatMap1()
    map1()
  }
}
```
结果 :

```txt
List(2, 4, a, b)
List(2, 4, List(a, b))
```
## 11.8 for 和 yield 的用法

for 的用法总结:

```scala
// 1、可以以变量<-表达式的形式提供多个生成器，用分号将它们隔开
for(i <- 1 to 3; j <- 1 to 3) {
    print((10 * i + j) + " ");
  }
 
outputs: 11 12 13 21 22 23 31 32 33

// 2、每个生成器都可以带一个守卫，以if开头的Boolean表达式
for(i <- 1 to 3; j <- 1 to 3 if i != j) {
    print((10 * i + j) + " ");
  }
 
outputs: 12 13 21 23 31 32

// 3、可以使用任意多的定义，引入可以在循环中使用的变量
for(i <- 1 to 3; from = 4 - i; j <- from to 3) {
    print((10 * i + j) + " ");
  }
 
outputs： 13 22 23 31 32 33
```

yield 的用法总结:
1、针对每一次 for 循环的迭代, yield 会产生一个值，被循环记录下来 (内部实现上，像是一个缓冲区).

2、当循环结束后, 会返回所有 yield 的值组成的集合.

3、返回集合的类型与被遍历的集合类型是一致的.

```scala
scala> val a = Array(1, 2, 3, 4, 5)
a: Array[Int] = Array(1, 2, 3, 4, 5)
 
scala> for (e <- a if e > 2) yield e
res1: Array[Int] = Array(3, 4, 5)
```
## 11.9 lazy 的用法

1、在类的初始化的时候可能某个变量初始化比较耗时，那么可以使用lazy，等真正使用到这个变量的时候再初始化

```scala
scala> class Person{
     |   val properties = {
     |     //模拟长时间的某种操作
     |     println("init")
     |     Thread.sleep(2000)
     |   }
     | }
defined class Person

scala> def main() {
     |   println("Start")
     | 
     |   val startTime = System.currentTimeMillis()
     |   val person = new Person
     |   val endTime = System.currentTimeMillis()
     | 
     |   println("End and take " + (endTime - startTime) + "ms")
     | 
     |   person.properties
     | }
main: ()Unit

scala> main
Start
init
End and take 2002ms
```
可以看到new了一个对象，需要大约2000ms，如果改成lazy变量

```scala
scala> class Person{
     |   lazy val properties = {
     |     //模拟长时间的某种操作
     |     println("init")
     |     Thread.sleep(2000)
     |   }
     | }
defined class Person

scala> main
Start
End and take 0ms
init
```
new操作几乎瞬间完成，而且在真正使用变量的时候才初始化的

2、构造顺序问题

```scala
trait Person{
  val name: String
  val nameLen = name.length
}

class Student extends Person{
  override val name: String = "Tom"
}
```
如果这时候在main函数中new一个Student，会报空指针异常
这是因为父类的constructor先与子类执行，那么在执行val nameLen = name.length的时候name还没有被赋值，所以才会导致空指针。
解决办法就是在nameLen前加lazy，延迟初始化

```scala
lazy val nameLen = name.length
```