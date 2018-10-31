# 9 Hive

在 Hive 中，当使用 local 时，表示是本地路径，当不使用 local 时，表示 HDFS 路径，当使用 into 时，表示追加写，当使用 overwrite 时，表示覆盖写

## 9.1 原理
### 9.1.1 数据仓库和数据库的区别

1、数据库是面向事务而设计，数据仓库是面向主题而设计

2、数据库主要用来存储业务数据，数据仓库主要用来存储历史数据

3、数据库主要用于捕获数据，例如捕获 JavaEE 中页面传过来的数据，进行处理后然后进行响应，数据仓库主要用于分析数据，从数据仓库中获取数据，然后进行分析

4、数据库应尽量避免数据冗余，数据仓库有时为了数据分析的方便，会有意地引入数据冗余

## 9.2 DDL

```sql
CREATE TABLE employee (
NAME string,
salary FLOAT,
subordinated array < string >,
deductions map < string, FLOAT >,
address struct < province:string, city:string, state:string, zip:INT > 
)
partitioned by (province string, state string)
clustered by (salary)
into 4 buckets
ROW format delimited
FIELDS TERMINATED BY '\t'
collection items TERMINATED BY ','
map KEYS TERMINATED BY '=';

测试数据 :
John Doe    10000.0    Mary Sith,Todd Jones    Federal Taxes=0.2,State Taxes=0.1,Insurance=0.1    1 Michigan Ave.,Chicago,IL,60600
Mary Smith    80000.0    Bill King    Federal Taxes=0.2,State Taxes=0.05,Insurance=0.1    100 Ontario St.,Chicago,IL,60601
Todd Jones    70000.0        Federal Taxes=0.15,State Taxes=0.03,Insurance=0.1    200 Chicago Ave.,Oak Park,NY,60700
Bill King    60000.0        Federal Taxes=0.15,State Taxes=0.03,Insurance=0.1    300 Obscure Dr.,Obscur,CA,6010
```
### 9.2.1 分区表
#### 9.2.1.1 分区表的好处

分区表主要用于辅助查询，缩小查询范围，提高检索速度，同时可以按照一定的规格和条件对数据进行管理，分区表中的每一个分区对应的都是一个文件夹，分区中的数据对应于文件夹中的数据

#### 9.2.1.2 分区表的常用操作

1、将数据从文件中加载入分区表

```sql
// 将数据从文件 file 加载到表 tableName 的分区值分别为 partitionValue1、partitionValue2 的分区 partitionName1、partitionName2 中
load data local inpath 'file' into table tableName partition(partitionName1='partitionValue1', partitionName2='partitionValue2')
```
2、将数据动态的插入到分区表

```sql
// 从表 tableName2 中查询数据并将其插入到表 tableName2 中的指定分区，其中表 tableName1 中的字段包括 field1、field2，分区值分别为 partitionName1、partitionName2
insert into table tableName1 partition (partitionName1, partitionName2) select field1, field2, field3 as partitionName1, field4 as partitionName2 from tableName2;
```
### 9.2.2 桶表
#### 9.2.3 桶表的好处

粪桶表可以用于对数据的抽样调查，同时由于桶表的内部机制，可以减少 join 的次数，从而提高效率，桶表相当于 MapReduce 中的分区，每一桶对应于一个文件，每一桶中的数据对应于每一个文件中的数据，桶表中通过将桶表中分桶字段的值与桶的个数进行模运算，从而确定每一条数据处于哪一个桶中

#### 9.2.4 桶表的常用操作

1、在创建桶表时，需要先开启分桶

```sql
set hive.enforce.bucketing = true;
```
2、设置 reduce 的个数（当在创建桶表的时候指定了桶的个数时也可以不用设置这个参数值，此时 reduce 的个数就等于桶的个数）

```sql
set mapreduce.job.reduces=4;
```
3、将数据插入到桶表

```sql
// 从表 tableName2 中将数据查询出来并将其插入到 tableName1 中
insert into tableName1 select * from tableName2 cluster by (clusterName);
```
### 9.2.3 内部表和外部表
#### 9.2.3 内部表和外部表的区别

1、删除表时，删除内部表，会将元数据和数据目录一起删除，删除外部表，只会删除元数据，而数据目录不会被删除

2、创建表时，创建内部表会出现在数据库目录中，创建外部表不会出现在数据库目录中

### 9.2.4 常用 DDL 命令
#### 9.2.4.1 修改表

1、增加分区

```sql
alter table tableName add partition(partitionName1=partitionValue1) location '/user/hive/warehouse/databaseName/tableName/partitionName1=partitionValue1' partition(partitionName2=partitionValue2) location '/user/hive/warehouse/databaseName/tableName/partitionName2=partitionValue2'
```
2、删除分区

```sql
alter table tableName drop if exists partition(partitionName1=partitionValue1, partitionName2=partitionValue2)
```
3、修改分区

```sql
alter table tableName partition(partitionName=partitionValue) rename to partition(partitionName=newPartitionValue)
```
4、添加列

```sql
alter table tableName add columns(name string)
```
5、修改列

```sql
alter table tableName change id int

alter table tableName change id int after name

alter table tableName change id int first
```
6、表重命名

```sql
alter table tableName rename to newTableName
```
7、like

```sql
// 创建表 tableName1，其结构与 tableName2 一样，但没有数据
create table tableName1 like tableName2

// 创建 tableName1，其结构与 tableName2 一样，同时数据也和 tableName2 一样
create table tableName1 as select * from tableName2
```
## 9.2 DML
### 9.2.1 常用 DML 命令

1、insert

```sql
// 将从 tableName2 中查到的数据插入到 tableName1 中，其中表 tableName2 查询结果的结构与表 tableName1 一样
insert into table tableName1 select * from tableName2

// 将从 tableName1 中查询到的 id 插入到 tableName2 中，将从 tableName1 中查询到的 name 插入到 tableName3 中
from tableName1
insert into table tableName2
select id
insert into table tableName3
select name

// 将从 tableName 中查到的数据写入到本地文件夹 file 中
insert into local directory 'file' select * from tableName

// 将从 tableName1 中查到的 id 写入到本地文件夹 file1 中，将从 tableName2 中查到的 name 写入到本地文件夹 file2 中
from tableName1
insert into local directory 'file1'
select id
insert into local directory 'file2'
select name
```
2、select

```sql
// order by : 所有数据进入同一个 reducer 进行处理，并在 reducer 中对数据进行全局排序，由于只有一个 reducer，所以对于大量数据，将会消耗很长时间去执行
// sort by : 为每一个 reducer 进行排序，保证了数据的局部有序，接下来再通过一次归并排序就可以实现全局有序，可以为全局排序提高效率
// distribute by : 指定数据在 reducer 端是如何划分的，例如一个表有 mid、name两个字段，当使用 distribute by (mid) 时，所有的 mid 相同的数据会被放到相同的 reducer 中进行处理，一般可以结合 sort by 一起使用
// cluster by : 相当于 distribute by 和 sort by 结合在一起，例如 cluster by (id) 就相当于 distribute by (id) sort by (id)
select * from tableName order by id asc
select * from tableName distribute by (id) sort by (salary)
select * from tableName cluster by (id)
```
## 9.3 内置函数
### 9.3.1 regexp_replace

```shell
# 对字符串中指定的字符串使用指定的分隔符进行替换
select regexp_replace('"http://www.taobao.com/3c/items?id=001&name=book"', '\"', '');

运行结果 :
+--------------------------------------------------+--+
|                       _c0                        |
+--------------------------------------------------+--+
| http://www.taobao.com/3c/items?id=001&name=book  |
+--------------------------------------------------+--+
```
### 9.3.2 split

```shell
# 将字符串以指定的分隔符进行分割
select split('beijing,shanghai,guangzhou,shenzhen', ',');

运行结果 :
+------------------------------------------------+--+
|                      _c0                       |
+------------------------------------------------+--+
| ["beijing","shanghai","guangzhou","shenzhen"]  |
+------------------------------------------------+--+
```
### 9.3.3 array

```shell
# 生成数组
select array('a','b','c');

运行结果 :
+----------------+--+
|      _c0       |
+----------------+--+
| ["a","b","c"]  |
+----------------+--+
```
### 9.3.4 explode

```shell
# 将一个数组中的每一个元素分行显示
select explode(split('beijing,shanghai,guangzhou,shenzhen', ','));

运行结果 :
+------------+--+
|    col     |
+------------+--+
| beijing    |
| shanghai   |
| guangzhou  |
| shenzhen   |
+------------+--+
```
### 9.3.5 parse_url_tuple

```shell
# 从一个 url 字符串中查询出主机名、请求路径、请求参数等信息
select  parse_url_tuple('http://www.taobao.com/3c/items?id=001&name=book','HOST','PATH', 'QUERY', 'QUERY:name') as (host,path,query,name);

运行结果 :
+-----------------+------------+-------------------+-------+--+
|      host       |    path    |       query       | name  |
+-----------------+------------+-------------------+-------+--+
| www.taobao.com  | /3c/items  | id=001&name=book  | book  |
+-----------------+------------+-------------------+-------+--+
```
### 9.3.6 union

```shell
# 将两个表中的数据联合起来放在一个表中
select 'http://www.taobao.com/3c/items?id=001&name=book' as url union select 'http://www.tmall.com/3c/items?id=001&name=book&date=2018' as url;

+-----------------------------------------------------------+--+
|                          _u2.url                          |
+-----------------------------------------------------------+--+
| http://www.taobao.com/3c/items?id=001&name=book           |
| http://www.tmall.com/3c/items?id=001&name=book&date=2018  |
+-----------------------------------------------------------+--+
```
### 9.3.7 lateral view

```shell
# 将一个表中的每一条数据都和另一个表中的相应数据连接起来
select a.name, b.* from (select 'zhangsan' as name, 'beijing,shanghai' as location union select 'lisi' as name, '广州,shenzhen,海南' as location) a lateral view explode(split(location, ',')) b as kkk;

运行结果 :
+-----------+-----------+--+
|  a.name   |   b.kkk   |
+-----------+-----------+--+
| lisi      | 广州        |
| lisi      | shenzhen  |
| lisi      | 海南        |
| zhangsan  | beijing   |
| zhangsan  | shanghai  |
+-----------+-----------+--+
```