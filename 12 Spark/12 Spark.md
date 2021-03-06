# 12 Spark
## 12.1 Spark Core
### 12.1.1 RDD
#### 12.1.1.1 RDD 概念

RDD (Resilient Distributed Dataset) 全称为分布式数据集，是整个Spark的计算基石。是分布式数据的抽象，为用户屏蔽了底层复杂的计算和映射环境，RDD 具有如下特点 :

1、RDD是不可变的，如果需要在一个RDD上进行转换操作，则会生成一个新的RDD

2、RDD是分区的，RDD里面的具体数据是分布在多台机器上的Executor里面的。堆内内存和堆外内存 + 磁盘。

3、RDD是弹性的，RDD 的弹性包括以下几个方面 :

a、存储：Spark会根据用户的配置或者当前Spark的应用运行情况去自动将RDD的数据缓存到内存或者磁盘。他是一个对用户不可见的封装的功能。
  			 
b、容错：当你的RDD数据被删除或者丢失的时候，可以通过血统或者检查点机制恢复数据。这个用户透明的。
  			
c、计算：计算是分层的，有应用->JOb->Stage->TaskSet-Task  每一层都有对应的计算的保障与重复机制。保障你的计算不会由于一些突发因素而终止。
  			
d、分片：你可以根据业务需求或者一些算子来重新调整RDD中的数据分布。
  			 
#### 12.1.1.2 Spark Core 操作 RDD 的过程

![](../img/_1536544352_22043.png)

RDD的创建--》RDD的转换--》RDD的缓存--》RDD的行动--》RDD的输出。
#### 12.1.1.3 WordCount 解析

![](../img/_1536544298_5636.png)

#### 12.1.1.4 转换 RDD
#### 12.1.1.4.1 map

```scala
def map[U: ClassTag](f: T => U): RDD[U] : 将函数应用于RDD的每一元素，并返回一个新的RDD
```
![](../img/_1536567837_20934.png)

```scala
 val mapadd = source.map(_ * 2)
```
#### 12.1.1.4.2 filter

```scala
def filter(f: T => Boolean): RDD[T] : 通过提供的产生boolean条件的表达式来返回符合结果为True新的RDD
```
![](../img/_1536567972_7974.png)

```scala
 val filter = sourceFilter.filter(_.contains("xiao"))
```
#### 12.1.1.4.3 flatmap

```scala
def flatMap[U: ClassTag](f: T => TraversableOnce[U]): RDD[U] : 将函数应用于RDD中的每一项，对于每一项都产生一个集合，并将集合中的元素压扁成一个集合。
```
![](../img/_1536568077_22862.png)

```scala
 val flatMap = sourceFlat.flatMap(1 to _)
```
#### 12.1.1.4.4 mapPartitions

```scala
def mapPartitions[U: ClassTag]( f: Iterator[T] => Iterator[U], preservesPartitioning: Boolean = false): RDD[U] : 将函数应用于RDD的每一个分区，每一个分区运行一次，函数需要能够接受Iterator类型，然后返回Iterator。
```
![](../img/_1536568175_22955.png)

```scala
def partitionsFun(iter : Iterator[(String,String)]) : Iterator[String] = {
  var woman = List[String]()
  while (iter.hasNext){
    val next = iter.next()
    next match {
       case (_,"female") => woman = next._1 :: woman
       case _ =>
    }
  }
  woman.iterator
}
```
```scala
 val result = rdd.mapPartitions(partitionsFun)
```
#### 12.1.1.4.5 mapPartitionsWithIndex

```scala
def mapPartitionsWithIndex[U: ClassTag]( f: (Int, Iterator[T]) => Iterator[U], preservesPartitioning: Boolean = false): RDD[U] : 将函数应用于RDD中的每一个分区，每一个分区运行一次，函数能够接受 一个分区的索引值 和一个代表分区内所有数据的Iterator类型，需要返回Iterator类型。
```
![](../img/_1536568334_19210.png)

```scala
def partitionsFun(index : Int, iter : Iterator[(String,String)]) : Iterator[String] = {
  var woman = List[String]()
  while (iter.hasNext){
    val next = iter.next()
    next match {
       case (_,"female") => woman = "["+index+"]"+next._1 :: woman
       case _ =>
    }
  }
  woman.iterator
}
```
```scala
val result = rdd.mapPartitionsWithIndex(partitionsFun)
```
#### 12.1.1.4.6 sample

```scala
def sample(withReplacement: Boolean, fraction: Double, seed: Long = Utils.random.nextLong): RDD[T] : 在RDD中以seed为种子返回大致上有fraction比例个数据样本RDD，withReplacement表示是否采用放回式抽样。
```
![](../img/_1536568446_25271.png)

```scala
 var sample2 = rdd.sample(false,0.2,3)
```
#### 12.1.1.4.7 union

```scala
def union(other: RDD[T]): RDD[T] : 将两个RDD中的元素进行合并，返回一个新的RDD
```
![](../img/_1536568545_20160.png)

```scala
 val rdd2 = sc.parallelize(5 to 10)
 val rdd3 = rdd1.union(rdd2)
```
#### 12.1.1.4.8 intersaction

```scala
def intersection(other: RDD[T]): RDD[T] : 将两个RDD做交集，返回一个新的RDD
```
![](../img/_1536568705_12126.png)

```scala
val rdd1 = sc.parallelize(1 to 7)
 val rdd2 = sc.parallelize(5 to 10)
  val rdd3 = rdd1.intersection(rdd2)
```
#### 12.1.1.4.9 distinct

```scala
def distinct(): RDD[T] : 将当前RDD进行去重后，返回一个新的RDD
```
![](../img/_1536568810_12199.png)

```scala
 val unionRDD = distinctRdd.distinct()
```
#### 12.1.1.4.10 partitionBy

```scala
def partitionBy(partitioner: Partitioner): RDD[(K, V)] : 根据设置的分区器重新将RDD进行分区，返回新的RDD。
```
![](../img/_1536568954_15470.png)

```scala
 var rdd2 = rdd.partitionBy(new org.apache.spark.HashPartitioner(2))
```
#### 12.1.1.4.11 reduceByKey

```scala
def reduceByKey(func: (V, V) => V): RDD[(K, V)] : 根据Key值将相同Key的元组的值用func进行计算，返回新的RDD
```
![](../img/_1536569053_16614.png)

```scala
 val reduce = rdd.reduceByKey((x,y) => x+y)
```
#### 12.1.1.4.12 groupByKey

```scala
def groupByKey(): RDD[(K, Iterable[V])] : 将相同Key的值进行聚集，输出一个(K, Iterable[V])类型的RDD
```
![](../img/_1536569128_30729.png)

```scala
 val group = wordPairsRDD.groupByKey()
```
#### 12.1.1.4.13 combineByKey

```scala
def combineByKey[C](createCombiner: V => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) => C, numPartitions: Int): RDD[(K, C)] : 根据key分别使用CreateCombiner和mergeValue进行相同key的数值聚集，通过mergeCombiners将各个分区最终的结果进行聚集。
```
![](../img/_1536569213_30255.png)

```scala
 val combine = input.combineByKey(
     |     (v)=>(v,1),
     |     (acc:(Int,Int),v)=>(acc._1+v,acc._2+1),
     |     (acc1:(Int,Int),acc2:(Int,Int))=>(acc1._1+acc2._1,acc1._2+acc2._2))
```
#### 12.1.1.4.14 aggregateByKey

```scala
def aggregateByKey[U: ClassTag](zeroValue: U, partitioner: Partitioner)(seqOp: (U, V) => U,
    combOp: (U, U) => U): RDD[(K, U)] : 通过seqOp函数将每一个分区里面的数据和初始值迭代带入函数返回最终值，comOp将每一个分区返回的最终值根据key进行合并操作。
```
![](../img/_1536569337_32125.png)

```scala
 val agg = rdd.aggregateByKey(0)(math.max(_,_),_+_)
```
#### 12.1.1.4.15 foldByKey

```scala
def foldByKey(zeroValue: V, partitioner: Partitioner)(func: (V, V) => V): RDD[(K, V)] : aggregateByKey的简化操作，seqop和combop相同
```
```scala
 val agg = rdd.foldByKey(0)(_+_)
```
#### 12.1.1.4.16 sortByKey

```scala
def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length) : RDD[(K, V)]   在一个(K,V)的RDD上调用，K必须实现Ordered接口，返回一个按照key进行排序的(K,V)的RDD
```
```scala
rdd.sortByKey(true).collect()
```
#### 12.1.1.4.17 sortBy

```scala
def sortBy[K](f: (T) => K, ascending: Boolean = true, numPartitions: Int = this.partitions.length) (implicit ord: Ordering[K], ctag: ClassTag[K]): RDD[T] : 底层实现还是使用sortByKey，只不过使用fun生成的新key进行排序。
```
```scala
 rdd.sortBy(x => x%3).collect()
```
#### 12.1.1.4.18 join

```scala
def join[W](other: RDD[(K, W)], partitioner: Partitioner): RDD[(K, (V, W))] : 在类型为(K,V)和(K,W)的RDD上调用，返回一个相同key对应的所有元素对在一起的(K,(V,W))的RDD，但是需要注意的是，他只会返回key在两个RDD中都存在的情况。
```
```scala
 rdd.join(rdd1).collect()
```
#### 12.1.1.4.19 cogroup

```scala
def cogroup[W](other: RDD[(K, W)], partitioner: Partitioner) : RDD[(K, (Iterable[V], Iterable[W]))] : 在类型为(K,V)和(K,W)的RDD上调用，返回一个(K,(Iterable<V>,Iterable<W>))类型的RDD，注意，如果V和W的类型相同，也不放在一块，还是单独存放。
```
![](../img/_1536576475_23814.png)

```scala
 rdd.cogroup(rdd2).collect()
```
#### 12.1.1.4.20 cartesian

```scala
def cartesian[U: ClassTag](other: RDD[U]): RDD[(T, U)] : 做两个RDD的笛卡尔积，返回对偶的RDD
```
![](../img/_1536576592_11776.png)

```scala
 rdd1.cartesian(rdd2).collect()
```
#### 12.1.1.4.21 pipe

```scala
def pipe(command: String): RDD[String] : 对于每个分区，都执行一个perl或者shell脚本，返回输出的RDD，注意，如果你是本地文件系统中，需要将脚本放置到每个节点上。
```
```shell
# Shell脚本
#!/bin/sh
echo "AA"
while read LINE; do
   echo ">>>"${LINE}
done
```
```scala
 rdd.pipe("/home/bigdata/pipe.sh").collect()
```
#### 12.1.1.4.22 coalesce

```scala
def coalesce(numPartitions: Int, shuffle: Boolean = false, partitionCoalescer: Option[PartitionCoalescer] = Option.empty) (implicit ord: Ordering[T] = null) : RDD[T] : 缩减分区数，用于大数据集过滤后，提高小数据集的执行效率。
```
```scala
val coalesceRDD = rdd.coalesce(3)
```
#### 12.1.1.4.23 repartition

```scala
def repartition(numPartitions: Int)(implicit ord: Ordering[T] = null): RDD[T] : 根据你传入的分区数重新通过网络分区所有数据，重型操作。
```
```scala
 val rerdd = rdd.repartition(2)
```
#### 12.1.1.4.24 repartitionsAndSortWithinPartitions

```scala
def repartitionAndSortWithinPartitions(partitioner: Partitioner): RDD[(K, V)] : 性能要比repartition要高。在给定的partitioner内部进行排序
```
#### 12.1.1.4.25 glom

```scala
def glom(): RDD[Array[T]] : 将每一个分区形成一个数组，形成新的RDD类型时RDD[Array[T]]
```
![](../img/_1536577096_23594.png)

```scala
 rdd.glom().collect()
```
#### 12.1.1.4.26 mapValues

```scala
def mapValues[U](f: V => U): RDD[(K, U)] : 将函数应用于（k，v）结果中的v，返回新的RDD
```
![](../img/_1536577166_28001.png)

```scala
 rdd3.mapValues(_+"|||").collect()
```
#### 12.1.1.4.27 substract

```scala
def subtract(other: RDD[T]): RDD[T] : 计算差的一种函数去除两个RDD中相同的元素，不同的RDD将保留下来。
```
![](../img/_1536577275_13321.png)

```scala
rdd.subtract(rdd1).collect()
```
#### 12.1.1.5 行动 RDD
#### 12.1.1.5.1 takeSample

```scala
def takeSample( withReplacement: Boolean, num: Int, seed: Long = Utils.random.nextLong): Array[T] : 抽样但是返回一个scala集合。
```
```scala
 rdd1.takeSample(true, 5, 3)
```
#### 12.1.1.5.2 reduce

```scala
def reduce(f: (T, T) => T): T : 通过func函数聚集RDD中的所有元素
```
```scala
 rdd1.reduce(_+_)
```
#### 12.1.1.5.3 collect

```scala
def collect(): Array[T] : 在驱动程序中，以数组的形式返回数据集的所有元素
```
```scala
rdd1.collect()
```
#### 12.1.1.5.4 count

```scala
def count(): Long : 返回RDD中的元素个数
```
```scala
rdd.count()
```
#### 12.1.1.5.5 first

```scala
def first(): T : 返回RDD中的第一个元素
```
```scala
rdd.first()
```
#### 12.1.1.5.6 take

```scala
def take(num: Int): Array[T] : 返回RDD中的前n个元素
```
```scala
rdd.take(5)
```
#### 12.1.1.5.7 takeOrdered

```scala
def takeOrdered(num: Int)(implicit ord: Ordering[T]) : 返回前几个的排序
```
```scala
rdd1.takeOrdered(2)
```
#### 12.1.1.5.8 aggregate

```scala
def aggregate[U: ClassTag](zeroValue: U)(seqOp: (U, T) => U, combOp: (U, U) => U): U : aggregate函数将每个分区里面的元素通过seqOp和初始值进行聚合，然后用combine函数将每个分区的结果和初始值(zeroValue)进行combine操作。这个函数最终返回的类型不需要和RDD中元素类型一致。
```
![](../img/_1536579112_20566.png)

```scala
rdd1.aggregate(1)(
| {(x : Int, y: Int) => x * y},
| {(a : Int, b : Int) => a + b}
)
```
#### 12.1.1.5.9 fold

```scala
def fold(zeroValue: T)(op: (T, T) => T): T : 折叠操作，aggregate的简化操作，seqop和combop一样。
```
```scala
 rdd1.fold(1)(_+_)
```
#### 12.1.1.5.10 saveAsTextFile

```scala
def saveAsTextFile(path: String): Unit : 将RDD以文本文件的方式保存到本地或者HDFS中
```
```scala
rdd.saveAsTextFile("hdfs://hadoop1:8020/file/spark")
```
#### 12.1.1.5.11 saveAsSequenceFile

```scala
def saveAsSequenceFile(path: String): Unit : 将RDD中的元素以Hadoop sequencefile的格式保存到指定的目录下，可以是HDFS或者其他Hadoop支持的文件系统。
```
```scala
rdd.saveAsSequenceFile("hdfs://hadoop1:8020/file/spark")
```
#### 12.1.1.5.12 saveAsObjectFile

```scala
def saveAsObjectFile(path: String): Unit : 将RDD中的元素以序列化后对象形式保存到本地或者HDFS中。
```
```scala
rdd.saveAsObjectFile("hdfs://hadoop1:8020/file/spark")
```
#### 12.1.1.5.13 countByKey

```scala
def countByKey(): Map[K, Long] : 针对(K,V)类型的RDD，返回一个(K,Int)的map，表示每一个key对应的元素个数。
```
```scala
rdd.countByKey()
```
#### 12.1.1.5.14 foreach

```scala
def foreach(f: T => Unit): Unit : 在数据集的每一个元素上，运行函数func进行更新。
```
```scala
 var sum = sc.accumulator(0)
 rdd.foreach(sum+=_)
```
#### 12.1.1.6 RDD 需要注意的地方
当你在RDD中使用到了class的方法或者属性的时候，该class需要继承java.io.Serializable接口，或者可以将属性赋值为本地变量来防止整个对象的传输。
#### 12.1.1.7 RDD 的依赖关系

![](../img/_1536587927_19725.png)

1、RDD的依赖关系分为窄依赖和宽依赖。

2、窄依赖是说父RDD的每一个分区最多被一个子RDD的分区应用，也就是他的出度为1。

3、宽依赖是说父RDD的每一个分区被多个子RDD的分区来应用，也就是他的出度大于等于2.

4、应用在整个过程中，RDD之间形成的产生关系，就叫做血统关系，RDD在没有持久化的时候默认是不保存的，如果需要那么就要根据血统关系来重新计算。

5、应用在执行过程中，是分为多个Stage来进行的，划分Stage的关键就是判断是不是存在宽依赖。从Action往前去推整个Stage的划分。
#### 12.1.1.8 RDD 的持久化
1、RDD的持久化主要通过persist和cache操作来实现。cache操作就相当于StoageLevel为MEMORY_ONLY的persist操作。

2、持久化它的存储等级可以分为：存储的位置（磁盘、内存、非堆内存）、是否序列化、存储的份数（1,2）
#### 12.1.1.9 RDD 的检查点机制
#### 12.1.1.9.1 checkpoint 的使用

```scala
//  1、我需要先创建一个RDD
val data = sc.parallelize(1 to 100 , 5)

//  2、需要设置sparkContext他的checkpoint目录，如果你要用spark-shell,那么就是sc.setCheckpointDir("....")
sc.setCheckpointDir("hdfs://master01:9000/checkpoint")

//  3、在RDD上调用checkpoint方法
data.checkpoint
 
//  4、触发RDD的行动操作，让RDD的数据真实写入checkpoint目录。
ch2.collect
```
注意：整个checkpoint的读取时用户透明
#### 12.1.1.10 RDD 分区
Spark提供了RDD的分区操作，主要针对K-V结构，提供了诸如HashPartitioner和RangePartitioner

#### 12.1.1.10.1 自定义分区器

```scala
//  1、首先需要创建一个新的class继承Partitioner
//  2、复写抽象方法
//  3、通过partitionBy操作应用新的分区器。

class CustomerPartitioner(numParts:Int) extends Partitioner {

  //覆盖分区数
  override def numPartitions: Int = numParts

  //覆盖分区号获取函数
  override def getPartition(key: Any): Int = {
    val ckey: String = key.toString
    ckey.substring(ckey.length-1).toInt%numParts
  }
}

object CustomerPartitioner {
  def main(args: Array[String]) {
    val conf=new SparkConf().setAppName("partitioner")
    val sc=new SparkContext(conf)

    val data=sc.parallelize(List("aa.2","bb.2","cc.3","dd.3","ee.5"))
	
    data.map((_,1)).partitionBy(new CustomerPartitioner(5)).keys.saveAsTextFile("hdfs://master01:9000/partitioner")
  }
}
```
#### 12.1.1.11 RDD 累加器
#### 12.1.1.11.1 累加器的使用

```scala
//  1、首先需要通过sparkcontext去声明一个累加器，方法时accumulator，在声明的过程中需要提供累加器的初始值。
 val blanklines = sc.accumulator(0)
 
//  2、你可以在转换操作或者行动上直接使用累加器， 可以通过 += 操作符增加累加器的值，但是不能够读取累加器。
//  注意： 一般不推荐在转换操作使用累加器。一般推荐在行动操作中去使用。
 val tmp = notice.flatMap(line => {
     |    if (line == "") {
     |       blanklines += 1
     |    }
     |    line.split(" ")
     | })
     
//  3、 Driver可以通过 累加器.value 操作类读取累加器的值并输出。
 blanklines.value
```
#### 12.1.1.11.2 自定义累加器
1、继承AccumulatorV2这个虚拟类，然后提供类型参数 : 增加值类型参数、输出值类型参数。

2、需要创建一个SparkContext

3、需要创建一个自定义累加器实例

4、需要通过SparkContext去注册你的累加器，  sc.register(accum, "logAccum")

5、需要在转换或者行动操作中使用累加器。

6、在Driver中输出累加器的结果。

```scala
//    CustomerAccumulator.scala
class CustomerAccumulator extends AccumulatorV2[String, java.util.Set[String]]{

  //  定义一个累加器的内存结构，用来保存带有字母的字符串
  private val _stringArr = new util.HashSet[String]()

  //  该累加器内部数据是否为空
  override def isZero: Boolean = {
    _stringArr.isEmpty
  }

  //  让 Spark 框架能够调用 copy 函数产生一个新的累加器实例
  override def copy(): AccumulatorV2[String, util.Set[String]] = {
    val customerAccumulator = new CustomerAccumulator
    _stringArr.synchronized{
      customerAccumulator._stringArr.addAll(_stringArr)
    }
    customerAccumulator
  }

  //  重置该累加器数据结构
  override def reset(): Unit = {
    _stringArr.clear()
  }

  //  提供修改累加器的方法
  override def add(v: String): Unit = {
    _stringArr.add(v)
  }

  //  用于合并多个分区的累加器实例
  override def merge(other: AccumulatorV2[String, util.Set[String]]): Unit = {
    other match {
      case o : CustomerAccumulator => _stringArr.addAll(o.value)
    }
  }

  //  通过 value 方法输出累加器最终结果
  override def value: util.Set[String] = {
    java.util.Collections.unmodifiableSet(_stringArr)
  }

}
```
```scala
//    CunstomerAccumulatorTest.scala
object CunstomerAccumulatorTest {

  val config: SparkConf = new SparkConf().setMaster("local[4]").setAppName("CustomerAccumulatorTest")

  val acc: AccumulatorV2[String, java.util.Set[String]] = new CustomerAccumulator

  def main(args: Array[String]): Unit = {

    val sc = new SparkContext(config)

    //  注册自定义累加器
    sc.register(acc, "customerAccumulator")

    val sum = sc.makeRDD(List("1", "2a", "3", "4b", "5", "6", "7cd", "8", "9"), 2).filter(str => {
      val pattern = """^-?(\d+)"""
      //  判断当前字符串是否为纯数字
      val flag = str.matches(pattern)
      if (!flag) {
        //  当前字符串不是纯数字，将其添加到累加器中
        acc.add(str)
      }
      flag
    }).map(_.toInt).reduce(_ + _)

    println(s"sum = ${sum}")
    println(s"acc.value = ${acc.value}")

  }
}
```
#### 12.1.1.12 RDD 广播变量
1、广播变量的出现是为了解决只读大对象分发的问题。

2、如果不是广播变量，那么使用的变量会跟分区进行分发，效率比较低。
#### 12.1.1.12.1 广播变量的使用

```scala
//    1、通过SparkContext.broadcast(对象)  来声明一个广播变量。
 val broadcastVar = sc.broadcast(Array(1, 2, 3))
 
//    2、通过广播变量的变量名的value方法来获取广播变量的值。
 broadcastVar.value
```
#### 12.1.1.13 RDD 中较易混淆的点
#### 12.1.1.13.1 map、flatMap、mapPartitions

```java
/**
 * 方法一 : map
 * map 是将数据中的每一行经过 map 中定义的方法，其方法中参数的类型为每一行数据的类型
 */

JavaRDD<String[]> mapresult=lines.map(new Function<String, String[]>() {

    @Override
    public String[] call(String s) throws Exception {
        return s.split(":");
    }
});

/**
 * 方法二 : flatMap
 * 同 map 一样，将数据中的每一行经过 flatmap 中的方法，不过一般情况下当对数据处理可能出现集合，但结果数据由不需要集合时，可以使用 flatMap
 */
JavaRDD<String> objectJavaRDD = lines.flatMap(new FlatMapFunction<String, String>() {

    @Override
    public Iterable<String> call(String s) throws Exception {
        return Arrays.asList(s.split(" "));
    }
});

/**
 * 方法三 : mapPartitions
 * mapPartitions 是对每个分区中的数据进行处理，其函数中的参数为一个迭代器，返回值同样是一个迭代器，当对分区中的数据处理完之后，再使用 flatMap 将结果压平，最终结果的类型和 flatMap 返回结果的类型相同
 *
 */
lines2.mapPartitions(new FlatMapFunction<Iterator<String>, String>() {
    ArrayList<String> results = new ArrayList<String>();

    @Override
    public Iterable<String> call(Iterator<String> s) throws Exception {
        while (s.hasNext()) {
            results.addAll(Arrays.asList(s.next().split(":")));
        }
        return results;
    }
}).saveAsTextFile("/Users/luoluowushengmimi/Documents/result");
```

## 12.2 Spark SQL
### 12.2.1 特点
1、和Spark Core的无缝集成，我可以在写整个RDD应用的时候，配置Spark SQL来实现我的逻辑。

2、统一的数据访问方式，Spark SQL提供标准化的SQL查询。

3、Hive的继承，Spark SQL通过内嵌Hive或者连接外部已经部署好的hive实例，实现了对Hive语法的继承和操作。

4、标准化的连接方式，Spark SQL可以通过启动thrift Server来支持JDBC、ODBC的访问，将自己作为一个BI Server使用。
### 12.2.2 数据抽象
RDD（Spark1.0）-> DataFrame（Spark1.3）-> DataSet（Spark1.6）
#### 12.2.2.1 RDD
1、RDD是一个懒执行的不可变的可以支持Lambda表达式的并行数据集合。

2、RDD的最大好处就是简单，API的人性化程度很高。

3、RDD的劣势是性能限制，它是一个JVM驻内存对象，这也就决定了存在GC的限制和数据增加时Java序列化成本的升高。
#### 12.2.2.2 DataFrame

![](../img/_1536634961_18715.png)

1、DataFrame是为数据提供了Schema的视图。可以把它当做数据库中的一张表来对待

2、DataFrame也是懒执行的。

3、性能上比RDD要高，主要有两方面原因：

a. 定制化内存管理：数据以二进制的方式存在于非堆内存，节省了大量空间之外，还摆脱了GC的限制。

b. 优化的执行计划：查询计划通过Spark catalyst optimiser进行优化.

4、Dataframe的劣势在于在编译期缺少类型安全检查，导致运行时出错.
#### 12.2.2.3 DataSet
1、是Dataframe API的一个扩展，是Spark最新的数据抽象

2、用户友好的API风格，既具有类型安全检查也具有Dataframe的查询优化特性。

3、Dataset支持编解码器，当需要访问非堆上的数据时可以避免反序列化整个对象，提高了效率。

4、样例类被用来在Dataset中定义数据的结构信息，样例类中每个属性的名称直接映射到DataSet中的字段名称。

5、Dataframe是Dataset的特例，DataFrame=Dataset[Row] ，所以可以通过as方法将Dataframe转换为Dataset。Row是一个类型，跟Car、Person这些的类型一样，所有的表结构信息我都用Row来表示。

6、DataSet是强类型的。比如可以有Dataset[Car]，Dataset[Person].
### 12.2.3 DataFrame 查询方式
DataFrame支持两种查询方式一种是DSL风格，另外一种是SQL风格
#### 12.2.3.1 DSL 风格
需要引入  import spark.implicit._  这个隐式转换，可以将DataFrame隐式转换成RDD。

```scala
import spark.implicits._
    
val df = spark.read.json("/file/spark/people.json")
    
df.show()
    
df.filter($"age" > 21).show()
```
#### 12.2.3.2 SQL 风格

1、你需要将DataFrame注册成一张表格，如果你通过CreateTempView这种方式来创建，那么该表格Session有效，如果你通过CreateGlobalTempView来创建，那么该表格跨Session有效，但是SQL语句访问该表格的时候需要加上前缀  global_temp

 2、你需要通过sparkSession.sql 方法来运行你的SQL语句。

```scala
val df = spark.read.json("/file/spark/people.json")

df.createOrReplaceTempView("people")

spark.sql("select * from people where age > 21").show()
```
### 12.2.4 RDD、DataFrame、DataSet 之间的转换

![](../img/_1536678435_17914.png)

#### 12.2.4.1 RDD、DataFrame 之间的转换
RDD 转换为 DataFrame

```scala
var peopleDF = rdd.map(para=>(para(0).trim(),para(1).trim().toInt)).toDF("name","age")

// 通过反射
b. var peopleDF = rdd.map(para=>Person(para(0).trim(),para(1).trim().toInt)).toDF
// 通过编程设置
import org.apache.spark.sql.types._
var schemaString = “name age”
var fields = schemaString._split(“ ”).map(fieldName = > StructField(fieldName, StringType, nullable = true))
var schema = StructType(fields)
var rowRDD = peopleRDD.map(_.split(“,”)).map(para => Row(para(0).trim(), pra(1).trim()))
val peopleDF = spark.createDataFrame(rowRDD, schema)

DataFrame = RDD + Schema
```
DataFrame 转换为 RDD

```scala
//    注意输出：Array([Michael,29], [Andy,30], [Justin,19])
dataFrame.rdd
```
#### 12.2.4.2 RDD、DataSet 之间的转换
RDD 转换为 DataSet

```scala
//    定义一个 Case 类
case class Person(name : String, age : Int)

rdd.map(para=> Person(para(0).trim(),para(1).trim().toInt)).toDS
```
DataSet 转换为 RDD

```scala
//    注意输出： Array(Person(Michael,29), Person(Andy,30), Person(Justin,19))
dataSet.rdd
```
#### 12.2.4.3 DataFrame、DataSet 之间的转换
DataFrame 转换为 DataSet

```scala
dataFrame.as[Person]
```
DataSet 转换为 DataFrame

```scala
dataSet.toDF
```
### 12.2.5 用户自定义函数
#### 12.2.5.1 用户自定义 UDF 函数

```scala
// 1、通过spark.udf.register(name,func)来注册一个UDF函数，name是UDF调用时的标识符，func是一个函数，用于处理字段。
spark.udf.register("addName", (x : String) => "Name : " + x)

// 2、你需要将一个DF或者DS注册为一个临时表。
peopleDF.createOrReplaceTempView("people")

// 3、通过spark.sql去运行一个SQL语句，在SQL语句中可以通过 name(列名) 方式来应用UDF函数。
spark.sql("select addName(name) as newName, age from people").show
```
#### 12.2.5.2 用户自定义聚合函数

![](../img/_1536725709_2017.png)

#### 12.2.5.2.1 弱类型用户自定义聚合函数

1、新建一个Class 继承UserDefinedAggregateFunction  ，然后复写方法：

2、你需要通过spark.udf.resigter去注册你的UDAF函数。

3、需要通过spark.sql去运行你的SQL语句，可以通过 select UDAF(列名) 来应用你的用户自定义聚合函数。

```scala
// MyAverage.scala

object MyAverage extends UserDefinedAggregateFunction{

  //  聚合函数输入参数的数据类型
  override def inputSchema: StructType = StructType(StructField("inputColumn", LongType) :: Nil)

  //  保存聚合函数业务逻辑数据的一个数据结构
  override def bufferSchema: StructType = StructType(StructField("sum", LongType) :: StructField("count", LongType) :: Nil)

  //  返回值的数据类型
  override def dataType: DataType = LongType

  //  对于相同的输入一直有相同的输出
  override def deterministic: Boolean = true

  //  初始化数据结构
  override def initialize(buffer: MutableAggregationBuffer): Unit = {
    //  存工资的总额
    buffer(0) = 0L
    //  存员工的个数
    buffer(1) = 0L
  }

  //  同分区内 Row 对聚合函数的更新操作
  override def update(buffer: MutableAggregationBuffer, input: Row): Unit = {
    //  判断输入的数据是否为空
    if (!input.isNullAt(0)) {
      //  输入的数据不为空，将工资进行累加，将员工人数加 1
      buffer(0) = buffer.getLong(0) + input.getLong(0)
      buffer(1) = buffer.getLong(1) + 1
    }
  }

  //  不同分区对聚合结果的聚合
  override def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
    buffer1(0) = buffer1.getLong(0) + buffer2.getLong(0)
    buffer1(1) = buffer1.getLong(1) + buffer2.getLong(1)
  }

  //  计算最终结果
  override def evaluate(buffer: Row): Any = buffer.getLong(0) / buffer.getLong(1)

}
```
```scala
// MyAverageTest.scala

object MyAverageTest {

  def main(args: Array[String]): Unit = {

    //  创建 SparkSession
    val spark: SparkSession = SparkSession.builder().appName("MyAverageTest").master("local[4]").getOrCreate()

    //  注册函数
    spark.udf.register("myAverage", MyAverage)
    //  读取数据
    val df: DataFrame = spark.read.json("hdfs://hadoop1:8020/file/spark/employees.json")
    //  创建临时表
    df.createOrReplaceTempView("employees")
    //  显示数据
    df.show()
    //  查询数据
    val result: DataFrame = spark.sql("select myAverage(salary) as average_salary from employees")
    //  显示数据
    result.show()

  }
}
```
#### 12.2.5.2.2 强类型用户自定义聚合函数

1、新建一个class，继承Aggregator[Employee, Average, Double]，其中Employee是在应用聚合函数的时候传入的对象，Average是聚合函数在运行的时候内部需要的数据结构，Double是聚合函数最终需要输出的类型。这些可以根据自己的业务需求去调整。复写相对应的方法：

2、新建一个UDAF实例，通过DF或者DS的DSL风格语法去应用。

```scala
// MyAverage.scala

case class Employee(name : String, salary : Long)
case class Average(var sum : Long, var count : Long)

object MyAverage2 extends Aggregator[Employee, Average, Double]{

  //  用于定义一个聚合函数内部需要的数据结构
  override def zero: Average = Average(0L, 0L)

  //  针对每个分区内部每一个输入更新数据结构
  override def reduce(b: Average, a: Employee): Average = {
    b.sum += a.salary
    b.count += 1
    b
  }

  //  用于对于不同分区的结构进行聚合
  override def merge(b1: Average, b2: Average): Average = {
    b1.sum += b2.sum
    b1.count += b2.count
    b1
  }

  //  计算输出
  override def finish(reduction: Average): Double = reduction.sum.toDouble / reduction.count

  //  用于数据结构的转换
  override def bufferEncoder: Encoder[Average] = Encoders.product

  //  用于最终结果的转换
  override def outputEncoder: Encoder[Double] = Encoders.scalaDouble

}
```
```scala
object MyAverageTest2 {
// MyAverageTest2.scala

  def main(args: Array[String]): Unit = {

    //  创建 SparkSession 对象
    val spark: SparkSession = SparkSession.builder().appName("MyAverageTest2").master("local[4]").getOrCreate()
    import spark.implicits._
    //  读取数据
    val ds: Dataset[Employee] = spark.read.json("hdfs://hadoop1:8020/file/spark/employees.json").as[Employee]
    //  显示数据
    ds.show()
    //  创建 MyAverage2 对象，并将其转换为列
    val averageSalary: TypedColumn[Employee, Double] = MyAverage2.toColumn.name("average_salary")
    //  查询数据
    val result: Dataset[Double] = ds.select(averageSalary)
    //  显示数据
    result.show()

  }
}
```
### 12.2.6 输入输出
#### 12.2.6.1 输入

对于Spark SQL的输入需要使用  sparkSession.read方法

1、通用模式   sparkSession.read.format("json").load("path")   支持类型：parquet、json、text、csv、orc、jdbc

2、专业模式   sparkSession.read.json、 csv  直接指定类型。

#### 12.2.6.2 输出

对于Spark SQL的输出需要使用  sparkSession.write方法

1、通用模式   dataFrame.write.format("json").save("path")  支持类型：parquet、json、text、csv、orc

2、专业模式   dataFrame.write.csv("path")  直接指定类型

#### 12.2.6.2 需要注意的地方

1、如果你使用通用模式，spark默认parquet是默认格式，sparkSession.read.load 他加载的默认是parquet格式。dataFrame.write.save也是默认保存成parquet格式。

2、如果需要保存成一个text文件，那么需要dataFrame里面只有一列。

### 12.2.7 Spark SQL 与第三方工具的集成
#### 12.2.7.1 Spark SQL 与 Hive 的集成
#### 12.2.7.1.1 内置 Hive

1、需要将core-site.xml和hdfs-site.xml 拷贝到spark的conf目录下。如果Spark路径下发现metastore_db，需要删除【仅第一次启动的时候】。

2、在你第一次启动创建metastore的时候，你需要指定spark.sql.warehouse.dir这个参数，比如：bin/spark-shell --conf spark.sql.warehouse.dir=hdfs://master01:9000/spark_warehouse

3、注意，如果你在load数据的时候，需要将数据放到HDFS上。

#### 12.2.7.1.2 外部 Hive

1、需要将hive-site.xml 拷贝到spark的conf目录下。

2、如果hive的metestore使用的是mysql数据库，那么需要将mysql的jdbc驱动包放到spark的jars目录下。

3、可以通过spark-sql或者spark-shell来进行sql的查询。完成和hive的连接。

### 12.2.8 Spark SQL 操作其它数据库
#### 12.2.8.1 MongoDB
#### 12.2.8.1.1 写入数据

```scala
//  将当前数据写入到 MongoDB
movieDF
   .write
   .option("uri", "mongodb://hadoop4:27017/recommender")
   .option("collection", "Movie")
   .mode("overwrite")
   .format("com.mongodb.spark.sql")
   .save()
```
#### 12.2.8.2 ElasticSearch
#### 12.2.8.2.1 写入数据

```scala
//  新建一个配置
val settings: Settings = Settings.builder().put("cluster.name", "es-cluster").build()

//  新建一个 ES 的客户端
val esClient = new PreBuiltTransportClient(settings)

//  需要将 TransportHosts 添加到 esClient 中
val REGEX_HOST_PORT = "(.+):(\\d+)".r
eSConfig.transportHosts.split(",").foreach {
  case REGEX_HOST_PORT(host: String, port: String) => {
    esClient.addTransportAddress(new InetSocketTransportAddress(InetAddress.getByName(host), port.toInt))
  }
}

//  需要清除掉 ES 中遗留的数据
if (esClient.admin().indices().exists(new IndicesExistsRequest("recommender")).actionGet().isExists) {
  esClient.admin().indices().delete(new DeleteIndexRequest("recommender"))
}
esClient.admin().indices().create(new CreateIndexRequest("recommender"))

//  将数据写入到 ES 中
movieDF
  .write
  .option("es.nodes", "hadoop4:9200")
  .option("es.http.timeout", "100m")
  .option("es.mapping.id", "mid")
  .mode("overwrite")
  .format("com.elasticsearch.spark.sql")
  .save("recommender/Movie")
```

## 12.3 Spark Streaming
### 12.3.1 原理

1、SPark Streaming是Spark中一个组件，基于Spark Core进行构建，用于对流式进行处理，类似于Storm。

2、Spark Streaming能够和Spark Core、Spark SQL来进行混合编程。

3、Spark Streaming我们主要关注：

a、Spark Streaming 能接受什么数据？ kafka、flume、HDFS、Twitter等。

b、Spark Streaming 能怎么处理数据？ 无状态的转换（前面处理的数据和后面处理的数据没啥关系）、有转换转换（前面处理的数据和后面处理的数据是有关系的，比如叠加关系）

### 12.3.2 架构

![](../img/_1536891105_25080.png)

![](../img/_1536891123_1279.png)

1、Spark Streaming 采用“微批次”架构。

2、对于整个流式计算来说，数据流你可以想象成水流，微批次架构的意思就是将水流按照用户设定的时间间隔分割为多个水流段。一个段的水会在Spark中转换成为一个RDD，所以对水流的操作也就是对这些分割后的RDD进行单独的操作。每一个RDD的操作都可以认为是一个小的批处理（也就是离线处理）。

### 12.3.3 DStream
#### 12.3.3.1 原理

![](../img/_1536891242_24036.png)

![](../img/_1536891276_32052.png)

1、DStream是类似于RDD和DataFrame的针对流式计算的抽象类。在源码中DStream是通过HashMap来保存他所管理的数据流的。K是RDD中数据流的时间，V是包含数据流的RDD。

2、对于DStream的操作也就是对于DStream他所包含的所有以时间序列排序的RDD的操作。

### 12.3.4 输入 
#### 12.3.4.1 文件数据源

1、Spark Streaming通过streamingContext.fileStream[KeyClass, ValueClass, InputFormatClass](dataDirectory) 这个方法提供了对目录下文件数据源的支持。

2、如果你的文件是比较简单的文本文件，你可以使用 streamingContext.textFileStream(dataDirectory)  来代替。

3、文件数据源目前不支持嵌套目录：

a、 文件需要有相同的数据格式

b、文件进入 dataDirectory的方式需要通过移动或者重命名来实现

c、一旦文件移动进目录，则不能再修改，即便修改了也不会读取新数据。

#### 12.3.4.2 自定义 Receiver

1、你需要新建一个Class去继承Receiver，并给Receiver传入一个类型参数，该类型参数是你需要接收的数据的类型。

2、你需要去复写Receiver的方法： onStart方法（在Receiver启动的时候调用的方法）、onStop方法（在Receiver正常停止的情况下调用的方法）

3、你可以在程序中通过streamingContext.receiverStream( new CustomeReceiver)来调用你定制化的Receiver。

```scala
//  CustomReceiver.scala

//  Receiver 需要提供一个类型参数，该类型参数是 Receiver 接收到的数据的类型
class CustomReceiver(host: String, port : Integer) extends Receiver[String](StorageLevel.MEMORY_AND_DISK){

  //  Receiver 启动的时候需要调用的方法
  override def onStart(): Unit = {
    //  定义一个新的线程去运行 receive 方法
    new Thread("customThread") {
      override def run(): Unit = {receive()}
    }.start()
  }

  //  receive 方法
  def receive() : Unit = {
    //  新建一个 Socket 连接
    val socket: Socket = new Socket(host, port)
    //  获取 Socket 的输入
    val reader: BufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream, StandardCharsets.UTF_8))
    var lines = ""
    //  如果 Receiver 没有停止并且读取到的数据不为 null
    while (!isStopped() && (lines = reader.readLine()) != null) {
      //  将读取到的 lines 提交给 Spark 框架
      store(lines)
    }
    //  释放资源
    reader.close()
    socket.close()
  }

  //  Receiver 停止的时候需要调用的方法
  override def onStop(): Unit = {
  }

}
```
```scala
// WordCount3.scala

object WordCount3 {

  //  创建 SparkConf 对象
  var conf: SparkConf = new SparkConf().setAppName("WordCount2").setMaster("local[4]")

  def main(args: Array[String]): Unit = {

    //  创建 StreamingContext 对象
    val ssc: StreamingContext = new StreamingContext(conf, Seconds(1))
    //  监控数据变化并处理数据
    val result: DStream[(String, Int)] = ssc.receiverStream(new CustomReceiver("hadoop1", 9999)).flatMap(_.split(" ")).map((_, 1)).reduceByKey(_ + _)
    //  打印结果
    result.print()
    //  启动流式处理程序
    ssc.start()
    //  等待停止信号
    ssc.awaitTermination()

  }
```
#### 12.3.4.3 RDD 数据源

可以通过StreamingContext.queueStream(rddQueue)这个方法来监控一个RDD的队列，所有加入到这个RDD队列中的新的RDD，都会被Streaming去处理。

```scala
object QueueRdd {

  def main(args: Array[String]) {

    val conf = new SparkConf().setMaster("local[2]").setAppName("QueueRdd")
    val ssc = new StreamingContext(conf, Seconds(1))

    // Create the queue through which RDDs can be pushed to
    // a QueueInputDStream
    //创建RDD队列
    val rddQueue = new mutable.SynchronizedQueue[RDD[Int]]()

    // Create the QueueInputDStream and use it do some processing
    // 创建QueueInputDStream
    val inputStream = ssc.queueStream(rddQueue)

    //处理队列中的RDD数据
    val mappedStream = inputStream.map(x => (x % 10, 1))
    val reducedStream = mappedStream.reduceByKey(_ + _)

    //打印结果
    reducedStream.print()

    //启动计算
    ssc.start()

    // Create and push some RDDs into
    for (i <- 1 to 30) {
      rddQueue += ssc.sparkContext.makeRDD(1 to 300, 10)
      Thread.sleep(2000)

      //通过程序停止StreamingContext的运行
      //ssc.stop()
    }
  }
}
```
#### 12.3.4.4 Spark Streaming 和 Kafka 的集成

![](../img/_1536936816_4474.png)

#### 12.3.4.4.1 BaseKafkaProducerFactory.scala

```scala
class BaseKafkaProducerFactory(brokerList: String,
                                  config: Properties = new Properties,
                                  defaultTopic: Option[String] = None)
  extends KafkaProducerFactory(brokerList, config, defaultTopic) {

  override def newInstance() = new KafkaProducerProxy(brokerList, config, defaultTopic)

}
```
#### 12.3.4.4.2 KafkaProducerFactory.scala

```scala
abstract class KafkaProducerFactory(brokerList: String, config: Properties, topic: Option[String] = None) extends Serializable {
  def newInstance(): KafkaProducerProxy
}
```
#### 12.3.4.4.3 KafkaProducerPool.scala

```scala
//单例对象
object KafkaProducerPool{

  //用于返回真正的对象池GenericObjectPool
  def apply(brokerList: String, topic: String):  GenericObjectPool[KafkaProducerProxy] = {
    val producerFactory = new BaseKafkaProducerFactory(brokerList, defaultTopic = Option(topic))
    val pooledProducerFactory = new PooledKafkaProducerAppFactory(producerFactory)
    //指定了你的kafka对象池的大小
    val poolConfig = {
      val c = new GenericObjectPoolConfig
      val maxNumProducers = 10
      c.setMaxTotal(maxNumProducers)
      c.setMaxIdle(maxNumProducers)
      c
    }
    //返回一个对象池
    new GenericObjectPool[KafkaProducerProxy](pooledProducerFactory, poolConfig)
  }
}
```
#### 12.3.4.4.4 KafkaStreaming.scala

```scala
object KafkaStreaming{

  def main(args: Array[String]) {

    //设置sparkconf
    val conf = new SparkConf().setMaster("local[4]").setAppName("NetworkWordCount")
    //新建了streamingContext
    val ssc = new StreamingContext(conf, Seconds(1))

    //kafka的地址
    val brobrokers = "hadoop1:9092,hadoop2:9092,hadoop3:9092"
    //kafka的队列名称
    val sourcetopic="source";
    //kafka的队列名称
    val targettopic="target";

    //创建消费者组名
    var group="con-consumer-group"

    //kafka消费者配置
    val kafkaParam = Map(
      "bootstrap.servers" -> brobrokers,//用于初始化链接到集群的地址
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      //用于标识这个消费者属于哪个消费团体
      "group.id" -> group,
      //如果没有初始化偏移量或者当前的偏移量不存在任何服务器上，可以使用这个配置属性
      //可以使用这个配置，latest自动重置偏移量为最新的偏移量
      "auto.offset.reset" -> "latest",
      //如果是true，则这个消费者的偏移量会在后台自动提交
      "enable.auto.commit" -> (false: java.lang.Boolean)
    );

    //创建DStream，返回接收到的输入数据
    var stream=KafkaUtils.createDirectStream[String,String](ssc, LocationStrategies.PreferConsistent,ConsumerStrategies.Subscribe[String,String](Array(sourcetopic),kafkaParam))

    //每一个stream都是一个ConsumerRecord
    stream.flatMap(_.value().split(" ")).map((_, 1)).reduceByKey(_ + _).foreachRDD(rdd => {
      //对于RDD的每一个分区执行一个操作
      rdd.foreachPartition(partitionOfRecords => {
        // kafka连接池。
        val pool = KafkaProducerPool(brobrokers, targettopic)
        //从连接池里面取出了一个Kafka的连接
        val p = pool.borrowObject()
        //发送当前分区里面每一个数据
        partitionOfRecords.foreach {message => System.out.println(message);p.send(String.valueOf(message),Option(targettopic))}

        // 使用完了需要将kafka还回去
        pool.returnObject(p)

      })
    })

    ssc.start()
    ssc.awaitTermination()

  }
}
```
#### 12.3.4.4.5 PooledKafkaProducerAppFactory.scala

```scala
case class KafkaProducerProxy(brokerList: String,
                            producerConfig: Properties = new Properties,
                            defaultTopic: Option[String] = None,
                            producer: Option[KafkaProducer[String, String]] = None) {

  type Key = String
  type Val = String

  require(brokerList == null || !brokerList.isEmpty, "Must set broker list")

  private val p = producer getOrElse {

    var props:Properties= new Properties();
    props.put("bootstrap.servers", brokerList);
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");

    new KafkaProducer[String,String](props)
  }


  //把我的消息包装成了ProducerRecord
  private def toMessage(value: Val, key: Option[Key] = None, topic: Option[String] = None): ProducerRecord[Key, Val] = {
    val t = topic.getOrElse(defaultTopic.getOrElse(throw new IllegalArgumentException("Must provide topic or default topic")))
    require(!t.isEmpty, "Topic must not be empty")
    key match {
      case Some(k) => new ProducerRecord(t, k, value)
      case _ => new ProducerRecord(t, value)
    }
  }

  def send(key: Key, value: Val, topic: Option[String] = None) {
    //调用KafkaProducer他的send方法发送消息
    p.send(toMessage(value, Option(key), topic))
  }

  def send(value: Val, topic: Option[String]) {
    send(null, value, topic)
  }

  def send(value: Val, topic: String) {
    send(null, value, Option(topic))
  }

  def send(value: Val) {
    send(null, value, None)
  }

  def shutdown(): Unit = p.close()

}

// 继承一个基础的连接池，需要提供池化的对象类型
class PooledKafkaProducerAppFactory(val factory: KafkaProducerFactory)
  extends BasePooledObjectFactory[KafkaProducerProxy] with Serializable {

  // 用于池来创建对象
  override def create(): KafkaProducerProxy = factory.newInstance()

  // 用于池来包装对象
  override def wrap(obj: KafkaProducerProxy): PooledObject[KafkaProducerProxy] = new DefaultPooledObject(obj)

  // 用于池来销毁对象
  override def destroyObject(p: PooledObject[KafkaProducerProxy]): Unit = {
    p.getObject.shutdown()
    super.destroyObject(p)
  }

}
```
### 12.3.5 转换
#### 12.3.5.1 方法定义位置

1、DStream的类定义中，主要提供对于值类型的DStream的操作。

2、PairDStreamFunctions的类定义中，主要提供对于K-V形式的DStream的操作。

#### 12.3.5.2 无状态转换
#### 12.3.5.2.1 map

```scala
def map[U: ClassTag](mapFunc: T => U): DStream[U] : 将源DStream中的每个元素通过一个函数func从而得到新的DStreams。
```
#### 12.3.5.2.2 flatMap

```scala
def flatMap[U: ClassTag](flatMapFunc: T => TraversableOnce[U]): DStream[U] : 和map类似，但是每个输入的项可以被映射为0或更多项。
```
#### 12.3.5.2.3 filter

```scala
def filter(filterFunc: T => Boolean): DStream[T] : 选择源DStream中函数func判为true的记录作为新DStream
```
#### 12.3.5.2.4 repartition

```scala
def repartition(numPartitions: Int): DStream[T] : 通过创建更多或者更少的partition来改变此DStream的并行级别。
```
#### 12.3.5.2.5 union

```scala
def union(that: DStream[T]): DStream[T] : 将一个具有相同slideDuration新的DStream和当前DStream进行合并，返回新的DStream
```
#### 12.3.5.2.6 count

```scala
def count(): DStream[Long] : 统计源DStreams中每个RDD所含元素的个数得到单元素RDD的新DStreams。
```
#### 12.3.5.2.7 reduce

```scala
def reduce(reduceFunc: (T, T) => T): DStream[T] : 通过函数func(两个参数一个输出)来整合源DStreams中每个RDD元素得到单元素RDD的DStream。
```
#### 12.3.5.2.8 countByValue

```scala
def countByValue(numPartitions: Int = ssc.sc.defaultParallelism)(implicit ord: Ordering[T] = null) : DStream[(T, Long)] : 对于DStreams中元素类型为K调用此函数，得到包含(K,Long)对的新DStream，其中Long值表明相应的K在源DStream中每个RDD出现的次数。
```
#### 12.3.5.2.9 reduceByKey

```scala
def reduceByKey(reduceFunc: (V, V) => V): DStream[(K, V)] : 对(K,V)对的DStream调用此函数，返回同样（K,V)对的新DStream，但是新DStream中的对应V为使用reduce函数整合而来
```
#### 12.3.5.2.10 join

```scala
def join[W: ClassTag](other: DStream[(K, W)]): DStream[(K, (V, W))] : 两DStream分别为(K,V)和(K,W)对，返回(K,(V,W))对的新DStream。
```
#### 12.3.5.2.11 cogroup

```scala
def cogroup[W: ClassTag](other: DStream[(K, W)]): DStream[(K, (Iterable[V], Iterable[W]))] : 两DStream分别为(K,V)和(K,W)对，返回(K,(Seq[V],Seq[W])对新DStream
```
#### 12.3.5.2.12 transform

```scala
def transform[U: ClassTag](transformFunc: RDD[T] => RDD[U]): DStream[U] : 将RDD到RDD映射的函数func作用于源DStream中每个RDD上得到新DStream。这个可用于在DStream的RDD上做任意操作。注意的是，在这个转换函数里面能够应用所有RDD的转换操作。
```
#### 12.3.5.3 有状态转换
#### 12.3.5.3.1 updateStateByKey

![](../img/_1536979962_19204.png)

```scala
def updateStateByKey[S: ClassTag]( updateFunc: (Seq[V], Option[S]) => Option[S] ): DStream[(K, S)]
1、S是你需要保存的状态的类型。
2、updateFunc 是定义了每一批次RDD如何来更新的状态值。 Seq[V] 是当前批次相同key的值的集合。 Option[S] 是框架自动提供的，上一次保存的状态的值。
3、updateStateByKey会返回一个新的DStream，该DStream中保存了（Key,State）的序列。
```
```scala
object StateFullWordCount {

  //  创建 SparkConf 对象
  val conf: SparkConf = new SparkConf().setAppName("StateFullWordCount").setMaster("local[4]")

  //  定义一个更新方法，values 是当前批次 RDD 中相同 key 的 value 集合，state 是框架提供的上次 state 的值
  val updateFunc: (Seq[Int], Option[Int]) => Option[Int] = (values : Seq[Int], state : Option[Int]) => {
    //  计算当前批次相同 key 的单词总数
    val currentCount: Int = values.foldLeft(0)(_ + _)
    //  获取上一次保存的单词总数
    val previousCount: Int = state.getOrElse(0)
    //  返回新的单词总数
    Some(currentCount + previousCount)
  }

  def main(args: Array[String]): Unit = {

    //  创建 StreamingContext 对象
    val ssc: StreamingContext = new StreamingContext(conf, Seconds(1))
    //  设置 checkpoint 的目录
    ssc.checkpoint(".")
    //  读取文本数据
    val stream: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop1", 9999)
    //  处理文本数据，是用 updateStateByKey 方法，类型参数是状态的类型，后面传入一个更新方法
    val result: DStream[(String, Int)] = stream.flatMap(_.split(" ")).map((_, 1)).updateStateByKey(updateFunc)
    //  打印数据
    result.print()
    result.saveAsTextFiles("hdfs://hadoop1:8020/output/spark/wordcount/state-full-wordcount/")

    //  启动流式处理程序
    ssc.start()
    //  等待停止信号
    ssc.awaitTermination()

  }
}
```
#### 12.3.5.3.2 window 函数

![](../img/_1536980236_31797.png)

**window :**

```scala
def window(windowDuration: Duration, slideDuration: Duration): DStream[T] : 基于对源DStream窗化的批次进行计算返回一个新的DStream，windowDuration是窗口大小，slideDuration滑动步长。
```
**countByWindow :**

```scala
def countByWindow( windowDuration: Duration, slideDuration: Duration): DStream[Long] : 注意，返回的是window中记录的条数。
```
**reduceByWindow :**

```scala
def reduceByWindow( reduceFunc: (T, T) => T, windowDuration: Duration, slideDuration: Duration): DStream[T] : 通过使用自定义函数整合滑动区间流元素来创建一个新的单元素流。
```
**reduceByKeyAndWindow :**

![](../img/_1536980504_30101.png)

```scala
def reduceByKeyAndWindow(reduceFunc: (V, V) => V,windowDuration: Duration， slideDuration: Duration): DStream[(K, V)] : 通过给定的窗口大小以滑动步长来应用reduceFunc函数，返回DStream[(K, V)], K就是DStream中相应的K，V是window应用了reduce之后产生的最终值。

def reduceByKeyAndWindow(reduceFunc: (V, V) => V,invReduceFunc: (V, V) => V,windowDuration: Duration,slideDuration: Duration = self.slideDuration, numPartitions: Int = ssc.sc.defaultParallelism, filterFunc: ((K, V)) => Boolean = null): DStream[(K, V)] : 这个版本是上一个版本的优化版本，计算方式不同，采用增量计算的模式，每次计算只是在上一次计算的基础上减去丢失的值，加上增加的值。
```
```scala
object WindowWordCount {

  //  创建 SparkConf 对象
  val conf: SparkConf = new SparkConf().setAppName("WindowWordCount").setMaster("local[4]")

  def main(args: Array[String]): Unit = {

    //  创建 StreamingContext 对象
    val ssc: StreamingContext = new StreamingContext(conf, Seconds(2))
    //  设置 checkpoint 路径
    ssc.checkpoint(".")
    //  读取文本数据
    val stream: ReceiverInputDStream[String] = ssc.socketTextStream("hadoop1", 9999)
    //  处理数据
    val resultStream: DStream[(String, Int)] = stream.flatMap(_.split(" ")).map((_, 1)).reduceByKeyAndWindow((x : Int, y : Int) => (x + y), Seconds(6), Seconds(4))
    //  显示数据
    resultStream.print()

    //  启动流式处理程序
    ssc.start()
    //  等待停止信号
    ssc.awaitTermination()

  }
}
```
### 12.3.6 检查点

1、StreamingContext是能够从检查点中恢复数据的，可以通过StreamingContext.getOrCreate(checkPointDir)来创建。

2、Streaming中的累加器和广播变量是不能够从检查点中恢复。

### 12.3.7 输出
#### 12.3.7.1 print

```scala
print() : 在运行流程序的驱动结点上打印DStream中每一批次数据的最开始10个元素。这用于开发和调试。在Python API中，同样的操作叫pprint()。 
```
#### 12.3.7.2 saveAsTextFiles

```scala
saveAsTextFiles(prefix, [suffix]) : 以text文件形式存储这个DStream的内容。每一批次的存储文件名基于参数中的prefix和suffix。”prefix-Time_IN_MS[.suffix]”. 
```
#### 12.3.7.3 saveAsObjectFiles

```scala
saveAsObjectFiles(prefix, [suffix]) : 以Java对象序列化的方式将Stream中的数据保存为 SequenceFiles . 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]". Python中目前不可用。
```
#### 12.3.7.4 saveAsHadoopFiles

```scala
saveAsHadoopFiles(prefix, [suffix]) : 将Stream中的数据保存为 Hadoop files. 每一批次的存储文件名基于参数中的为"prefix-TIME_IN_MS[.suffix]". 
```
#### 12.3.7.5 foreachRDD

```scala
foreachRDD(func) : 这是最通用的输出操作，即将函数func用于产生于stream的每一个RDD。其中参数传入的函数func应该实现将每一个RDD中数据推送到外部系统，如将RDD存入文件或者通过网络将其写入数据库。注意：函数func在运行流应用的驱动中被执行，同时其中一般函数RDD操作从而强制其对于流RDD的运算。
```
## 12.4 内核解析
### 12.4.1 通信架构

![](../img/_1537066570_10053.png)

![](../img/_1537067316_7178.png)

### 12.4.2 脚本解析

![](../img/_1537067446_4210.png)

![](../img/_1537067498_26630.png)

### 12.4.3 启动流程

![](../img/_1537067600_3348.png)

![](../img/_1537067650_31740.png)

### 12.4.4 应用提交过程

![](../img/_1537067699_12473.png)

【橙色:应用提交】--》【紫色:启动Driver进程】--》【红色:注册Application】--》【蓝色:启动Executor进程】--》【粉色:启动Task执行】--》【绿色:Tash运行完成】

![](../img/_1537067791_31504.png)

### 12.4.5 Shuffle 过程

![](../img/_1537067913_23447.png)

### 12.4.6 内存管理与分配

![](../img/_1537067969_20433.png)

![](../img/_1537068004_29792.png)

![](../img/_1537068065_31469.png)

## 12.5 性能调优
### 12.5.1 数据倾斜的场景及解决方法

![](../img/_1537173072_15241.png)

#### 12.5.1.1 数据源中的数据分布不均匀，Spark需要频繁交互

1、实现原理：通过在Hive中对倾斜的数据进行预处理，以及在进行kafka数据分发时尽量进行平均分配。这种方案从根源上解决了数据倾斜，彻底避免了在Spark中执行shuffle类算子，那么肯定就不会有数据倾斜的问题了。

2、 方案优点：实现起来简单便捷，效果还非常好，完全规避掉了数据倾斜，Spark作业的性能会大幅度提升。

3、方案缺点：治标不治本，Hive或者Kafka中还是会发生数据倾斜。

4、适用情况：在一些Java系统与Spark结合使用的项目中，会出现Java代码频繁调用Spark作业的场景，而且对Spark作业的执行性能要求很高，就比较适合使用这种方案。将数据倾斜提前到上游的Hive ETL，每天仅执行一次，只有那一次是比较慢的，而之后每次Java调用Spark作业时，执行速度都会很快，能够提供更好的用户体验。

#### 12.5.1.2 数据集中的不同Key由于分区方式，导致数据倾斜

![](../img/_1537171794_23373.png)

1、实现原理：增加shuffle read task的数量，可以让原本分配给一个task的多个key分配给多个task，从而让每个task处理比原来更少的数据。
方案优点：实现起来比较简单，可以有效缓解和减轻数据倾斜的影响。

2、方案缺点：只是缓解了数据倾斜而已，没有彻底根除问题，根据实践经验来看，其效果有限。

3、实践经验：该方案通常无法彻底解决数据倾斜，因为如果出现一些极端情况，比如某个key对应的数据量有100万，那么无论你的task数量增加到多少，都无法处理。

#### 12.5.1.3 JOIN操作
#### 12.5.1.3.1 一个数据集中的数据分布不均匀，另一个数据集较小

![](../img/_1537171898_20823.png)

1、方案适用场景：在对RDD使用join类操作，或者是在Spark SQL中使用join语句时，而且join操作中的一个RDD或表的数据量比较小（比如几百M），比较适用此方案。

2、方案实现原理：普通的join是会走shuffle过程的，而一旦shuffle，就相当于会将相同key的数据拉取到一个shuffle read task中再进行join，此时就是reduce join。但是如果一个RDD是比较小的，则可以采用广播小RDD全量数据+map算子来实现与join同样的效果，也就是map join，此时就不会发生shuffle操作，也就不会发生数据倾斜。

3、方案优点：对join操作导致的数据倾斜，效果非常好，因为根本就不会发生shuffle，也就根本不会发生数据倾斜。

4、方案缺点：适用场景较少，因为这个方案只适用于一个大表和一个小表的情况。

#### 12.5.1.3.2 两个数据集都比较大，其中只有几个Key的数据分布不均匀

![](../img/_1537172018_29595.png)

1、适用场景：两张表都比较大，无法使用Map则Join。其中一个RDD有少数几个Key的数据量过大，另外一个RDD的Key分布较为均匀。

2、解决方案：将有数据倾斜的RDD中倾斜Key对应的数据集单独抽取出来加上随机前缀，另外一个RDD每条数据分别与随机前缀结合形成新的RDD（笛卡尔积，相当于将其数据增到到原来的N倍，N即为随机前缀的总个数），然后将二者Join后去掉前缀。然后将不包含倾斜Key的剩余数据进行Join。最后将两次Join的结果集通过union合并，即可得到全部Join结果。

3、优势：相对于Map侧Join，更能适应大数据集的Join。如果资源充足，倾斜部分数据集与非倾斜部分数据集可并行进行，效率提升明显。且只针对倾斜部分的数据做数据扩展，增加的资源消耗有限。

4、劣势：如果倾斜Key非常多，则另一侧数据膨胀非常大，此方案不适用。而且此时对倾斜Key与非倾斜Key分开处理，需要扫描数据集两遍，增加了开销。

#### 12.5.1.3.3 两个数据集都比较大，有很多Key的数据分布不均匀

1、方案适用场景：如果在进行join操作时，RDD中有大量的key导致数据倾斜，那么进行分拆key也没什么意义。

2、方案实现思路：将该RDD的每条数据都打上一个n以内的随机前缀。同时对另外一个正常的RDD进行扩容，将每条数据都扩容成n条数据，扩容出来的每条数据都依次打上一个0~n的前缀。最后将两个处理后的RDD进行join即可。和上一种方案是尽量只对少数倾斜key对应的数据进行特殊处理，由于处理过程需要扩容RDD，因此上一种方案扩容RDD后对内存的占用并不大；而这一种方案是针对有大量倾斜key的情况，没法将部分key拆分出来进行单独处理，因此只能对整个RDD进行数据扩容，对内存资源要求很高。

3、方案优点：对join类型的数据倾斜基本都可以处理，而且效果也相对比较显著，性能提升效果非常不错。

4、方案缺点：该方案更多的是缓解数据倾斜，而不是彻底避免数据倾斜。而且需要对整个RDD进行扩容，对内存资源要求很高。

5、方案实践经验：曾经开发一个数据需求的时候，发现一个join导致了数据倾斜。优化之前，作业的执行时间大约是60分钟左右；使用该方案优化之后，执行时间缩短到10分钟左右，性能提升了6倍。

#### 12.5.1.4 聚合操作中，数据集中的数据分布不均匀

![](../img/_1537172235_24724.png)

1、方案适用场景：对RDD执行reduceByKey等聚合类shuffle算子或者在Spark SQL中使用group by语句进行分组聚合时，比较适用这种方案

2、方案实现原理：将原本相同的key通过附加随机前缀的方式，变成多个不同的key，就可以让原本被一个task处理的数据分散到多个task上去做局部聚合，进而解决单个task处理数据量过多的问题。接着去除掉随机前缀，再次进行全局聚合，就可以得到最终的结果。具体原理见下图。

3、方案优点：对于聚合类的shuffle操作导致的数据倾斜，效果是非常不错的。通常都可以解决掉数据倾斜，或者至少是大幅度缓解数据倾斜，将Spark作业的性能提升数倍以上。

4、方案缺点：仅仅适用于聚合类的shuffle操作，适用范围相对较窄。如果是join类的shuffle操作，还得用其他的解决方案

#### 12.5.1.5 数据集中少数几个key数据量很大，不重要，其他数据均匀

1、适用场景：如果发现导致倾斜的key就少数几个，而且对计算本身的影响并不大的话，那么很适合使用这种方案。比如99%的key就对应10条数据，但是只有一个key对应了100万数据，从而导致了数据倾斜。

2、方案优点：实现简单，而且效果也很好，可以完全规避掉数据倾斜。

3、方案缺点：适用场景不多，大多数情况下，导致倾斜的key还是很多的，并不是只有少数几个。

4、实践经验：在项目中我们也采用过这种方案解决数据倾斜。有一次发现某一天Spark作业在运行的时候突然OOM了，追查之后发现，是Hive表中的某一个key在那天数据异常，导致数据量暴增。因此就采取每次执行前先进行采样，计算出样本中数据量最大的几个key之后，直接在程序中将那些key给过滤掉。

### 12.5.2 资源调优

![](../img/_1537174999_11644.png)

1、num-executors：应用运行时 executor 的数量，推荐 50 - 100 左右比较合适。

2、executor-memory：应用运行时 executor 的内存，推荐 4-  8 G 比较合适。

3、executor-cores：应用运行时 executor 的 cpu 核数，推荐 2 - 4 个比较合适。

4、driver-memory：应用运行时 driver 的内存量，主要考虑如果用 map side join 或者一些类似于 collect 的操作，那么要相应调大内存量。

5、spark.default.parallelism：每个 stage 默认的 task 数量，推荐参数为 num-executors * executor-cores 的 2 ~ 3 倍较为合适。

6、spark.storage.memoryFraction：每一个 executor 中用于 rdd 缓存的内存比例，如果程序中有大量的数据缓存，可以考虑调大整个比例，默认是 60 %。

7、spark.shuffle.memoryFraction：每一个 executor 中用于 shuffle 操作的内存比例，默认是 20 %，如果程序中有大量的 shuffle 类算子，那么可以考虑调整它的比例。

### 12.5.3 开发优化
#### 12.5.3.1 避免创建重复的RDD

![](../img/_1537177215_21980.png)

#### 12.5.3.2 尽可能复用同一个RDD

![](../img/_1537177273_3582.png)

#### 12.5.3.3 对多次使用的RDD进行持久化

![](../img/_1537177363_3589.png)

#### 12.5.3.4 尽量避免使用shuffle类算子

![](../img/_1537177405_8219.png)

#### 12.5.3.5 使用map-side预聚合的shuffle操作

![](../img/_1537177446_20743.png)

#### 12.5.3.6 使用高性能的算子

![](../img/_1537177489_28587.png)

#### 12.5.3.7 广播大变量

![](../img/_1537177547_2740.png)

#### 12.5.3.8 使用Kryo优化序列化性能

![](../img/_1537178302_29484.png)

#### 12.5.3.9 分区Shuffle优化

![](../img/_1537178406_29120.png)

#### 12.5.3.10 优化数据结构

![](../img/_1537178444_24577.png)

### 12.5.4 Shuffle 过程优化

**1、spark.shuffle.file.buffer：**该参数用于设置shuffle write task的BufferedOutputStream的buffer缓冲大小。将数据写到磁盘文件之前，会先写入buffer缓冲中，待缓冲写满之后，才会溢写到磁盘。如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如64k），从而减少shuffle write过程中溢写磁盘文件的次数，也就可以减少磁盘IO次数，进而提升性能。

**2、spark.reducer.maxSizeInFlight：**该参数用于设置shuffle read task的buffer缓冲大小，而这个buffer缓冲决定了每次能够拉取多少数据。如果作业可用的内存资源较为充足的话，可以适当增加这个参数的大小（比如96m），从而减少拉取数据的次数，也就可以减少网络传输的次数，进而提升性能

**3、spark.shuffle.io.maxRetries：**shuffle read task从shuffle write task所在节点拉取属于自己的数据时，如果因为网络异常导致拉取失败，是会自动进行重试的。该参数就代表了可以重试的最大次数。如果在指定次数之内拉取还是没有成功，就可能会导致作业执行失败。对于那些包含了特别耗时的shuffle操作的作业，建议增加重试最大次数（比如60次），以避免由于JVM的full gc或者网络不稳定等因素导致的数据拉取失败。在实践中发现，对于针对超大数据量（数十亿~上百亿）的shuffle过程，调节该参数可以大幅度提升稳定性。

**4、spark.shuffle.io.retryWait：**shuffle read task从shuffle write task所在节点拉取属于自己的数据时，如果因为网络异常导致拉取失败，是会自动进行重试的，该参数代表了每次重试拉取数据的等待间隔，默认是5s。建议加大间隔时长（比如60s），以增加shuffle操作的稳定性。

**5、spark.shuffle.memoryFraction：**该参数代表了Executor内存中，分配给shuffle read task进行聚合操作的内存比例，默认是20%。在资源参数调优中讲解过这个参数。如果内存充足，而且很少使用持久化操作，建议调高这个比例，给shuffle read的聚合操作更多内存，以避免由于内存不足导致聚合过程中频繁读写磁盘。在实践中发现，合理调节该参数可以将性能提升10%左右。

**6、spark.shuffle.manager：**该参数用于设置ShuffleManager的类型。Spark 1.5以后，有三个可选项：hash、sort和tungsten-sort。tungsten-sort与sort类似，但是使用了tungsten计划中的堆外内存管理机制，内存使用效率更高。由于SortShuffleManager默认会对数据进行排序，因此如果你的业务逻辑中需要该排序机制的话，则使用默认的SortShuffleManager就可以；而如果你的业务逻辑不需要对数据进行排序，那么建议参考后面的几个参数调优，通过bypass机制或优化的HashShuffleManager来避免排序操作，同时提供较好的磁盘读写性能。

**7、spark.shuffle.sort.bypassMergeThreshold：**当ShuffleManager为SortShuffleManager时，如果shuffle read task的数量小于这个阈值（默认是200），则shuffle write过程中不会进行排序操作，而是直接按照未经优化的HashShuffleManager的方式去写数据，但是最后会将每个task产生的所有临时磁盘文件都合并成一个文件，并会创建单独的索引文件。当你使用SortShuffleManager时，如果的确不需要排序操作，那么建议将这个参数调大一些，大于shuffle read task的数量。那么此时就会自动启用bypass机制，map-side就不会进行排序了，减少了排序的性能开销。但是这种方式下，依然会产生大量的磁盘文件，因此shuffle write性能有待提高。

**8、spark.shuffle.consolidateFiles：**如果使用HashShuffleManager，该参数有效。如果设置为true，那么就会开启consolidate机制，会大幅度合并shuffle write的输出文件，对于shuffle read task数量特别多的情况下，这种方法可以极大地减少磁盘IO开销，提升性能。如果的确不需要SortShuffleManager的排序机制，那么除了使用bypass机制，还可以尝试将spark.shffle.manager参数手动指定为hash，使用HashShuffleManager，同时开启consolidate机制。